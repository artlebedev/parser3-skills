# Parser3 Language Reference

## Operators

### ^eval
```parser3
^eval(expression)[format]
```
Evaluates arithmetic/logical expression. Supports:
- `#comments` — until end of line or closing parenthesis; nested parentheses allowed inside comments
- `|` bitwise XOR, `||` logical XOR, `~` bitwise negation, `\` integer division (`10\3=3`)
- `def` — checks if defined (empty string/table/hash = not defined)
- `eq ne lt gt le ge` — string comparison
- `in "/dir/"` — checks if current document is in directory (no expressions allowed inside)
- `is 'type'` — checks type of left operand
- `-f` — file exists on disk, `-d` — directory exists on disk
- Quoted strings (single or double) are strings; unquoted text is a string until nearest whitespace
- Hex literals: `0xABC`

Operator priorities (high to low):
```
/* logical */
%left "!||"
%left "||"
%left "&&"
%left '<' '>' "<=" ">=" "lt" "gt" "le" "ge"
%left "==" "!=" "eq" "ne"
%left "is" "def" "in" "-f" "-d"
%left '!'

/* bitwise */
%left '!|'
%left '|'
%left '&'
%left '~'

/* numerical */
%left '-' '+'
%left '*' '/' '%' '\\'
%left '~'     /* unary negation */
```

Literals: `true`, `false`

### ^if
```parser3
^if(condition){then}{else}
^if(condition1){yes}[(condition2){yes}[(condition3){yes}[...]]]{no}
```
Supports unlimited additional conditions (elseif).

### ^switch
```parser3
^switch[value]{^case[var1[;var2...]]{action}^case[DEFAULT]{default action}}
```

### ^while
```parser3
^while(condition){body}[[delimiter]|{delimiter executed before each non-empty non-first body}]
```

### ^for
```parser3
^for[i](0;4){body}[[delimiter]|{delimiter executed before each non-empty non-first body}]
```

### ^try / ^throw
```parser3
^try{
    ...
    ^throw[sql.connect[;vasya[;mistaken]]]
    ^throw[
        $.type[sql.connect]
        $.source[vasya]
        $.comment[mistaken]
    ]
    ...
}{
    ^if($exception.type eq "sql"){
        $exception.handled(1|true)   # flag that exception is handled
        ...
    }
    ^switch[$exception.type]{
        ^case[sql;mail]{
            $exception.handled(1)
            # $exception.type = sql.connect
            # $exception.file  $exception.lineno  $exception.colno
            # $exception.source = vasya
            # $exception.comment = mistaken
        }
        ^case[DEFAULT]{
            # ^throw[$exception] re-throw — DON'T! It's the default behaviour!
        }
    }
}
```

### ^break / ^continue
```parser3
^break[]             # breaks the loop
^break(true|false)   # breaks if true

^continue[]          # breaks current iteration
^continue(true|false)
```

### ^return
```parser3
^return[]        # stops method execution
^return[value]   # assigns $result the value and stops
```

### ^untaint / ^taint / ^apply-taint
```parser3
^untaint[[as-is|file-spec|uri|http-header|mail-header|sql|js|json|parser-code|regex|xml|html|optimized-[as-is|xml|html]]]{code}
# default: as-is

^taint[[lang]][code]
# default: "just tainted, language unknown"

