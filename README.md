## MuON: Micro Object Notation

**MuON** is a format for data interchange — like JSON — but different in
important ways:
  * *Line-oriented* — essentially a tree of key/value pairs
  * *Schemas* are part of the format — not bolted on as an afterthought
  * Comments are allowed!

Example document:
```
# Comments begin with #
sample: Text can contain "quotes" and colons (:)
the_table:
    a: 13
    b: true
    poem: Once upon a midnight dreary, … 🌑+🌁?
    pi: 3.141592653589793
```

### Structure

**MuON** documents must be UTF-8 encoded with no 
[byte-order mark](https://unicode.org/glossary/#byte_order_mark).  Since UTF-8
does not allow
[surrogate code points](https://unicode.org/glossary/#surrogate_code_point),
all characters are
[Unicode scalars](https://unicode.org/glossary/#unicode_scalar_value).

A document is a sequence of [lines](#line), which must be: [blank](#blank),
[comments](#comment), or [definitions](#definition).

Collectively, the definitions compose a *tree* of [tables](#table), with a
*root* table which represents the entire document.

A [schema](#schema) is required for [type](#type) information, otherwise all
[values](#value) are parsed as [text](#text).

<br/><a name="line"></a>
A **line** is a sequence of characters ending in a line feed (U+000A).  Every
line feed in a document terminates a single line.

<br/><a name="space"></a>
**Space** is the space character (U+0020) only.  Other whitespace characters
are not considered *space*: not tab, line feed, carriage return, vertical tab,
form feed, etc.

<br/><a name="blank"></a>
A **blank** line contains either nothing or a single carriage return (U+000D).
Carriage returns are *not* recommended.

<br/><a name="comment"></a>
A **comment** is a [line](#line) beginning with a number sign `#`.

`# Example comment`

Any number of [spaces](#space) may appear before a comment, but nothing else.

<br/><a name="definition"></a>
A **definition** is a [line](#line) of the form:
  1. zero or more [indents](#indent)
  2. [key](#key)
  3. one or two colons `:` or `::`
  4. one [space](#space)
  5. [value](#value)

```
key: value
table_a:
    key2: value2
```

[Table](#table) mappings are created with definitions.  With no
[indents](#indent), the mapping is for the *root* table.  With indents, the
mapping is for the previous table with one *less* indent level.

Usually a single colon should be used — double colons are only needed for
certain [text](#text) [lists](#list).

If the value contains no characters, the space after the colon is not required.

<br/><a name="append"></a>
To **append** a [value](#value) to the previous [definition](#definition), the
[key](#key) must be replaced with a sequence of [spaces](#space) of the same
length (including any quotes).  When the value is parsed as [text](#text), a
line feed is inserted before it.

```
key: value
   : appended
```

To append a value to an earlier (before the previous) definition, the key must
be included.

```
table_list:
    x: false
table_list:
    x: true
```

<br/><a name="key"></a>
A **key** is a sequence of one or more characters.  Sometimes it is desirable
to use keys directly within programming languages as 
[identifiers](https://en.wikipedia.org/wiki/Identifier#In_computer_languages).
In that case, keys should only contain ASCII alphanumeric and underscore `_`
characters.

A key must be `"`surrounded by quotes`"` if it:
  * begins with a [space](#space), quote mark `"` or number sign `#`
  * contains a colon `:`

A key should be quoted if it:
  * begins with any whitespace character
  * contains any control characters
  * contains any homoglyphs of colon

When surrounded by quotes, all `"` within the key must be escaped by doubling.

`"quoted" key` ⇒ `"""quoted"" key"`

<br/><a name="value"></a>
A **value** is a sequence of zero or more characters.  Values have no
[type](#type).

<br/><a name="indent"></a>
**Indents** are used for nesting values within [tables](#table).  Each indent
in a document must contain the same number of [spaces](#space), typically 2-4.
For multiple nesting levels, multiple indents are used.

```
mesa:
   # 3 space indent; ok
   comida: taco
   bandeja:
      # Two indents: 6 spaces
      nota: Lo dejo
```

If there are many deeply nested tables, it is recommended to include
[comments](#comment) every 24 lines to help provide context.  These comments
should be of the form:

`# table_1: table_2: … table_n:`
<br/><br/>

### <a name="schema"></a>Schemas

A schema is a document template with all [values](#value) containing
[types](#type).  It can either be separate or prepended to a document.  In any
case, it must begin and end with a [line](#line) containing three colons.

```
:::
# Sample schema
sample: text
the_table: table
    a: int
    b: bool
    poem: text
    d: float
:::
```

When parsing a document, any [definition](#definition) which does not have a
matching line in the schema must trigger an error.

For each [table](#table) in a schema, a single [definition](#definition) can
include `default` (after the [type](#type)).  This allows the table's
[value](#value) to be replace that definition (as a shortcut).

```
:::
table_x: table
    a: int default
    b: text
:::
table_x: 15
    b: a is equal to 15
```

<br/><a name="type"></a>
A **type** in a [schema](#schema) must be one of the following:
  * [`text`](#text), `text?`, `[text]`
  * [`bool`](#bool), `bool?`, `[bool]`
  * [`int`](#int), `int?`, `[int]`
  * [`float`](#float), `float?`, `[float]`
  * [`table`](#table), `table?`, `[table]`

Types ending in a question mark `?` are <a name="optional"></a>**optional** —
their [definition](#definition) may or may not be present.

Types `[`surrounded by square brackets`]` are [lists](#list) — they may
contain zero or more [values](#value).

<br/> <a name="text"></a>
**Text** is a sequence of zero or more characters.  In the absence of a schema,
all [types](#type) are text.

<br/><a name="bool"></a>
A **bool** [value](#value) represents a *boolean*: either `true` or `false`.

<br/><a name="int"></a>
An **int** is a whole number (integer) of one of these forms:

  * *Decimal*: sequence of digits `0`-`9` (not starting with `0`).  May have a
    sign prefix `+` or `-`
  * *Binary*: `0b` followed by sequence of digits `0` or `1`
  * *Octal*: `0o` followed by sequence of digits `0`-`7`
  * *Hexadecimal*: `0x` followed by sequence of characters `0`-`9`, `A`-`F` or
    `a`-`f`

Underscores `_` are allowed to help readability, but each underscore must be
surrounded by other digits.

`4 0b1000 0o17 0x10 23 0b10_1010`

<br/><a name="float"></a>
A **float** is an IEEE 754 floating point number, made up of three parts:
  1. Whole number part (same as decimal [int](#int))
  2. Fractional part (decimal point `.` followed by sequence of digits `0`-`9`)
  3. Exponent part (`e` followed by decimal [int](#int))

At least one of the whole or fractional parts must be present, but the exponent
part is not required.

The rules for underscores are the same as with [ints](#int).

*Infinity* and *not-a-number* are allowed, with `inf` and `nan` respectively.
Either can be prefixed with a `+` or `-` sign.

`-1.5 .0195 1e-10 13.835e12 -nan`

<br/><a name="table"></a>
A **table** contains mappings from [keys](#key) to [values](#value).  Each
subsequent [definition](#definition) with one additional [indent](#indent)
creates a mapping for the table.

<br/><a name="list"></a>
A **list** is a sequence of zero of more [values](#value) of the specified
[type](#type).  The values are delimited by [spaces](#space).

The [definition](#definition) should be omitted if the list is empty.

For [text](#text) containing spaces, a double colon `::`
[definition](#definition) can be used to treat the entire value as a single
item in the list.

A definition [appended](#append) to a list will append all items in the
[value](#value) to the end of the list.

```
:::
table_list: [table]
    a: int
    b: [text]
:::
table_list:
    a: 5
    b:: first item
      : second third fourth fifth
     :: sixth item
table_list:
    a: 10
    b: first second third fourth fifth
    :: sixth item
```
