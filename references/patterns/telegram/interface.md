# Telegram.p — интерфейс класса

Стабильный фасад над Telegram Bot API. Используется через экземпляр `$Telegram` (обычно создаётся в `auto.p`).

## Создание

```parser3
$Telegram[^Telegram::create[]]

# с переопределением токена или parse_mode:
$Telegram[^Telegram::create[
    $.token[1234567890:AAxxxxxxxx]
    $.parse_mode[HTML]
]]
```

Дефолты внутри класса: `parse_mode = MarkdownV2`.

## Методы

```parser3
# Отправить сообщение
^Telegram.sendMessage[
    $.chat_id[$currentData.chat_id]
    $.text[$text]
    $.parse_mode[MarkdownV2]
    $.reply_markup[...]      # опционально
]

# Удалить сообщение
^Telegram.deleteMessage[
    $.chat_id[$chat_id]
    $.message_id($message_id)
]

# Фото
^Telegram.sendPhoto[
    $.chat_id[$chat_id]
    $.photo[$file_id]
    $.caption[$text]
]

# Видео
^Telegram.sendVideo[
    $.chat_id[$chat_id]
    $.video[$file_id]
]

# Переслать сообщение
^Telegram.forwardMessage[
    $.chat_id($to_chat_id)
    $.from_chat_id($from_chat_id)
    $.message_id[$message_id]
]

# Скопировать сообщение (без отметки "переслано")
^Telegram.copyMessage[
    $.chat_id($to_chat_id)
    $.from_chat_id($from_chat_id)
    $.message_id[$message_id]
]

# Произвольный метод API
^Telegram.call[answerCallbackQuery;
    $.callback_query_id[$currentData.callback_query_id]
    $.text[Готово]
]

^Telegram.call[approveChatJoinRequest;
    $.chat_id($chat_id)
    $.user_id($user_id)
]
```

## Прямой ответ в тело HTTP-ответа

Экономит round-trip — Telegram принимает ответ прямо в теле webhook-запроса. Работает только для одного сообщения за запрос.

```parser3
^Telegram.response[
    $.method[sendMessage]
    $.chat_id[$currentData.chat_id]
    $.text[$text]
]
```

## Экранирование MarkdownV2

```parser3
$safeText[^Telegram.replaceText[$userText]]
```

Экранирует: `_ * [ ] ( ) ~ \` > # + - = | { } . !`

## setChatId

Привязывает `chat_id` ко всем последующим вызовам через `addData`. Сбрасывает внутреннюю очередь при смене chat_id.

```parser3
^Telegram.setChatId[$currentData.chat_id]
```

## Производственные замечания

- В `@getJson` может остаться `^throw[]` — дебаг-артефакт, убрать перед деплоем
- `proxy` захардкожен в классе — при переносе на другой сервер проверить или убрать
- `@replateText` — опечатка внутри класса, использовать `^Telegram.replaceText[]`