^apply-taint[[lang;]text]
# applies transformations; "indefinitely dirty" treated as lang, produces clean string
```

### ^process
```parser3
^process[[$caller.CLASS|$object|$CLASS:CLASS]]{string to be processed as code}[
    $.main[what to rename @main to]
    $.file[name of the file supposedly containing this text]
    $.lineno(line number in the file from where this text originated, can be negative)
]
^process..[path][what to rename @main to]
# by default methods are compiled into $self [in operator: $self=$MAIN:CLASS]
```

### ^connect (SQL)
```parser3
^connect[protocol://connection-string]{code with ^sql[...] calls}
```

Connection strings:
```
mysql://user:pass@{host[:port][, host[:port]]|[/unix/socket]}/database?
    ClientCharset=parser-charset&charset=UTF-8&timeout=3&compress=0&
    named_pipe=1&multi_statements=1&config_file=.my.cnf&config_group=parser3&autocommit=1
    # autocommit=0 triggers commit/rollback

pgsql://user:pass@{host[:port]|[local]}/database?
    client_encoding=win&datestyle=ISO&ClientCharset=parser-charset

odbc://DSN=dsn^;UID=user^;PWD=password^;ClientCharset=parser-charset

sqlite://DBfile?ClientCharset=parser-charset&autocommit=1
```

SQL drivers table (set in `auto.p`):
```parser3
$SQL[
    $.drivers[^table::create{protocol	driver	client
mysql	$prefix/libparser3mysql.so	libmysqlclient.so
pgsql	$prefix/libparser3pgsql.so	libpq.so
sqlite	$prefix/libparser3sqlite.so	sqlite3.so
odbc	parser3odbc.dll
}]
]
```

### ^rem
```parser3
^rem{}   # comment, removed at compile time
```

### ^syslog
```parser3
^syslog[ident;message[;info|warning|error|debug]]
```

### ^cache
```parser3
^cache[file](seconds){code}[{catch code}]       # relative time; 0 = don't cache, remove existing
^cache[file][expires date]{code}[{catch code}]  # absolute time
^cache[file]                                    # deletes file (no error if missing)
^cache(seconds)
^cache[expires date]    # signals upper-level ^cache to reduce to these seconds/expires
^cache[]                # returns current expires date
```
In catch code: `$exception.handled[cache]` to mark exception as handled.

## Variables and Methods

```parser3
$result   # local variable in every method; assign to set the method's return value
$caller   # parent stack frame; can write to its local variables

# use / @USE — searches for and includes a file:
# 1. Path starts with / → from web root
# 2. Relative to current directory
# 3. Relative to $MAIN:CLASS_PATH (global string or table of paths), bottom-up

# $CHARSETS[$.name[filename]] — defines which characters are letters, digits, etc., and Unicode
# Format: tab-delimited, header: char  white-space  digit  hex-digit  letter  word  lowercase  unicode1  unicode2
# UTF-8 is always available and is the default encoding for request and response
# Encoding name is case-insensitive
```

## Syntax

```parser3
$name[new value]
$name(arithmetic expression of new value)
$name{code of new value}
$name           # variable value (whitespace or ${name}something)
^name params    # method call
$name.CLASS     # class of the value
$name.CLASS_NAME  # name of the class
$name[$.key[] () {}]    # hash constructor with element $name.key
^method[$.key[] () {}]  # hash parameter constructor
$CLASS.name             # class variable access
```

Name ends before: `space tab linefeed ; ] } ) " < > + * / % & | = ! ' , ?`
- You can do `$name,aaaa`
- For `-` after name: use `${name}-`
- In expressions, `+` and `-` are additional name boundaries

Compound object access:
```parser3
$name.subname   # subname: a string | $variable | string$variable | [code]

$hash[$.age(88)]
$get[$.field[age]]
^hash.[$get.field].format{%05d}
```

Parameters:
```
parameters := one or more parameters
parameter :=
    (arithmetic expression)  evaluated multiple times inside the call
  | [code]                   evaluated once before the call
  | {code}                   evaluated zero or many times inside the call
';' allowed — multiple parameters in a single bracket
```

## void, int, double

```parser3
# void — all string class methods available; result behaves as empty string
^void:sql{query without result}{$.bind[see table::sql]}

# int / double
^name.int[]
^name.double[]
^name.bool[]  ^name.bool(true|false)
^name.inc(how much +)
^name.dec(how much -)
^name.++[]    # output value, then increment by 1
^name.--[]    # output value, then decrement by 1
^name.mul(how much *)
^name.div(how much /)
^name.mod(how much %)
^name.format[format]
^int/double:sql{query}[[$.limit(2) $.offset(4) $.default{0} $.bind[see table::sql]]]
# query result must be one column / one row
```

