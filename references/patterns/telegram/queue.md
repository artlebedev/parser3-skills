# Telegram — очередь сообщений

Паттерн: складываем сообщения в `tg_queue`, cron достаёт и шлёт.
Даёт: контроль rate limit, повторы при ошибках, не блокирует основной запрос.

## Добавление в очередь

```parser3
^void:sql{INSERT INTO tg_queue SET
    chat_id = $chat_id,
    text = '^taint[sql][$text]'
    ^if($buttons){,inline_buttons = '^taint[sql][^json:string[$buttons]]'}
}
```

## Дедупликация больших текстов через md5

```parser3
$key[^math:md5[$text]]
^if(!^keys.contains[$key]){
    ^void:sql{INSERT INTO tg_queue_texts SET
        _key = '$key',
        text = '^taint[sql][$text]'
        ON DUPLICATE KEY UPDATE type = type}
    $keys.[$key][$text]
}
```

## Обработка очереди в cron

```parser3
$list[^table::sql{
    SELECT id, chat_id, text FROM tg_queue
    WHERE message_id = 0 AND dt_exec IS NULL AND dt_result IS NULL
    ORDER BY id LIMIT 200
}]
^void:sql{UPDATE tg_queue SET dt_exec = NOW() WHERE id IN (^list.menu{$list.id}[,])}

^list.menu{
    $send[^Telegram.sendMessage[
        $.chat_id[$list.chat_id]
        $.text[^taint[as-is][$list.text]]
    ]]
    ^if($send.ok){
        ^void:sql{UPDATE tg_queue SET
            dt_result = NOW(),
            message_id = '$send.result.message_id'
            WHERE id = $list.id}
    }{
        ^if($send.error_code == 429 && $send.parameters.retry_after){
            ^sleep($send.parameters.retry_after + 1)
            ^void:sql{UPDATE tg_queue SET dt_exec = NULL WHERE id = $list.id}
            ^tg_code[]
            ^return[]
        }{
            ^void:sql{UPDATE tg_queue SET
                dt_result = NOW(),
                response_text = '^taint[sql][^json:string[$send]]'
                WHERE id = $list.id}
        }
    }
}
```

## Rate limit (429)

Telegram возвращает `retry_after` — количество секунд ожидания:

```parser3
^if($send.error_code == 429 && $send.parameters.retry_after){
    ^sleep($send.parameters.retry_after + 1)
    # повторить попытку
}
```
