# Parser3 Data Classes

## string

In expressions: `def value` means "not empty?"; logical/numerical value = attempt to convert to double (empty string → 0).

```parser3
^if(def $form:name)      # not empty?
^if($user.isAlive)       # true? [auto-convert to number, not zero?]

^string:sql{query}[[$.limit(1) $.offset(4) $.default{n/a} $.bind[see table::sql]]]
# query result must be one column / one row

^string.int[]  ^string.int(default)       # integer value; on failure returns default
^string.double[]  ^string.double(default)
^string.bool[]  ^string.bool(default)

^string.format[format]   # %d  %.2f  %02d ...

^string.match[string-pattern|regex-pattern][[search options]]
# sets: $prematch $match $postmatch $1 $2 ...
# search options:
#   i  CASELESS
#   x  whitespace in regex ignored
#   s  singleline: $ matches end of entire text
#   m  multiline: $ matches end of line [\n]
#   g  find all occurrences
#   '  create columns prematch, match, postmatch
#   n  return number of matches instead of a table
#   U  invert meaning of '?' modifier

^string.match[string-pattern|regex-pattern][search options]{replacement}
# additional option: g — replace all occurrences

^string.split[delimiter|regex][[lrhva]][[column name for vertical splitting]]
# l  left to right [default]
# r  right to left
# h  nameless table with keys 0, 1, 2, ...
# v  table of one column 'piece' or as provided [default]
# a  array

^string.{l|r}split[delimiter]   # table from $piece column (compatibility)

^string.upper[]  ^string.lower[]
^string.length[]
^string.mid(P[;N])              # without N: "until end of string"
^string.left(N)                 # -1 returns entire string
^string.right(N)
^string.pos[substring]
^string.pos[substring](position from which to search)  # <0 = not found

^string.replace[$table_of_substitutions_string_to_string]
^string.replace[$what;$to]

^string.save[[append;]path]
^string.save[path[;$.charset[encoding] $.append(true)]]

^string.trim[start|both|end|left|right[;chars]]   # default chars = whitespace
^string.trim[chars]                               # removes from both ends

^string.base64[ $.pad(bool) $.wrap(bool) $.url-safe(bool) ]   # encode
^string:base64[encoded[; $.pad(bool) $.strict(bool) $.url-safe(bool) ]]  # decode

^string.idna[]           # IDNA encoding (supports Cyrillic domains)
^string:idna[encoded]    # IDNA decoding

^string.js-escape[]               # encode for JS (%uXXXX)
^string:js-unescape[escaped]      # decode from JS
^string:unescape[js|uri;escaped; $.charset[] ]   # decode from JS or URI

^string.contains[key]   # compatibility alias for hashtable
```

## table

In expressions: logical value = "not empty?"; numerical value = `count[]`.

```parser3
$table.field                  # get field value of current row
$table.field[new value]       # set field value of current row
$table.fields                 # current row as a Hash (named table)

^table::create[[nameless]]{data}[[$.separator[^#09] $.encloser[]]]
^table::create[table][[$.limit(1) $.offset(5) $.offset[cur] $.reverse(1)]]
# clones the table; reverse — in reverse order

^table::load[[nameless;]path[;options]]
# if not nameless, column names from first line
# empty lines and lines with '#' in first column are ignored
# $.separator[^#09]
# $.encloser["] by default none

^table::sql{query}[[$.limit(2) $.offset(4) $.bind[hash]]]
# bind: hash associating ":name" query variables with values (currently Oracle only)

^table.save[[nameless|append;]path[;options]]

^table.menu{body}[[delimiter]]            # iterate each row
^table.foreach[position;value]{body}[[delimiter]]

^table.line[]                             # current row number, starting from 1
^table.offset[]                           # current row offset from start, starting from 0
^table.offset[[whence]](5)               # shift; whence=cur|set, default=cur

^table.count[]  ^table.count[rows]        # number of rows
^table.count[columns]                     # number of columns
^table.count[cells]                       # cells in current row

^table.sort{{string-key-maker}|(numeric-key-maker)}[{desc|asc}]   # default=asc
# the OUTER bracket you call .sort with IS the mode switch — {} sorts as string, () sorts as number:
^table.sort{$table.name}          # alphabetic — "10" sorts before "2"
^table.sort($table.age)           # numeric   — 2 sorts before 10
^table.sort($table.age)[desc]

^table.append{data}
^table.append[ $.column_name[column_value] ]
^table.insert{data}                       # add record at current position
^table.insert[ $.column_name[column_value] ]
^table.delete[]                           # delete record at current position

^table.join[table][$.limit(1) $.offset(5) $.offset[cur]]   # append rows; same structure required
^table.flip[]                             # returns transposed table

^table.locate[field;value][[$.limit(1) $.offset(5) $.offset[cur] $.reverse(1)]]
^table.locate(logical expression)[[$.limit(1) $.offset(5) $.offset[cur] $.reverse(1)]]
# moves current row if found; returns bool

^table.hash{[field]|{code}|(expression)}[[value field(s)|table of value fields]{value code}][[$.distinct(1) $.distinct[tables] $.type[hash]]]
# default: $hash.key value is a hash where value fields are keys
# value fields optional — defaults to all columns including key
# distinct(1): no error on duplicate keys
# distinct[tables]: hash of tables containing rows with that key
# $.type[string|table]: element value as string (one column) or table

^table.columns[[column name]]             # table of one 'column' column (or as provided)
^table.cells[]  ^table.cells(limit)       # array of cells in current row
^table.array[]                            # array of hashes (one per row)
^table.array[column]                      # array of values from specified column
^table.array{code}                        # array of results from code per row

^table.rename[column name;new column name]
^table.rename[ $.column_name[new column name] ...]

$selected[^table.select(expression)]      # rows where condition matched
$adults[^man.select($man.age>=18)]

^table.color[color1;color2]               # alternate colors per row
```