## MAIN

`MAIN` is the class automatically assembled from configuration `auto.p`, a chain of `auto.p` files down the directory tree, and the requested document.

```
Configuration auto.p location:
  cgi:    CGI_PARSER_SITE_CONFIG env var, or next to parser binary
  isapi:  windows directory
  apache: ParserConfig directive (can be in .htaccess)

Traversal: DOCUMENT_ROOT/ → down through directories → processed file's directory
Last loaded = MAIN; previous files have no names; each becomes parent of the prior.
```

Execution flow:
1. `@main[]` is called
2. Result passed to `@postprocess[data]` if `$data` is string
3. Result returned to user

```parser3
# Error handler (define in MAIN):
@unhandled_exception[exception;stack]
    $exception.type     # "type of problem"
    $exception.file  $exception.lineno  $exception.colno
    $exception.source   # line that caused the problem
    $exception.comment  # English comment
    # stack: table[file, line, name] in reverse call order
```

### HTTP file loading options

When loading via HTTP in `file::load`, `table::load`, `xdoc::load`:
```parser3
^file::load[http://domain/document][
    $.method[GET|POST|HEAD]
    $.timeout(3)               # seconds, default=2
    $.cookies[ $.name[value] ]
    $.headers[ $.field[value] ]
    $.enctype[multipart/form-data]
    $.form[
        $.field1[string]
        $.field2[^table::create{one_column_only^#0Avalue1^#0Avalue2}]
        $.field3[file]
    ]
    $.body[string|file]
    $.charset[default encoding]         # overridden by server content-type charset
    $.response-charset[encoding]        # NOT overridden by content-type charset
    $.user[user]
    $.password[password]
    $.any-status(1)                     # disable http.status error for status != 200
]
# file::load writes response fields: FIELD:value (uppercase), tables hash
```

## System Error Types

| Type | Example trigger | Description |
|------|----------------|-------------|
| `parser.compile` | `^test[}` | Compilation error (unmatched bracket, ...) |
| `parser.runtime` | `^if(0).` | Wrong parameter count/type |
| `number.zerodivision` | `^eval(1/0)` | Division by zero |
| `number.format` | `^eval(abc*5)` | Number format error |
| `file.lock` | | Shared/exclusive lock error |
| `file.missing` | `^file:delete[delme]` | File not found |
| `file.access` | `^table::load[.]` | No permissions |
| `file.read` | `^file::load[...]` | Error reading file |
| `file.seek` | | Seek failed |
| `file.execute` | `^file::cgi[...]` | Bad CGI header / can't execute |
| `image.format` | `^image::measure[index.html]` | Not gif/jpg |
| `sql.connect` | `^connect[mysql://bad:pass@host/db]{}` | Not found/timeout |
| `sql.execute` | `^void:sql{select bad}` | SQL syntax error |
| `sql.duplicate` | | Duplicate key |
| `sql.access` | | SQL access denied |
| `sql.missing` | | SQL record not found |
| `xml` | `^xdoc::create{<forgot?>}` | XML/XSLT error |
| `smtp.connect` | | SMTP not found/timeout |
| `smtp.execute` | | SMTP communication error |
| `email.format` | `hren tam@null.ru` | Bad email format |
| `email.send` | `$MAIL.sendmail[/shit]` | sendmail not executable |
| `http.host` | `^file::load[http://notfound/]` | Host not found |
| `http.connect` | `^file::load[http://not_accepting/]` | Host found, no connection |
| `http.timeout` | `^file::load[http://host/doc]` | Load timed out |
| `http.response` | `^file::load[http://ok/]` | Bad response |
| `http.status` | `^file::load[http://ok/]` | Status != 200 |
| `date.range` | `^date::create(10000;1;1)` | Date out of valid range |

## Miscellaneous

```parser3
# If $SIGPIPE(1) is defined in MAIN:
# when processing is interrupted by the user, a message is written to parser3.log

# If a method explicitly declares the local variable $result:
# whitespace-literal output code is omitted from the final bytecode
```
