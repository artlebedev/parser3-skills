# SQL — паттерны

## Безопасная вставка строк

Всегда экранировать пользовательский ввод через `^taint[sql]`:

```parser3
^void:sql{INSERT INTO t SET
    name = '^taint[sql][$name]',
    email = '^taint[sql][$email]',
    data = '^taint[sql][^json:string[$hash]]'
}
```

Числа не нужно экранировать если это действительно числа:
```parser3
^void:sql{UPDATE t SET count = $count WHERE id = $id}
```

## IN список из таблицы или хеша

```parser3
# из таблицы через menu
WHERE id IN (^list.menu{$list.id}[,])

# из таблицы через foreach (доступ к значению)
WHERE uid IN (^list.foreach[;v]{'$v.uid'}[,])

# из хеша
WHERE tag_id IN (^h.foreach[k;]{'$k'}[,])

# из вложенного хеша (хеш хешей)
AND type NOT IN (
    ^MAIN:config.gift.foreach[k;]{'$k'}[,]
)
```

## JSON из БД — снимать taint при парсинге

Строки из БД помечены как «грязные», `^json:parse` требует чистые:

```parser3
$json_str[^string:sql{SELECT data FROM t WHERE id = '$id'}]
$data[^json:parse[^taint[as-is][$json_str]]]
```

## ON DUPLICATE KEY UPDATE

```parser3
^void:sql{INSERT INTO t SET key = '$key', value = '$value'
    ON DUPLICATE KEY UPDATE value = '$value'}

# только вставить, при дубле ничего не делать
^void:sql{INSERT IGNORE INTO t SET key = '$key'}

# обновить только дату
^void:sql{INSERT INTO t SET key = '$key', dt = now()
    ON DUPLICATE KEY UPDATE dt = now()}
```

## table::sql с join через Parser

Иногда проще сделать два запроса и объединить через `^table.join[]`:

```parser3
$base[^table::sql{SELECT id, tg_chat_id FROM users_details WHERE ...}]
^base.join[^table::sql{
    SELECT 0 AS id, chat_id AS tg_chat_id FROM users_tg WHERE ...
    ^if($base){
        AND chat_id NOT IN (^base.menu{$base.tg_chat_id}[,])
    }
}]
```

## hash::sql — хеш из запроса

```parser3
# ключ = первый столбец, значение = строка второго столбца
$h[^hash::sql{SELECT email, name FROM users}[$.type[string]]]
$name[$h.[$email]]

# ключ = первый столбец, значение = хеш остальных столбцов (по умолчанию)
$h[^hash::sql{SELECT id, email, name FROM users}]
$email[$h.[$id].email]

# с distinct — не падать при дублях ключей
$h[^hash::sql{SELECT tag_id FROM tags}[$.distinct(true)]]
```

## Динамические условия в запросе

```parser3
$users[^table::sql{
    SELECT u.id, u.email
    FROM users AS u
    WHERE
        u.is_active = 1
        ^if(def $email){AND u.email = '$email'}
        ^if($minAge){AND u.age >= $minAge}
        ^if($ids){AND u.id IN (^ids.foreach[k;]{'$k'}[,])}
    ORDER BY u.id
    LIMIT ^if(def $limit){$limit}{100}
}]
```

## Обновление нескольких записей по id из цикла

```parser3
# вместо UPDATE в каждой итерации — батч после цикла
^void:sql{UPDATE email_queue SET dt_send = NOW()
    WHERE id IN (^list.foreach[;v]{'$v.id'}[,])}
```
