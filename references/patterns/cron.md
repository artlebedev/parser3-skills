# Cron — паттерны

## Сессия для graceful stop и защиты от параллельного запуска

Файл-маркер: cron создаёт файл при старте, проверяет его наличие в цикле. Удаление файла — сигнал к остановке. Предотвращает параллельный запуск.

```parser3
@create_session[name][locals]
$now[^date::now[]]
$s[^now.sql-string[]]
$s[^s.match[\D+][g]{_}]          # "2024-01-15 12:00:00" → "2024_01_15_12_00_00"
$dir[/data/sessions/$name]
$list[^file:list[$dir]]
^list.menu{^file:delete[$dir/$list.name]}   # удаляем старые сессии
$result[$dir/$s]
$e[ ]
^e.save[$result]                  # создаём файл-маркер

@in_session[]
$result(-f $MAIN:SESSION_FILE)    # true если файл существует
```

Использование в cron-скрипте:
```parser3
@auto[]
$MAIN:SESSION_FILE[^create_session[my_cron]]

@main[]
^if(!^in_session[]){^return[]}

^list.menu{
    ^break(!^in_session[])        # выходим если файл удалён
    # ... обработка элемента
    ^if(!^in_session[]){^return[]}  # проверка после тяжёлых операций
}
```

## Управление памятью в длинных циклах

Parser3 не освобождает системную память процесса, но `^memory:compact[]` освобождает внутренние структуры:

```parser3
@main[]
^memory:auto-compact(3)           # автоматическая сборка мусора (уровень 0-5)

^list.menu{
    # создаём тяжёлые объекты...
    $content[^SomeClass::create[...]]

    ^if(^list.line[]%10 == 0){
        $content[]                # явно освобождаем
        ^memory:compact[]         # принудительная сборка
    }
}
```

## Логирование в файл

```parser3
@log[text][locals]
$now[^date::now[]]
$line[^now.sql-string[]^#09$text^#0A]
^line.save[append;/cron/log.p]

# использование
^log[начало обработки]
^log[обработано: $count]
```

## Паттерн пакетной отправки email

Очередь в БД → cron обходит и шлёт, помечая отправленные:

```parser3
@cron_send_emails[_email][locals]
$list[^table::sql{
    SELECT q.id, q.db_id, u.email
    FROM email_queue AS q, users_details AS u
    WHERE q.dt_send is null AND u.id = q.user_id
    ^if(def $_email){AND u.email = '$_email'}
}]
^list.menu{
    ^break(!^in_session[])
    $sHtml[^buildEmailHtml[$list.db_id]]
    ^mail:send[
        $.to[$list.email]
        $.subject[$subject]
        $.from[Название <noreply@example.com>]
        $.html[
            $.value{^sHtml.base64[]}
            $.content-transfer-encoding[base64]
        ]
    ]
    ^void:sql{update email_queue set dt_send = now() where id = $list.id}
    ^if(^list.line[]%10 == 0){^memory:compact[]}
}
```

## Ожидание сигнала через файл-триггер

```parser3
^while(^in_session[]){
    ^if(-f '/cron/data/my_trigger'){
        ^file:delete[/cron/data/my_trigger]
        ^do_work[]
    }{
        ^sleep(2)
    }
}
```
