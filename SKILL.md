---
name: parser3
description: "Use this skill when writing, reading, debugging, or reviewing Parser3 code — server-side templating language by Art. Lebedev Studio. Trigger on: .p files, auto.p, Parser3 syntax questions, ^method[] calls, $variable assignments, @handler[] methods, ^if/^while/^for operators, ^table::sql queries, ^void:sql, ^int:sql, Parser3 errors, or any mention of Parser3, p3, or parser language."
version: 0.4.0
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
- **Используемые классы**: table, hash, string, json, date, curl, mail, file, memcached
- **Среда**: есть ли `^server{}`, `^alsSite`, `$TABLE_NAME`, доступ к БД

## Procedure

1. Прочитай релевантный reference-файл перед ответом (см. Reference map ниже)
2. Учти критически важные особенности языка (секция Gotchas)
3. Пиши код в стиле проекта — не привноси чужие паттерны из Python/JS
4. Проверь: правильный тип скобок, правильный тип присвоения, `def` для проверки наличия

## Reference map

- Read `references/patterns/telegram/SKILL.md` when user asks anything about Telegram — содержит процедуру, валидацию и gotchas специфичные для Telegram.p
- Read `references/patterns/sql.md` when user asks about SQL queries, ^table::sql, ^void:sql, ^int:sql, taint, IN lists, or ON DUPLICATE KEY
- Read `references/patterns/cron.md` when user asks about cron jobs, scheduled tasks, @auto[], graceful stop, or background processes
- Read `references/patterns/api.md` when user asks about HTTP requests, curl, JSON endpoints, file::load, or API integrations
- Read `references/patterns/email.md` when user asks about sending email, mail class, HTML letters, attachments, or email queues
- Read `references/language.md` for operators and syntax: `^eval`, `^if`, `^switch`, `^while`, `^for`, `^try`/`^throw`, `^break`, `^continue`, `^return`, `^untaint`, `^taint`, `^process`, `^connect`/SQL drivers, `^cache`, `^rem`, `^syslog`, variable/parameter syntax, void/int/double, MAIN loading, HTTP file loading options, system error types
- Read `references/classes/data.md` for data classes: string, table, hash, hashfile, array
- Read `references/classes/io.md` for I/O classes: file, curl, mail, inet
- Read `references/classes/web.md` for web classes: cookie, form, request, response
- Read `references/classes/util.md` for utility classes: date, json, math, regex, console, env, image, memory, reflection, status, xdoc, xnode
- Read `references/docs/classes/memcached/` when user asks about memcached, кэш в памяти, ^memcached::open, mget
- Read `references/docs/classes/bool/common.md` when user asks about bool type, булевые значения
- Read `references/docs/classes/junction/common.md` when user asks about junction, method reference, передача метода как параметра
- Read `references/docs/classes/void/` when user asks about void type, ^void:sql
- Read `references/docs/common/operators/common/sleep.md` when user asks about ^sleep, задержка, пауза в коде
- Read `references/docs/common/operators/common/user-operators.md` when user asks about пользовательских операторах
- Read `references/docs/common/constructions/` when user asks about синтаксисе классов, методов, параметров, объектов, наследовании
- Read `references/docs/addition/install/common/install-apache.md` when user asks about установке на Apache, mod_parser, httpd.conf, ParserConfig, AddHandler, LoadModule
- Read `references/docs/addition/install/common/install-cgi.md` when user asks about CGI-режиме, parser3.cgi
- Read `references/docs/addition/install/common/install-iis.md` when user asks about IIS, Windows-сервере
- Read `references/docs/addition/install/common/config-file.md` when user asks about конфигурационном файле auto.p, $SQL, $MAIL, $CLASS_PATH, настройке сайта
- Read `references/docs/addition/install/common/install.md` when user asks об общей установке Parser3, с чего начать
- Read `references/docs/addition/parts/common/mysql.md` when user asks о MySQL, соединении с MySQL, mysql://
- Read `references/docs/addition/parts/common/postgresql.md` when user asks о PostgreSQL, pgsql://
- Read `references/docs/addition/parts/common/sqlite.md` when user asks о SQLite, sqlite://
- Read `references/docs/addition/parts/common/oracle.md` when user asks об Oracle, odbc://
- Read `references/docs/addition/parts/common/connect.md` when user asks о ^connect, строке подключения к БД
- Read `references/docs/addition/parts/common/class-path.md` when user asks о CLASS_PATH, путях поиска классов, ^use[]
- Read `references/docs/addition/parts/common/pcre.md` when user asks о regex, PCRE, регулярных выражениях, флагах
- Read `references/docs/addition/parts/common/naming.md` when user asks о соглашениях именования, стиле кода Parser3
- Read `references/docs/classes/<class>/` for detailed API любого класса — если curated-файл не содержит нужной детали

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
- **Служебные символы в строках**: `^ $ ; @ ( ) [ ] { } " : #` имеют специальное значение; если символ нужен как текст, экранируй его через `^` или код символа (`^#XX`). Особенно важно: `;` внутри параметров метода/оператора разделяет параметры, поэтому в текстовом значении пиши `^;`, например `$s[пункт1^;пункт2]`. См. `stringlit.htm`.
- **Версия Parser важна**: класс `array` и синтаксис `$a[value;value;...]` доступны только с Parser 3.5.0. В проектах на версиях ниже 3.5.0 такая запись является синтаксической ошибкой. Если версия проекта неизвестна, не используй array-синтаксис без проверки версии или существующих паттернов проекта.
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
- **`hash::sql` использует первый столбец как ключ**: при `^hash::sql{SELECT id, name FROM table}` значение `id` станет ключом хеша, а не полем строки. Если внутри строки нужен `$item.id`, выбирай `SELECT id, id, name ...` или используй `table::sql`, если нужен обычный набор строк
- **Taint — отложенная пометка, не трансформация**: `^taint[]` лишь помечает текст, преобразование происходит при выводе (в браузер, SQL-серверу, в файл, в письмо). Код разработчика помечен `optimized-as-is` (свёртка пробелов), данные извне (form, DB, file, cookie) — «грязные» (HTML-escaping). `^taint[as-is][$var]` нужен когда надо вывести HTML из базы от доверенного источника
- **Пробелы в коде разработчика сжимаются**: несколько пробельных символов подряд схлопываются в первый из них перед выдачей. Для `<pre>` или точного пробела — `^taint[as-is][...]`
- **SQL-преобразование работает только внутри `^connect{}`**: `^taint[sql]` применится только если вывод происходит внутри блока `^connect[]{}`, иначе преобразование не выполняется
- **Дата — дробное число суток, не секунд**: `$date1 - $date2` даёт дни (с дробью). Неделя назад — `$now - 7`, а не `$now - 604800`
- **`locals` в заголовке метода делает все переменные локальными**: `@method[params][locals]` — после этого bare `$name` не достаёт поля класса/объекта, нужно `$self.name`
- **`$result` — всё или ничего**: метод должен либо всегда возвращать через `$result`, либо никогда. Присвоение в одной ветке `^if` без присвоения в другой даёт непредсказуемый результат
- **`^table.csv-string[]` существует**: метод для сериализации таблицы в CSV — не изобретать велосипед через `^table.menu`
- **`junction` — ссылка на метод**: позволяет передать метод как параметр и вызвать его позже. Создаётся через `^reflection:method[]`
- **`memcached` — отдельный класс**: для кэширования в памяти. Не путать с `^cache[]` (файловый кэш). `^memcached::open[host:port]`
- **`{}` vs `()` у `.sort`/`^switch`/`^case` — переключатель режима сравнения**: `{}`/`[]` — строка (алфавит, "10" раньше "2"), `()` — число. `^table.sort{$t.field}` vs `^table.sort($t.field)`. Перепутать легко: компилируется без ошибки, просто даёт неверный порядок
- **`number.format`** — исключение при попытке использовать нечисловую строку как число. Частый источник: `$flag[yes]` (через `[]`) затем `^if($flag)` — падает, т.к. "yes" не число. Для флагов, которые пойдут в `^if()`/`^switch()`, присваивай через `()`: `$flag(def $x)`, `$flag(1)`
- **Chaining методов не поддерживается вообще, для любых типов**: `^method1[].method2[]` — `.method2[]` после закрывающей `]` первого вызова считается просто текстом, а не вызовом метода. Всегда: `$x[^method1[]]` → `^x.method2[]`, не `^method1[].method2[]`
- **`^file:list[dir][$.stat(true)]`'s даты (`mdate`/`adate`/`cdate`) — это сырые unix-timestamp (секунды), а не объекты `date`**: методы вроде `.iso-string[]` на них не работают напрямую. Если нужен настоящий `date`-объект на файл — используй `^file::stat[fullpath]`, его `.mdate` (после сохранения в переменную) уже полноценная дата
- **`$exception.handled(true)` нужно ставить явно в каждом `^try{}{}`, который должен реально погасить ошибку**: без этого флага исключение считается необработанным и уходит в `@unhandled_exception[]`, даже если catch-блок успел построить свой ответ — этот ответ будет отброшен/перезаписан
- **`@OPTIONS partial`** позволяет "переоткрыть" класс из нескольких файлов: если позже загруженный `^use[]`'ом файл объявляет `@CLASS` с тем же именем и `@OPTIONS partial`, его методы ДОБАВЛЯЮТСЯ к уже загруженному классу, а методы с совпадающим именем — переопределяют более ранние
- **`^table.line[]`** — номер текущей строки при итерации (`.menu{}`), считая с 1. Удобно для "это первая строка или нет" без отдельного счётчика
- **`$a$b` (две bare-переменные подряд без разделителя) не конкатенируется — склеивается в одно имя переменной**: `-` и `$` не входят в список символов, завершающих имя, поэтому `$a$b`/`$a-$b` молча ищут несуществующую переменную `a$b`/`a-` и дают пустую строку. Разделяй: пробел (`$a $b`), явные скобки имени (`${a}${b}`), любой символ-терминатор (`$a,$b`)
