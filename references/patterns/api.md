# API — паттерны

## JSON API endpoint

```parser3
@main[]
$data[^json:parse[^taint[as-is][$request:body]]]

# обработка...

$response:content-type[application/json]
$response:body[^json:string[
    $.ok(true)
    $.data[$result]
]]
```

## Стандартный ответ об ошибке

```parser3
$response:content-type[application/json]
$response:body[^json:string[
    $.ok(false)
    $.error[не авторизован]
]]
^return[]
```

## curl — HTTP запрос к внешнему API

```parser3
$f[^curl:load[
    $.url[https://api.example.com/endpoint]
    $.charset[UTF-8]
    $.response-charset[UTF-8]
    $.timeout(30)
    $.ssl_verifypeer(0)
    $.httpheader[
        $.Authorization[Bearer $token]
        $.Content-type[application/json]
        $.Accept[application/json]
    ]
]]
$result[^json:parse[^taint[as-is][$f.text]]]
```

POST запрос:
```parser3
$f[^curl:load[
    $.url[https://api.example.com/endpoint]
    $.post(1)
    $.postfields[^json:string[$data]]
    $.httpheader[$.Content-type[application/json]]
    $.timeout(20)
    $.ssl_verifypeer(0)
]]
$result[^json:parse[^taint[as-is][$f.text]]]
```

## file::load — HTTP запрос с опциями

```parser3
$f[^file::load[text;https://api.example.com/data;
    $.method[POST]
    $.timeout(10)
    $.headers[
        $.Authorization[Bearer $token]
        $.Content-type[application/json]
    ]
    $.body[^json:string[$payload]]
    $.charset[UTF-8]
]]
```

## Проверка среды (prod/dev/local)

```parser3
# через файл-маркер на сервере
@isProductionSite[]
$result(-f '/../cgi/is_production')

# через SERVER_NAME
$isLocal($env:SERVER_NAME eq 'localhost' || ^env:SERVER_NAME.pos[.local] > -1)
$isLocal($env:REMOTE_ADDR eq '127.0.0.1')

# паттерн: в dev пишем в файл вместо реальной отправки
^if(^isProductionSite[]){
    ^mail:send[$params]
}{
    $s[^json:string[$params]]
    ^s.save[/tmp/email_debug.json]
}
```

## Кэш-бастинг статики

```parser3
@m_time[src]
$result[$src]
^if(-f $src){
    $stat[^file::stat[$src]]
    $result[$src?^stat.mdate.unix-timestamp[]]
}
```

Использование в HTML:
```parser3
<script src="^m_time[/js/app.js]"></script>
<link rel="stylesheet" href="^m_time[/css/main.css]">
```

## ^server{} — только HTTP-контекст

```parser3
^server{
    # не выполнится в console-режиме (cron, cli)
    $alsLogin[^alsLogin::create[]]
    $userData[^getUserData[]]
}
```
