# Email — паттерны

## HTML письмо через base64

Стандартный способ отправки HTML-письма. Base64 нужен чтобы корректно передать HTML с кириллицей и спецсимволами:

```parser3
^mail:send[
    $.to[$email]
    $.subject[$subject]
    $.from[Имя Отправителя <noreply@example.com>]
    $.html[
        $.value{^sHtml.base64[]}
        $.content-transfer-encoding[base64]
    ]
]
```

## Письмо с unsubscribe header

```parser3
^mail:send[
    $.to[$email]
    $.subject[$subject]
    $.from[$from]
    $.[list-unsubscribe][$unsubscribe_url]
    $.html[
        $.value{^sHtml.base64[]}
        $.content-transfer-encoding[base64]
    ]
]
```

## Письмо с вложением

```parser3
^mail:send[
    $.to[$email]
    $.subject[$subject]
    $.from[$from]
    $.html[
        $.value{^html.base64[]}
        $.content-transfer-encoding[base64]
    ]
    $.file[
        $.value[^file::load[binary;/path/to/file.pdf]]
        $.name[document.pdf]
        $.format[base64]
    ]
]
```

## Очередь email через БД

```parser3
# создание очереди
^users.menu{
    ^void:sql{INSERT INTO email_queue SET
        uid = '^math:uuid[]',
        db_id = '$db_id',
        user_id = $users.id
    }
}

# отправка из cron
$list[^table::sql{
    SELECT q.id, u.email, q.db_id
    FROM email_queue AS q, users_details AS u
    WHERE q.dt_send IS NULL AND u.id = q.user_id
}]
^list.menu{
    ^break(!^in_session[])
    $sHtml[^buildHtml[$list.db_id;$list.email]]
    ^mail:send[
        $.to[$list.email]
        $.subject[$subject]
        $.from[$from]
        $.html[$.value{^sHtml.base64[]} $.content-transfer-encoding[base64]]
    ]
    ^void:sql{UPDATE email_queue SET dt_send = NOW() WHERE id = $list.id}
    ^if(^list.line[]%10 == 0){^memory:compact[]}
}
```

## Валидация email

```parser3
# простая проверка через pos
^if(^email.pos[@] > -1){
    $email[^email.trim[]]
    $email[^email.lower[]]
    # email валиден
}

# домен из email
$domain[^email.mid(^email.pos[@])]
```
