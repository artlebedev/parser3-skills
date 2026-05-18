# Parser3 Utility Classes

## date

Date type can be used in expressions as number of days since epoch (1 January 1970 UTC), fractional.
String value is in local time; numeric value is in UTC.
Range: `0000-00-00 00:00:00` to `9999-12-31 23:59:59`. Default timezone = OS-defined.

```parser3
^date::now[]
^date::now(days offset)           # now + offset

^date::today[]                    # date at 00:00:00 of current day
^date::today(integer days offset) # date at 00:00:00 of current day + offset

^date::create(days since epoch)
^date::create(year;month[;day[;hour[;minute[;second[;TZ]]]]])
^date::create[date in format %Y-%m-%d %H:%M:%S]
# format1: %Y[-%m[-%d[ %H[:%M[:%S]]]]]
# format2: %H:%M[:%S]
^date::create[date in ISO 8601: %Y-%m-%dT%H:%M[:%S]TZ]
# TZ: Z (UTC) or +-hour[:minute] (offset from UTC)

^date::unix-timestamp()
^date.unix-timestamp[]

# Readable/writable fields:
$date.year  $date.month  $date.day  $date.hour  $date.minute  $date.second

# Read-only fields:
$date.weekday  $date.yearday  $date.daylightsaving  $date.TZ  $date.weekyear
# TZ="" means local zone

^date.double[]  ^date.int[]       # days since epoch, fractional or truncated

^date.roll[year|month|day](+-offset)    # shift the date
^date.roll[TZ;New zone]                 # interpret date in new timezone (affects .hour etc.)
^date:roll[TZ;New zone]                 # set default timezone for all dates

^date.sql-string[[datetime|date|time]]
# datetime (default): %Y-%m-%d %H:%M:%S
# date:               %Y-%m-%d
# time:               %H:%M:%S
^date:sql-string[[datetime|date|time]]  # sql-string for now

^date:calendar[rus|eng](year;month)
# unnamed table with columns: 0..6, week, year
^date:calendar[rus|eng](year;month;day)
# named table with columns: year, month, day, weekday

^date:last-day(year;month)        # last day of the month
^date.last-day[]                  # last day of $date's month

^date.gmt-string[]                # Fri, 23 Mar 2001 09:32:23 GMT
^date:gmt-string[]                # gmt-string for now

^date.iso-string[]                # 2001-03-23T12:32:23+03
^date:iso-string[]                # iso-string for now
```

## json

```parser3
^json:parse[-json-string-[;
    $.depth(19)                    # maximum nesting depth, default = 19
    $.double(false)                # disable built-in float parsing (appear as strings)
    $.int(false)                   # disable built-in integer parsing (appear as strings)
    $.distinct[first|last|all]     # duplicate key handling:
                                   # first: keep first, last: keep last
                                   # all: keep all with numeric suffixes (key_2, ...)
                                   # default: duplicate keys cause exception
    $.object[method-junction]      # user method[key;object] called for all parsed objects
    $.array[method-junction]       # user method called for arrays
    $.taint[taint language]        # sets transformation language for all result strings
]]
# parses JSON string into a hash

^json:string[system or user object[;
    $.skip-unknown(false)          # output 'null' for unknown types instead of exception
    $.indent(true)                 # format with indentation
    $.date[sql-string|gmt-string|iso-string|unix-timestamp]   # default = sql-string
    $.table[object|array|compact]  # default = object
    # object:  [{"c1":"v11","c2":"v12",...}, ...]
    # array:   [["c1","c2",...] || null, ["v11","v12",...], ...]
    # compact: ["v11" || ["v11","v12",...], ...]
    $.file[text|base64|stat]       # file content in output (default: not included)
    $.xdoc[hash]                   # parameters for ^xdoc.string[]
    $.type[method-junction]        # user method(key, object, options) for specific types
    $._default[method]             # user method for all user-class objects
    $._default[method name]        # method name on the object for serialization
    $.void[null|string]            # undefined value as null (default) or empty string
]]
# serializes object to JSON string
```

## math

