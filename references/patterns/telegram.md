# Telegram Bot — паттерны

## Webhook handler — нормализация update

Точка входа принимает POST от Telegram, парсит JSON и нормализует в единый `$currentData`:

```parser3
@main[]
$data[^json:parse[^taint[as-is][$request:body]]]

$currentData[
    $.chat_id[$data.message.chat.id]
    $.user_id[$data.message.from.id]
    $.message_id($data.message.message_id)
]
^if($data.message && def $data.message.text){
    $currentData.text[$data.message.text]
}
^if($data.callback_query){
    $currentData[
        $.chat_id[$data.callback_query.message.chat.id]
        $.user_id[$data.callback_query.from.id]
        $.message_id($data.callback_query.message.message_id)
        $.callback_data[$data.callback_query.data]
    ]
}($data.channel_post){
    $currentData[
        $.chat_id[$data.channel_post.chat.id]
        $.user_id[0]
        $.message_id($data.channel_post.message_id)
    ]
}($data.inline_query){
    $currentData[$.user_id[$data.inline_query.from.id]]
}($data.my_chat_member){
    $currentData.chat_id[$data.my_chat_member.chat.id]
}($data.chat_join_request){
    $currentData.chat_id[$data.chat_join_request.chat.id]
    $currentData.user_id[$data.chat_join_request.from.id]
}

# личные сообщения боту: chat_id совпадает с user_id
$currentData.isInBot($currentData.chat_id == $currentData.user_id)
$MAIN:currentData[$currentData]
```

## Ответ боту напрямую через response

Экономит round-trip — Telegram принимает ответ прямо в теле HTTP-ответа:

```parser3
$response:content-type[application/json]
$response:body[^json:string[
    $.method[sendMessage]
    $.chat_id[$currentData.chat_id]
    $.text[текст]
    $.parse_mode[MarkdownV2]
]]
```

## Вызов API через curl

```parser3
@getJson[method;data]
$f[^curl:load[
    $.url[https://api.telegram.org/bot$self.token/$method]
    $.ssl_verifypeer(0)
    $.timeout(20)
    $.post(1)
    $.postfields[^json:string[$data]]
    $.httpheader[$.Content-type[application/json]]
]]
$result[^json:parse[^taint[as-is][$f.text]]]]
```

## Обработка rate limit (429)

```parser3
^if(!$send.ok){
    ^if($send.error_code == 429 && $send.parameters.retry_after){
        ^sleep($send.parameters.retry_after + 1)
        ^tg_code[]   # рекурсивный повтор
        ^return[]
    }
}
```

## Экранирование MarkdownV2

Telegram требует экранировать спецсимволы в MarkdownV2:

```parser3
@replaceText[text]
$symbols[_*[]()~`>#+-=|{}.!]
^for[i](0;^symbols.length[]){
    $s[^symbols.mid($i;1)]
    ^if(def $s){$text[^text.replace[$s;\$s]]}
}
$result[$text]
```

## Inline keyboard

```parser3
^Telegram.sendMessage[
    $.chat_id[$chat_id]
    $.text[$text]
    $.reply_markup[
        $.inline_keyboard[
            $.0[
                $.0[$.text[Кнопка 1] $.callback_data[action1]]
                $.1[$.text[Ссылка] $.url[https://example.com]]
            ]
            $.1[
                $.0[$.text[Кнопка 2] $.callback_data[action2]]
            ]
        ]
    ]
]
```

## Reply keyboard

```parser3
$keyboard[^hash::create[]]
$row(0)
$maxPerRow(2)
^items.menu{
    ^if(!def $keyboard.[$row]){$keyboard.[$row][^hash::create[]]}
    $keyboard.[$row].[^keyboard.[$row].count[]][$.text[$items.name]]
    ^if(^keyboard.[$row].count[] >= $maxPerRow){^row.inc[]}
}
^Telegram.sendMessage[
    $.chat_id[$chat_id]
    $.text[Выберите:]
    $.reply_markup[
        $.resize_keyboard(true)
        $.keyboard[$keyboard]
    ]
]
```

## Очередь сообщений через БД

Паттерн: складываем в `tg_queue`, cron достаёт и шлёт. Позволяет контролировать rate limit, повторять неудачные, не блокировать основной запрос.

```parser3
# добавление в очередь
^void:sql{insert into tg_queue set
    chat_id = $chat_id,
    text = '^taint[sql][$text]'
    ^if($buttons){,inline_buttons = '^taint[sql][^json:string[$buttons]]'}
}

# кэшируем тексты по md5 чтобы не дублировать большие тексты в БД
$key[^math:md5[$text]]
^if(!^keys.contains[$key]){
    ^void:sql{insert into tg_queue_texts set _key = '$key', text = '^taint[sql][$text]'
        ON DUPLICATE KEY UPDATE type = type}
    $keys.[$key][$text]
}
```

Обработка очереди в cron:
```parser3
$list[^table::sql{
    SELECT id, chat_id, text FROM tg_queue
    WHERE message_id = 0 AND dt_exec is null AND dt_result is null
    ORDER BY id LIMIT 200
}]
^void:sql{update tg_queue set dt_exec = now() where id in (^list.menu{$list.id}[,])}
^list.menu{
    $send[^Telegram.sendMessage[$.chat_id[$list.chat_id] $.text[^taint[as-is][$list.text]]]]
    ^if($send.ok){
        ^void:sql{update tg_queue set dt_result = now(), message_id = '$send.result.message_id' where id = $list.id}
    }{
        ^if($send.error_code == 429 && $send.parameters.retry_after){
            ^sleep($send.parameters.retry_after + 1)
            ^void:sql{update tg_queue set dt_exec = null where id = $list.id}
            ^tg_code[]
            ^return[]
        }{
            ^void:sql{update tg_queue set dt_result = now(), response_text = '^taint[sql][^json:string[$send]]' where id = $list.id}
        }
    }
}
```

## approveChatJoinRequest / declineChatJoinRequest

```parser3
^if($data.chat_join_request){
    $user_id[$data.chat_join_request.from.id]
    $chat_id[$data.chat_join_request.chat.id]
    ^if(^isUserAllowed[$user_id]){
        ^Telegram.call[approveChatJoinRequest;
            $.chat_id($chat_id)
            $.user_id($user_id)
        ]
    }{
        ^Telegram.call[declineChatJoinRequest;
            $.chat_id($chat_id)
            $.user_id($user_id)
        ]
    }
}
```

## Поддержка — форвард сообщений между ботом и группой

```parser3
# пользователь → группа поддержки
^Telegram.forwardMessage[
    $.chat_id($groupChatId)
    $.from_chat_id($currentData.chat_id)
    $.message_id[$currentData.message_id]
]

# оператор отвечает → копия пользователю
^Telegram.copyMessage[
    $.chat_id($userChatId)
    $.from_chat_id($groupChatId)
    $.message_id[$currentData.message_id]
]
```
