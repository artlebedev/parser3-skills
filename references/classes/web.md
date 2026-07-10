# Parser3 Web Classes

## cookie

```parser3
$cookie:name                  # read old or newly set cookie

$cookie:name[value]           # set cookie for 90 days

$cookie:name[
    $.value[value]
    $.expires[VALUE]          # 'session', a date, or number of days (fractions allowed)
    $.secure(true)
    $.domain[domain name]
    $.httponly(true)
]
# if expires is a date, it is converted to "Sun, 25-Aug-2002 12:03:45 GMT"
# if value is empty and expires is omitted, Parser deletes the cookie

$cookie:empty[
    $.value[]
    $.expires(365)
]                             # set an empty cookie explicitly

$cookie:fields                # hash with all cookies
```

## form

The first element with the same name is taken from GET, then from POST.

```parser3
$form:field                   # string or file

$form:nameless                # value from nameless parameter "?value&..." or "...&value"

$form:qtail                   # string after second "?xxxxx" if no ',' [imap]

$form:fields                  # hash with all form fields

$form:elements.field          # array with all values of the field (both string and file)

$form:tables.field            # table with one column "field" containing all values for multiple entries

$form:files.field             # hash with file-type field values; keys: 0, 1, ...; value: file

$form:imap                    # hash with keys 'x' and 'y' for ?1,2 suffixes (server-side image map)
```

## request

```parser3
# URL: https://site.name/a%20b/?name=some%20value

$request:query               # name=some%20value
$request:uri                 # /a%20b/?name=value
$request:path                # /a b/

$request:document-root
# directory relative to which paths are considered in parser
# default = $env:DOCUMENT_ROOT

$request:argv                # hash with command-line parameters; keys 0, 1, ... (0 = processed file)

$request:charset
# source document encoding; used in upper/lower and match[][i]
# WARNING: must set $request/$response:charset before using form class fields

$request:method              # GET|POST|PUT

$request:body                # POST-request body as text
$request:body-file           # POST-request body as file
$request:body-charset        # POST-request encoding

$request:headers             # hash with request headers (without HTTP_ prefix)
```

## response

```parser3
$response:field[value]       # set response header field
$response:field              # read previously set header field

# Value can be string or hash:
#   $value[abc]          → field: abc
#   $attribute[zzz]      → field: abc; attribute=zzz
# String or date; date converts to "Sun, 25-Aug-2002 12:03:45 GMT"

$response:headers            # accumulated response fields

$response:body[DATA]         # replace standard response
$response:download[DATA]     # replace standard response; browser prompted to download

$response:status             # HTTP status

^response:clear[]            # forget all set response fields

$response:charset
# client encoding:
# 1) $form: fields transcoded from browser using this charset
# 2) document transcoded to this charset before sending to browser
# 3) URI language text transcoded to this charset
# does NOT add anything to content-type (set manually if needed)
# WARNING: must set $request/$response:charset before using form class fields
```
