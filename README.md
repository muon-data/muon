## MuON: Micro Object Notation

**MuON** is a format for data interchange — like JSON — but different in
important ways:
  * *Line-oriented* — essentially a tree of key/value pairs
  * *Schemas* are part of the format — not bolted on as an afterthought
  * Comments are allowed!

```
# Example MuON document
sample: Text can contain "quotes" and colons (:)
the_table:
    a: 13
    b: true
    poem: Once upon a midnight dreary, … 🌑 + 🌁?
    pi: 3.141592653589793
```

### Specification

**MuON** documents must be UTF-8 encoded with no
[byte-order mark](https://unicode.org/glossary/#byte_order_mark).  Since UTF-8
does not allow
[surrogate code points](https://unicode.org/glossary/#surrogate_code_point),
all characters are
[Unicode scalars](https://unicode.org/glossary/#unicode_scalar_value).

Every line feed (U+000A) marks the end of a <a name="line">**line**</a>.
There are threes types: **blank**, <a name="comment">**comment**</a>, and
**definition**.  A blank line simply contains no characters.  Comments begin
with any number of space characters followed by a number sign:

`# Example comment`

A MuON document represents a *tree* of [tables](#table).
A <a name="definition"></a>**definition** creates a table mapping between a
[key](#key) and [value](#value), like so:
```
key: value
```

With no <a name="indent">**indents**</a>, mappings are created for the *root*
of the tree.  If a mapping defines a new table, subsequent definitions create
mappings for that table by using an additional indent.

```
key in root: value in root
a:
    key in a: value in a
```

Every indent in a document must contain the same number of spaces, typically
2 to 4.  Only space characters (U+0020) are allowed, not tabs or other
whitespace.  For nested tables, multiple indents are used.

```
mesa:
   # 3 space indent; ok
   comida: taco
   bandeja:
      # Two indents: 6 spaces
      nota: Lo dejo
```

A <a name="key"></a>**key** is a sequence of one or more characters.  It must
be `"`surrounded by quotes`"` if it:
  * begins with a space character, quote mark or number sign
  * contains a colon

When `"`quoted`"`, all quote marks must be escaped by doubling:

`"quoted" key` ⇒ `"""quoted"" key"`

Also, a key *should* be quoted if it:
  * begins with any whitespace character
  * contains any control characters
  * contains any homoglyphs of colon

It is common to use keys directly within programming languages as
[identifiers](https://en.wikipedia.org/wiki/Identifier#In_computer_languages).
In this case, they should only contain ASCII alphanumeric and underscore
characters.

A <a name="value"></a>**value** is a sequence of zero or more characters.  If
it contains none, the space after the colon in the definition is not required.

### <a name="schema"></a>Schema

A schema is a document template with all [values](#value) containing
[types](#type).  It can either be separate or prepended to a document.  In
either case, it must begin and end with a line containing three colons.

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

If no schema is available, all values are treated as [text](#text).

For each [table](#table) in a schema, a single [definition](#definition) can
include `default` (after the [type](#type)).  This allows the table's
[value](#value) to replace that definition (as a shortcut).

```
:::
table_x: table
    a: int default
    b: text
:::
table_x: 15
    b: a is equal to 15
```

<a name="type"></a>
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

<a name="text"></a>
**Text** is a sequence of zero or more characters.  Because values cannot
contain line feeds, the only way to define text containing them is by
**appending**.  To do this, create a definition with the key replaced by a
sequence of space characters of the same length as the previous key (including
any quotes).  A line feed will be inserted before the appended text.

```
text: value
    : appended
```

<a name="bool"></a>
A **bool** is a *boolean*: either `true` or `false`.

<a name="int"></a>
An **int** is a whole number (integer) of one of these forms:

  * *Decimal*: sequence of digits `0`-`9` (not starting with `0`).  May have a
    sign prefix `+` or `-`
  * *Binary*: `0b` followed by sequence of digits `0` or `1`
  * *Octal*: `0o` followed by sequence of digits `0`-`7`
  * *Hexadecimal*: `0x` followed by sequence of characters `0`-`9`, `A`-`F` or
    `a`-`f`

Underscores are allowed to improve readability, but each underscore must be
surrounded by other digits.

`4 0b1000 0o17 0x10 23 0b10_1010`

<a name="float"></a>
A **float** is an IEEE 754 floating point number, made up of three parts:
  1. Whole number part (same as decimal [int](#int))
  2. Fractional part (decimal point `.` followed by sequence of digits `0`-`9`)
  3. Exponent part (`e` followed by decimal [int](#int))

At least one of the whole or fractional parts must be present, but the exponent
part is not required.  The rules for underscores are the same as with
[ints](#int).

The values `inf` and `NaN` stand for *infinity* and *not a number*,
respectively.  Either can be prefixed with a `+` or `-` sign.

`-1.5 .0195 1e-10 13.835e12 +inf`

<a name="table"></a>
A **table** contains mappings from [keys](#key) to [values](#value).  Each
subsequent [definition](#definition) with one additional [indent](#indent)
creates a mapping for the table.

<a name="list"></a>
A **list** is a sequence of zero of more [values](#value) of the specified
[type](#type).  The values are delimited by single space characters, but the
[definition](#definition) should be omitted if the list is empty.

```
:::
flags: [bool]
checks: [bool]
:::
flags: true false true true
# checks list is empty
```

Like [text](#text), lists can be **appended**; all items are added to the end
of the list.

```
numbers: 2 4 6 8
       : 10 12 14 16
# same as numbers: 2 4 6 8 10 12 14 16
```

For lists of text containing spaces or line feeds, a definition using a double
colon `::` can be used to treat the values as a single item in the list.

```
:::
text_list: [text]
:::
text_list: first second
         :: third item
         : fourth fifth
         :: sixth
         :: item
         : seventh
```

To append to a table, the key must be included as normal.

```
:::
table_list: [table]
    a: int
:::
table_list:
    a: 5
table_list:
    a: 10
```
