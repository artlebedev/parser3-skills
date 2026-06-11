# Telegram — обработка webhook

## Нормализация входящего update

Точка входа (`index.html`) принимает POST от Telegram, парсит JSON и нормализует все типы update в единый `$currentData`:

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
        $.callback_query_id[$data.callback_query.id]
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

## Inline keyboard

```parser3
^Telegram.sendMessage[
    $.chat_id[$currentData.chat_id]
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
    $.chat_id[$currentData.chat_id]
    $.text[Выберите:]
    $.reply_markup[
        $.resize_keyboard(true)
        $.keyboard[$keyboard]
    ]
]
```

## approveChatJoinRequest / declineChatJoinRequest

```parser3
^if($data.chat_join_request){
    ^if(^isUserAllowed[$currentData.user_id]){
        ^Telegram.call[approveChatJoinRequest;
            $.chat_id($currentData.chat_id)
            $.user_id($currentData.user_id)
        ]
    }{
        ^Telegram.call[declineChatJoinRequest;
            $.chat_id($currentData.chat_id)
            $.user_id($currentData.user_id)
        ]
    }
}
```

## Поддержка — форвард между ботом и группой

```parser3
# пользователь → группа поддержки
^Telegram.forwardMessage[
    $.chat_id($groupChatId)
    $.from_chat_id($currentData.chat_id)
    $.message_id[$currentData.message_id]
]

# оператор отвечает → копия пользователю (без отметки "переслано")
^Telegram.copyMessage[
    $.chat_id($userChatId)
    $.from_chat_id($groupChatId)
    $.message_id[$currentData.message_id]
]
```
