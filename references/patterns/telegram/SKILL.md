# Telegram — скоупированные инструкции

## Procedure

1. Найди `Telegram.p` в проекте:
   ```bash
   find . -name "Telegram.p" -not -path "*/.git/*"
   ```
   - Если найден — используй его как интерфейс (читай `interface.md`)
   - Если не найден — читай `raw.md` для сырых curl-паттернов без класса

2. Определи тип задачи и прочитай нужный файл:
   - Методы класса, создание экземпляра, `response` → `interface.md`
   - Обработка webhook, `$currentData`, клавиатуры → `webhook.md`
   - Очередь сообщений, rate limit, cron → `queue.md`

3. Пиши код только через методы `Telegram.p` — не изобретай сырые curl-вызовы если класс доступен

## Validation

- `^Telegram.sendMessage[...]` — не `^curl:load[https://api.telegram.org/...]`
- `^Telegram.replaceText[$text]` — не самодельный цикл экранирования
- `^Telegram.response[...]` — для прямого ответа в webhook, не `$response:body[^json:string[...]]` вручную
- `^Telegram.call[method;data]` — для любого метода API которого нет отдельным методом в классе

## Gotchas

- **`^throw[]` в `@getJson`** — частый дебаг-артефакт в `Telegram.p`. Если остался — падает на каждом запросе. Проверь перед деплоем
- **`@replateText` vs `@replaceText`** — внутри класса опечатка, используй `^Telegram.replaceText[]`
- **`^Telegram.response[]`** — работает только раз за webhook-запрос. Второй вызов перезапишет первый
- **proxy захардкожен** — при переносе проекта на другой сервер проверить или убрать строку proxy из `@getJson`
- **`parse_mode` по умолчанию MarkdownV2** — все тексты должны быть экранированы через `^Telegram.replaceText[]`, иначе Telegram вернёт ошибку на спецсимволах
