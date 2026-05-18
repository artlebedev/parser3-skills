# Parser3 I/O Classes

## file

```parser3
# Uploaded file (from POST):
$uploaded_file_from_post.name
$uploaded_file_from_post.size
$uploaded_file_from_post.text

^file.save[text|binary;filename[;$.charset[encoding] $.append(false)]]
^file:delete[filename]
^file:find[filename][{if not found}]

^file:list[path[;pattern-string|pattern-regex]]
# table with columns: name  dir

^file:list[path;$.filter[pattern-string|pattern-regex] $.stat(true)]
# table with columns: name  dir  size  [mca]date

^file::load[text|binary;big.zip[;domain_press_release_2001_03_01.zip][;options]]
^file::create[text|binary;filename;data]
^file::create[text|binary;filename;data[;$.charset[charset] $.content-type[...]]]
^file::create[string-or-file-content[;$.name[name] $.mode[text|binary] $.content-type[...] $.charset[...]]]

$loaded_file.size
$loaded_or_created_file.mode    # text|binary

^file::stat[filename]
$stated_or_loaded_file.size  .adate  .mdate  .cdate

^file::cgi[[text|binary;]filename[;env hash +options[;1cmd[;2line[;3ar[;4g[;5s]]]]]]
# any argument can be string or array of strings
# returns: $fields (split headers), $status, $stderr

^file::exec[[text|binary;]filename[;env hash[;1cmd[;2line[;3ar[;4g[;5s;...up to 50 args]]]]]]]
# any argument can be string or array of strings
# options: $.stdin[text|file]  (empty = disable auto-passing of HTTP-POST data)

^file:move[oldfilename;newfilename]
# rename or move directories [win32: same disk only]
# dest directories created with 775 permissions
# source directory removed if empty after move

^file:copy[filename;copy_filename[; $.append(1) ]]   # files only

^file:lock[filename]{code}
# creates file if needed, locks, executes code, unlocks

# Path utilities:
^file:dirname[/a/some.tar.gz]    # = /a
^file:dirname[/a/b/]             # = /a
^file:basename[/a/some.tar.gz]   # = some.tar.gz
^file:basename[/a/b/]            # = b
^file:justname[/a/some.tar.gz]   # = some.tar
^file:justext[/a/some.tar.gz]    # = gz
^file:fullpath[a.gif]            # /some/page.html → /some/a.gif

^file.sql-string[]               # correctly escaped string for use inside ^connect queries

^file::sql{query}[[ $.name[filename_for_download] $.content-type[user content-type] ]]
# query result: one row
# first column = data; second = filename; third = content-type

^file.base64[ $.pad(bool) $.wrap(bool) $.url-safe(bool) ]    # encode
^file:base64[filename[; $.pad(bool) $.wrap(bool) $.url-safe(bool) ]]    # encode file
^file::base64[encoded string[; $.pad(bool) $.strict(bool) $.url-safe(bool) ]]   # decode
^file::base64[mode;filename;encoded string[; $.content-type[...] $.pad(bool) $.strict(bool) $.url-safe(bool) ]]  # decode

^file:crc32[filename]            # CRC32 of file
^file.crc32[]                    # CRC32 of object

^file.md5[]
^file:md5[filename]
# returns digest: 16 bytes as string, hex lowercase contiguous
```

## curl

```parser3
^curl:load[[
    $.url[http://URL]
    $.timeout(N)
    $.ssl_verifypeer(0)
    $.mode[text|binary]
    # any libcurl option in lowercase without CURLOPT_ prefix
]]
# downloads a file from remote server; can be called multiple times per session

^curl:options[[
    $.library[libcurl.so.4]
    $.charset[UTF-8]
    # ...
]]
# subsequent ^curl:load calls inherit these options
# must set library path before using curl

^curl:session{code}
# creates a cURL session; set common options, perform multiple downloads

^curl:info[name]   # information about last request (single value)
^curl:info[]       # information about last request (hash)
^curl:version[]    # cURL library version
```

## mail

### Received message structure

```parser3
$mail.received=MESSAGE:
    .from
    .reply-to
    .subject
    .date                       # class date
    .message-id
    .raw[ .RAW_USER_HEADER_FIELD ]
    $.{text|html|file#}[        # numbered: text, text2, ..., file, file2, ...
        $.content-type[
            $.value[text/plain]
            $.charset[windows-1251]
            $.USER_DEFINED_HEADER_FIELD
        ]
        $.description
        $.content-id
        $.content-md5
        $.content-location
        .raw[ .RAW_USER_HEADER_FIELD ]
        $.value[string|FILE]
    ]
    $.message#[MESSAGE]         # nested messages: message, message2, ...
```

### Sending mail (modern format)

```parser3
^mail:send[
    $.options[-odd]             # unix: string added to sendmail command; win32: ignored
    $.charset[encoding of headers and text blocks]
    $.any-header-field
    $.text[string]
    $.text[
        $.any-header-field
        $.value[string]
    ]
    $.html{string}
    $.html[
        $.any-header-field
        $.value{string}
    ]
    $.file#[FILE]
    $.file#[
        $.any-header-field
        $value[FILE]
    ]
]
# if charset specified, email is transcoded to that charset
# content-type.charset does NOT affect transcoding
# after part name, # number can follow
```

### Sending mail (simple)

```parser3
^mail:send[
    $.charset[windows-1251]
    $.content-type[$.value[text/plain] $.charset[windows-1251]]
    $.from["vasya" <vasya@design.ru>]
    $.to["petya" <petya@design.ru>]
    $.subject[subject]
    $.body[ text ]
]
```

### Sending mail (multipart)

```parser3
^mail:send[
    $.from["vasya" <vasya@design.ru>]
    $.to["petya" <petya@design.ru>]
    $.subject[subject]
    $.body[
        $.text[
            $.charset[windows-1251]
            $.content-type[$.value[text/plain] $.charset[windows-1251]]
            $.body[words]
        ]
        $.file[
            $.value[^file::load[my beloved.doc]]
            $.name[my beloved.doc]
            $.format[base64]
        ]
        $.file2[
            $.value[^file::load[my beloved.doc]]
            $.name[my beloved.doc]
        ]
    ]
]
# body as hash = multipart; text parts first, then attachments
# for multipart: do NOT specify content-type
# part name starts with "text" = text block; "file" = attachment
```

### Mail transport configuration

```parser3
# Unix:
$MAIL.sendmail[command]
# if not set, checks /usr/sbin/sendmail or /usr/lib/sendmail and runs with "-t"

# Windows:
$MAIL.SMTP[smtp.domain.ru]
```

## inet

```parser3
^inet:ntoa(long)             # integer IP → dotted string
^inet:aton[IP]               # dotted string → integer IP
^inet:name2ip[name][[ $.ipv[4|6|any] $.table(true) ]]   # DNS lookup
^inet:ip2name[ip][ $.ipv[4|6|any] ]                     # reverse DNS lookup
^inet:hostname[]             # current host name
```