## hash

In expressions: logical value = "not empty?" (hash with `_default` is already not empty); numerical value = `count[]`.

```parser3
$hash.key                    # access key; _default returned for missing keys if defined
$hash.fields                 # returns $hash (makes hash similar to table class)

^hash::create[[|copy_from_hash|copy_from_hashfile]]

^hash.add[term]              # overwrites entries with same name
^hash.sub[subtracted]
^hash.union[b]               # union; same-named keys remain as-is
^hash.intersection[b][[$.order[self|arg]]]   # new hash; order = element order source
^hash.intersects[b]          # returns bool

^hash::sql{query}[[$.distinct(1) $.limit(2) $.offset(4) $.type[hash|string|table]]]
# result: hash(key=first column) of hash(key=other column names)
# string: two columns only, value as string
# table: value as table

^hash.keys[[name of key column]]          # table of one 'key' column (or as provided)
^hash.count[]
^hash.foreach[key;value]{body}[[delimiter]|{delimiter before each non-empty non-first body}]
^hash.delete[key]
^hash.contains[key]                       # bool

^hash.at[first|last][[key|value|hash]]
^hash.at([-]N)[[key|value|hash]]          # access by ordinal number
^hash.set[first|last;value]
^hash.set([-+]N)[value]                   # set by ordinal number

^hash.array[[keys|values]]               # array of hash, or array of keys/values
^hash.rename[old_key;new_key]
^hash.rename[ $.old_key[new_key] ...]
^hash.sort[key;value]{{string-key-maker}|(numeric-key-maker)}[[desc|asc]]   # default=asc
# same {} vs () mode switch as ^table.sort — see above

$reversed_hash[^hash.reverse[]]
$selected[^hash.select[key;value](expression)[ $.limit(N) $.reverse(bool) $.default(bool) ]]
# hash of keys/values for which condition is true
```

## hashfile

Persistent key-value store on disk with optional expiry.

```parser3
^hashfile::open[filename]

^hashfile.clear[]                         # forget all entries
$hashfile.key[value]                      # put value
$hashfile.key[$.value[value] $.expires[VALUE]]
# expires: a date, or number of days (0 days = forever)
$hashfile.key                             # retrieve value
^hashfile.delete[key]                     # delete key
^hashfile.delete[]                        # delete files containing data

^hashfile.hash[]                          # convert to regular hash; removes expired pairs
^hashfile.foreach[key;value]{body}[[delimiter]|{delimiter before each non-empty non-first body}]

^hashfile.release[]
# write data and release locks; next access reopens automatically

^hashfile.cleanup[]
# iterate all elements and delete expired ones
```

Example:
```parser3
$sessions[^hashfile::open[/db/sessions]]
$sid[^math:uuid[]]
$sessions.$sid[$.value[$uid] $.expires(1)]
$uid[$sessions.$sid]
```

## array

Available since Parser 3.5.0. Do not use array syntax in older projects.
Official docs: https://www.parser.ru/docs/lang/arraycreate.htm

In expressions: logical value = "not empty?"; numerical value = `count[]`.

```parser3
$array.index                              # value at index
$array.(expression)                       # value at computed index
$array.index[value]                       # assign by index
$array.(expression)[value]
$array[value;value;...]                   # create array with values, Parser 3.5.0+

^array::create[]
^array::create[value;value;...]           # Parser 3.5.0+
^array::copy[array or hash with numeric keys]

^array.add[array or hash with numeric keys]     # overwrite values for matching indexes
^array.join[array or any hash]                  # append to end of array

^array.append[value;value;...]            # append to end
^array.insert(index)[value;value;...]     # insert at position
^array.left(n)                            # new array of first n elements
^array.right(n)                           # new array of last n elements
^array.mid(m;n)                           # n elements starting from position m

^array.delete(index)                      # delete element, leaving empty spot
^array.remove(index)                      # delete and shift subsequent elements

^array.push[value]                        # add to end
^array.pop[]                              # return and remove last element
^array.contains(index)                    # bool

^array::sql{query}[ $.sparse(false|true) $.distinct(false|true) $.limit(n) $.offset(n) $.type[hash|string|table] ]
# sparse(false): normal array, row values added sequentially
# sparse(true): sparse array, first column = index, like ^hash::sql{}
# type: array of hash (default) | string (two columns) | table

^array.keys[[column name]]                # table of 'key' column with initialized indexes
^array.count[]                            # number of initialized elements
^array.count[all]                         # total elements including uninitialized

^array.foreach[index;value]{code}[[delimiter]]   # iterate initialized elements
^array.for[index;value]{code}[[delimiter]]       # iterate all elements

^array.at[first|last][[key|value|hash]]
^array.at([-]number)[[key|value|hash]]    # access by ordinal number
^array.set[first|last][value]
^array.set([-]number)[value]

^array.compact[]                          # remove uninitialized elements
^array.compact[undef]                     # remove uninitialized and empty elements

^array.sort[key;value]{{string-key-maker}|(numeric-key-maker)}[[desc|asc]]   # default=asc
# same {} vs () mode switch as ^table.sort — see above

$reversed_array[^array.reverse[]]
$selected[^array.select[key;value](expression)[ $.limit(N) $.reverse(bool) ]]
```