```parser3
$math:PI

^math:round  ^math:floor  ^math:ceiling
^math:trunc  ^math:frac
^math:abs    ^math:sign
^math:exp    ^math:log    ^math:log10
^math:sin    ^math:asin   ^math:cos  ^math:acos  ^math:tan  ^math:atan  ^math:atan2
^math:degrees  ^math:radians
^math:pow    ^math:sqrt
^math:random(range_width)

^math:convert[number|file](base-from;base-to)[[ $.format[string|file] ]]
^math:convert[number|file][alphabet](base-to)[[ $.format[string|file] ]]
^math:convert[number|file](base-from)[alphabet][[ $.format[string|file] ]]
# convert between numeral systems
# base: 2-16 (equivalent to 0123456789ABCDEF) or 256 (all ASCII chars); or custom alphabet

^math:eq(a;b[;max ULP])
# true if difference between doubles ≤ max ULPs (default 3)

^math:uuid[ $.lower(bool) $.solid(bool) ]
# 22C0983C-E26E-4169-BD07-77ECE9405BA5
# win32: cryptapi; unix: /dev/urandom → /dev/random → rand

^math:uuid7[ $.lower(bool) $.solid(bool) ]
# 0193CBF0-7898-7000-A391-AC513CC15658 (RFC 9562 UUID v7)

^math:uid64[ $.lower(bool) ]
# BA39BAB6340BE370

^math:md5[string]
# digest: 16 bytes as string, hex lowercase contiguous

^math:crypt[password;salt]
# $apr1$ prefix: built-in MD5 (generates random salt if body empty)
# $1$: OS crypt MD5 if supported; other salts: see OS crypt docs

^math:crc32[string]

^math:sha1[string]

^math:digest[[md5|sha1|sha256|sha512];string or file][[ $.format[hex|base64|file] $.hmac[key string|key file] ]]
# combines multiple hashing algorithms; $.hmac[key] for HMAC
```

## regex

In expressions: logical value is always true; numerical value = number of bytes of compiled pattern.

```parser3
^regex::create[pattern-string|regex][[search options]]

^pattern.size[]           # bytes of compiled pattern (large value → consult pcre docs)
^pattern.study_size[]     # size of study-structure; 0 = pattern cannot be "studied"

$pattern.pattern          # text of the pattern
$pattern.options          # original options string
```

## console

```parser3
$console:timeout          # read/write timeout
$console:line             # read/write string (one line)
```

## env

```parser3
$env:variable             # any environment variable
$env:fields               # hash with all environment variables
$env:PARSER_VERSION       # parser version
```

## image

```parser3
$image[^image::measure[DATA[; $.exif(bool) $.xmp(bool) $.xmp-charset[] $.video(bool) ]]]
# measures gif, jpg, tiff, bmp, webp, mp4/mov; extension check is case-insensitive

$image.exif               # hash with EXIF data (after jpeg measure with $.exif(true))
# $image.exif.DateTime etc; numbers as int/double, dates as date, enums as hash 0..count-1
# full list: https://exiftool.org/TagNames/EXIF.html

$image.src  $image.width  $image.height
$image.line-width          # number = line width
$image.line-style          # string = line style pattern, e.g. '*** * '

^image.html[[hash]]        # outputs <img ...>

^image::load[background.gif]    # only gif
^image::create(width;height[;background color])   # default background = white

^image.line(x0;y0;x1;y1;0xffFFff)
^image.fill(x;y;0xffFFff)
^image.rectangle(x0;y0;x1;y1;0xffFFff)
^image.bar(x0;y0;x1;y1;0xffFFff)
^image.replace(hex-color1;hex-color2)[table x:y polygon_vertices]
^image.polyline(color)[table x:y points]
^image.polygon(color)[table x:y polygon_vertices]
^image.polybar(color)[table x;y polygon_vertices]

^image.font[set_of_letters;font_file.gif][(space_width[;char_width])]
# char height = image height / number of letters in set
# char_width: 0 = gif width; omit = proportional
^image.font[set_of_letters;font_file.gif;
    $.space(space_width)        # default = gif width
    $.width(char_width)         # monospaced width; default = proportional
    $.spacing(letter_spacing)   # default = 1
]

^image.text(x;y)[text]          # AS_IS
^image.length[text]             # AS_IS

^image.gif[optional filename]   # encode to file; content-type=image/gif; filename used by $response:download

^image.arc(center_x;center_y;width;height;start_degrees;end_degrees;color)
^image.sector(center_x;center_y;width;height;start_degrees;end_degrees;color)
^image.circle(center_x;center_y;r;color)

^image.copy[source](src_x;src_y;src_w;src_h;dst_x;dst_y[;dest_w[;dest_h[;tolerance]]])
# dest_w/dest_h: resize the piece (resamples when reducing)
# suitable for simple low-color graphics; NOT for photo thumbnails
# dest_h omitted: keep aspect ratio
# tolerance: square distance in RGB space (default=150); smaller = more accurate but fewer colors

^image.pixel(x;y)[(color)]      # get or set pixel color
```

