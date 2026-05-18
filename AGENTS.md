---
name: parser3
description: "Use this skill when writing, reading, debugging, or reviewing Parser3 code — server-side templating language by Art. Lebedev Studio. Trigger on: .p files, auto.p, Parser3 syntax questions, ^method[] calls, $variable assignments, @handler[] methods, ^if/^while/^for operators, ^table::sql queries, ^void:sql, ^int:sql, Parser3 errors, or any mention of Parser3, p3, or parser language."
version: 0.1.0
---

# Parser3

Parser3 — объектно-ориентированный скриптовый язык программирования, созданный для генерации HTML-страниц на веб-сервере с поддержкой CGI.

## When to use

- Пишешь или правишь `.p` файлы (auto.p, index.html в Parser3, handler'ы)
- Отлаживаешь ошибки в Parser3 коде
- Строишь SQL-запросы в контексте Parser3
- Работаешь с Telegram webhook'ами, cron-задачами, email'ами, API в Parser3
- Объясняешь архитектуру Parser3 проекта (MAIN, auto.p, загрузка файлов)

## Inputs to identify

- **Тип задачи**: новый код / правка существующего / отладка / объяснение
- **Контекст**: webhook, cron, web-страница, API endpoint, admin
- **Используемые классы**: table, hash, string, json, date, curl, mail, file
- **Среда**: есть ли `^server{}`, `^alsSite`, `$TABLE_NAME`, доступ к БД

## Procedure

1. Прочитай релевантный reference-файл перед ответом (см. Reference map ниже)
2. Учти критически важные особенности языка (секция Gotchas)
3. Пиши код в стиле проекта — не привноси чужие паттерны из Python/JS
4. Проверь: правильный тип скобок, правильный тип присвоения, `def` для проверки наличия

## Reference map

- Read `references/patterns/telegram.md` when user asks about Telegram webhooks, bot handlers, sendMessage, callback_query, inline keyboard, MarkdownV2, or Telegram API
- Read `references/patterns/sql.md` when user asks about SQL queries, ^table::sql, ^void:sql, ^int:sql, taint, IN lists, or ON DUPLICATE KEY
- Read `references/patterns/cron.md` when user asks about cron jobs, scheduled tasks, @auto[], graceful stop, or background processes
- Read `references/patterns/api.md` when user asks about HTTP requests, curl, JSON endpoints, file::load, or API integrations
- Read `references/patterns/email.md` when user asks about sending email, mail class, HTML letters, attachments, or email queues
- Read `references/language.md` for operators and syntax: `^eval`, `^if`, `^switch`, `^while`, `^for`, `^try`/`^throw`, `^break`, `^continue`, `^return`, `^untaint`, `^taint`, `^process`, `^connect`/SQL drivers, `^cache`, `^rem`, `^syslog`, variable/parameter syntax, void/int/double, MAIN loading, HTTP file loading options, system error types
- Read `references/classes/data.md` for data classes: string, table, hash, hashfile, array
- Read `references/classes/io.md` for I/O classes: file, curl, mail, inet
- Read `references/classes/web.md` for web classes: cookie, form, request, response
- Read `references/classes/util.md` for utility classes: date, json, math, regex, console, env, image, memory, reflection, status, xdoc, xnode

## Tools to prefer

- `Read` — перед правкой любого .p файла всегда читать его целиком
- `Bash` + grep — искать паттерны использования в существующих файлах проекта

## Tools to avoid

- Не угадывать синтаксис — лучше прочитать references/ чем писать по памяти

## Validation

- `$var[string]` — не `$var = string`
- `^method[]` — не `method()` или `method()`
- `^if(condition){...}` — тело в фигурных скобках, условие в круглых
- `^table::sql{SELECT...}` — SQL в фигурных скобках
- Нет `;` в конце строк
- `def $var` для проверки — не `$var != ''`

## Gotchas

- **`$` и `^` — разные вещи**: `$var` — значение, `^method[]` — вызов
- **Три типа скобок**: `(expr)` — арифм. выражение, `[code]` — вычисляется до вызова, `{code}` — вычисляется внутри метода
- **Три типа присвоения**: `$var[string]`, `$var(2+2)`, `$var{lazy code}`
- **MAIN загружается снизу вверх**: конфиг → auto.p корень → auto.p по пути → файл. `@auto[]` каждого уровня выполняется при загрузке страницы — `^use[]` внутри handler'а НЕ выполняет `@auto[]` подключаемого файла
- **`$result`** — неявная переменная результата метода. `^return[value]` = записать в `$result` и остановить
- **`def` vs пустота**: пустая строка — не определена, пустой хеш — не определён, пустая таблица — не определена, `0` — определён
- **Имя переменной заканчивается** перед: пробелом, `;`, `]`, `}`, `)`, `"`, `<`, `>`, `+`, `*`, `/`, `%`, `&`, `|`, `=`, `!`, `'`, `,`, `?`. Для суффиксов: `${name}-suffix`
- **SQL в webhook-контексте**: использовать `^int:sql{}`, `^void:sql{}`, `^table::sql{}` — НЕ `^alsSite.oSql.*`
- **`^server{}`**: нужен в webhook index.html чтобы инициализировать соединение с БД
- **Регистр важен**: `$Name` и `$name` — разные переменные
- **Методы таблицы имеют приоритет над именами столбцов** (с v3.4.4): если столбец называется `hash`, `menu`, `sort` и т.п. — обращение через `$table.hash` вызовет метод, а не вернёт значение. Используй `$table.fields` чтобы получить строку как хеш
- **Regex: `^` → `^^`, `$` → `^$`**: в шаблонах для `^string.match[]` эти символы надо экранировать двойным `^`, иначе Parser интерпретирует их как свои
- **`match` без `'`**: столбцы `prematch`, `match`, `postmatch` вычисляются только при опции `'`; без неё они всегда пустые
- **`^array.delete()` оставляет дырку**: на месте удалённого элемента остаётся неинициализированный слот. Чтобы сдвинуть элементы — использовать `^array.remove()`
- **`^table.locate()` не сдвигает указатель при неудаче**: если элемент не найден, текущая строка таблицы остаётся прежней — молча
- **`^table.hash[]` падает при дублях ключей**: по умолчанию дублирующиеся значения ключевого столбца — ошибка. Добавить `$.distinct(1)` или `$.type[table]`
- **Taint — отложенная пометка, не трансформация**: `^taint[]` лишь помечает текст, преобразование происходит при выводе (в браузер, SQL-серверу, в файл, в письмо). Код разработчика помечен `optimized-as-is` (свёртка пробелов), данные извне (form, DB, file, cookie) — «грязные» (HTML-escaping). `^taint[as-is][$var]` нужен когда надо вывести HTML из базы от доверенного источника
- **Пробелы в коде разработчика сжимаются**: несколько пробельных символов подряд схлопываются в первый из них перед выдачей. Для `<pre>` или точного пробела — `^taint[as-is][...]`
- **SQL-преобразование работает только внутри `^connect{}`**: `^taint[sql]` применится только если вывод происходит внутри блока `^connect[]{}`, иначе преобразование не выполняется
- **Дата — дробное число суток, не секунд**: `$date1 - $date2` даёт дни (с дробью). Неделя назад — `$now - 7`, а не `$now - 604800`
- **`locals` в заголовке метода делает все переменные локальными**: `@method[params][locals]` — после этого bare `$name` не достаёт поля класса/объекта, нужно `$self.name`
- **`$result` — всё или ничего**: метод должен либо всегда возвращать через `$result`, либо никогда. Присвоение в одной ветке `^if` без присвоения в другой даёт непредсказуемый результат