## memory

```parser3
^memory:compact[]
# collect garbage, free space for new data
# WARNING: process memory is never released; useful before XSL transform

^memory:auto-compact(frequency)
# set automatic GC frequency: 0 (off) to 5 (max)
```

## reflection

```parser3
^reflection:create[class;constructor[;pa[;ra[;ms]]]]
# calls specified class constructor (up to 100 parameters)
^reflection:create[ $.class[name] $.constructor[name] $.arguments[ $.1[pa] $.2[ra] $.3[ms] ] ]

^reflection:classes[]                # hash of all classes; key=name, value=methoded or void
^reflection:class[object]            # class of the given object
^reflection:class_name[object]       # class name of the given object
^reflection:base[object]             # parent class of the given object
^reflection:base_name[object]        # parent class name

^reflection:class_by_name[class name]
^reflection:class_alias[class name;new class name]
^reflection:def[class;class name]    # checks if class exists (bool)

^reflection:methods[class]           # hash of methods; values: 'native' or 'parser'
^reflection:method[class or object;method name]   # returns junction-method

^reflection:filename[object or class or method]   # filename where defined

^reflection:fields[class or object]              # hash of static (class) or dynamic (object) fields
^reflection:fields_reference[object]             # editable hash of dynamic fields

^reflection:field[class or object;field name]    # field value; getters ignored
^reflection:copy[source;destination]             # copy fields between objects/classes

^reflection:uid[class or object]                 # object/class identifier

^reflection:method_info[class;method]
# hash with method parameters:
# $.inherited[class]  name of class where method was defined (if defined in ancestor)
# $.overridden[class]
# native classes: .min_params .max_params .call_type[dynamic|static|any]
# parser classes: key=param number (0,1,...), value=param name

^reflection:dynamical[[object or class]]
# true if method called from dynamic context; with parameter: true if dynamic object passed

^reflection:delete[class or object;variable name]

^reflection:is[element name;class name][[context]]
# analogous to 'is' operator; determines if element is code

^reflection:tainting[[language|tainted|optimized];string]
# one char per original char, showing transformation code

^reflection:stack[ $.args(false/true) $.locals(false/true) $.limit(n) $.offset(o)]
# current method call stack state

^reflection:mixin[source; $.to[target] $.name[name] $.methods(true/false) $.fields(true/false) $.overwrite(false/true)]
# copy methods and fields from one class to another

^reflection:override[method[; $.to[target] $.name[new_name]]]
# override or define a method
```

## status

```parser3
$status:sql          # cache table: url  time
$status:stylesheet   # cache table: file  time

$status:rusage       # hash with resource usage:
# utime  stime  maxrss  ixrss  idrss  isrss  tv_sec  tv_usec
$s[$status:rusage]
^s.tv_sec.format[%.0f].^s.tv_usec.format[%06.0f]

$status:memory       # hash:
# used                         (includes pages allocated but never written)
# free
# ever_allocated_since_compact (bytes allocated since last collection)
# ever_allocated_since_start   (total bytes ever allocated, never decreases)

$status:pid          # process id
$status:tid          # thread id
$status:mode         # working mode: cgi|console|mail|httpd|apache|isapi
$status:log-filename # path to parser3.log error log
```

## xdoc

XML document. String encoding defaults to `$response:charset`.

```parser3
$xdoc.search-namespaces   # hash: keys=prefixes, values=URLs

# DOM1 attributes (read-only):
$xdoc.doctype             # DocumentType
$xdoc.documentElement     # Element

# DOM1 creation methods:
^xdoc.createElement[tagName]
^xdoc.createDocumentFragment[]
^xdoc.createTextNode[data]
^xdoc.createComment[data]
^xdoc.createCDATASection[data]
^xdoc.createProcessingInstruction[target;data]
^xdoc.createAttribute[name]
^xdoc.createEntityReference[name]
^xdoc.getElementsByTagName[tagname]   # NodeList

# DOM2:
^xdoc.getElementById[elementId]       # xnode (requires DTD-defined ID attributes)

# Constructors:
^xdoc::sql{...}
^xdoc::create[[URI]]{<?xml?><string/>}   # old name: 'set'
^xdoc::create[[URI]][qualifiedName]
# URI default = disk path to requested document; trailing / required for directories
^xdoc::create[$f]   # from file object (e.g. ^file::load[binary;http://...])
^xdoc::load[file.xml[;options]]

# XSLT:
^xdoc.transform[rules.xsl|xdoc][[params hash]]   # returns dom
# template cached; cache invalidated by file date or .stamp file date change
# <xsl:output method="xml|html|text" encoding=... indent="yes|no" ...>
# parameters passed as-is (not XPath expressions)

# Output:
^xdoc.string[[output options]]
^xdoc.save[file.xml[;output options]]    # saves with XML header
^xdoc.file[[output options]]             # returns file object
# output options = xsl:output attributes (exception: cdata-section-elements ignored)
# returns media-type when used with $response:body[here]

# parser:// protocol:
# parser://method/param/to/that/method → ^MAIN:method[/param/to/that/method]
```

## xnode

DOM node object.

```parser3
# DOM1 attributes:
$node.nodeName
$node.nodeValue    # read and write
$node.nodeType     # int constant:
# ELEMENT_NODE=1, ATTRIBUTE_NODE=2, TEXT_NODE=3, CDATA_SECTION_NODE=4
# ENTITY_REFERENCE_NODE=5, ENTITY_NODE=6, PROCESSING_INSTRUCTION_NODE=7
# COMMENT_NODE=8, DOCUMENT_NODE=9, DOCUMENT_TYPE_NODE=10
# DOCUMENT_FRAGMENT_NODE=11, NOTATION_NODE=12
$vasyaNode.type==$xnode:ELEMENT_NODE

$node.parentNode
$node.childNodes       # array of nodes
$node.firstChild
$node.lastChild
$node.previousSibling
$node.nextSibling
$node.ownerDocument    # xdoc
$node.prefix
$node.namespaceURI

$element_node.attributes   # hash of xnodes
$element_node.tagName

$attribute_node.specified  # bool: true if explicitly set in XML or programmatically
$attribute_node.name
$attribute_node.value

$text_node.substringData   # also: cdata_node, comment_node
$pi_node.target            # first token after processing instruction markup
$pi_node.data              # content between target and ?>

# document_node read-only: .doctype, .implementation, .documentElement
# document_type_node read-only: .name, .entities (NamedNodeMap), .notations (NamedNodeMap)
# notation_node read-only: .publicId, .systemId

# DOM1 node methods:
^node.insertBefore[newChild;refChild]
^node.replaceChild[newChild;oldChild]
^node.removeChild[oldChild]
^node.appendChild[newChild]
^node.hasChildNodes[]
^node.cloneNode[deep]                # deep = boolean

# DOM1 element methods:
^node.getAttribute[name]
^node.setAttribute[name;value]
^node.removeAttribute[name]
^node.getAttributeNode[name]
^node.setAttributeNode[newAttr]
^node.removeAttributeNode[oldAttr]
^node.getElementsByTagName[name]
^node.normalize[]

# DOM Level 2:
^node.importNode[importedNode;deep]
^node.getElementsByTagNameNS[namespaceURI;localName]
^node.hasAttributes[]

# XPath:
^node.select[xpath/expression]         # array of nodes; empty if nothing found
^node.selectSingle[xpath/expression]   # first node or nothing
^node.selectBool[xpath/expression]     # bool or die
^node.selectNumber[xpath/expression]   # double or die
^node.selectString[xpath/expression]   # string or die
```

## DATA type

Used in file/mail operations:

```parser3
DATA ::= string | file | hash
# hash form:
[
    $.file[filename on disk]
    $.name[filename for user]
    $.mdate[date]
]
```
