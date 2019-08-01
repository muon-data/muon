## MuON v0.2.0alpha

*Micro Object Notation*

**MuON** is a format for configuration files and data interchange — as
expressive as other formats, but [much simpler](WHY.md).

```
# Example MuON
sample: Text can contain "quotes" and colons (:)
record:
  apples: 13
  owned: true
  poem: Once upon a midnight dreary, … 🌑 + 🌁?
  constant: 3.141592653589793
```

### Specification

**MuON** files are [UTF-8 encoded](https://en.wikipedia.org/wiki/UTF-8) with no
[byte-order mark](https://unicode.org/glossary/#byte_order_mark).  Since UTF-8
does not allow
[surrogate code points](https://unicode.org/glossary/#surrogate_code_point),
all characters are
[Unicode scalars](https://unicode.org/glossary/#unicode_scalar_value).

Each file is a *tree* of **dictionaries** (records), with an implied **root**.

Every line feed (U+000A) marks the end of a **line**.  There are three line
types: *blank*, *comment* and *definition*.  **Blank** lines contain no
characters.  **Comments** begin with zero or more spaces followed by a number
sign:

`# Example comment`

A **definition** creates a mapping between a **key** and **value**, like so:
```
key: value
```

With no **indents**, mappings are created for the root dictionary.  When a
mapping defines a new dictionary, subsequent definitions with an additional
indent are for that dictionary.

```
key_in_root: value in root
branch:
  key_in_branch: value in branch
```

Every indent in a file must contain the same number of spaces, typically 2 to 4.
Only spaces (U+0020) are allowed, not tabs or other whitespace.  For nested
dictionaries, multiple indents are used.

```
mesa:
   # One indent; 3 spaces
   comida: taco
   bandeja:
      # Two indents; 6 spaces
      nota: Lo dejo
```

A **key** is a sequence of one or more characters.  It must be `"`**quoted**`"`
if it contains a colon or begins with a space, quote mark or number sign.  When
*quoted*, all contained quote marks must be escaped by doubling:

```
# "skeleton" key must be quoted
"""skeleton"" key": value
```

Also, a key *should* be quoted if it contains any colon
[homoglyphs](https://en.wikipedia.org/wiki/Homoglyph#Unicode_homoglyphs),
[control characters](https://en.wikipedia.org/wiki/Unicode_control_characters)
or begins with [whitespace](https://en.wikipedia.org/wiki/Whitespace_character).

It is common to use keys directly within programming languages as
[identifiers](https://en.wikipedia.org/wiki/Identifier#In_computer_languages).
In this case, they should only contain ASCII alphanumeric and underscore
characters.

A **value** is a sequence of characters.  If empty, the space after the colon
in the definition is not required.

### Schema

A **schema** is a template with all values containing *types*.  It can either be
separate or prepended to a MuON file.  In either case, it begins and ends with a
line of three colons.

```
:::
# Example schema
sample: text
record: dict
  apples: int
  owned: bool
  poem: text
  constant: float
:::
```

A **type** is one of the following:

Basic   | Optional |   List
------- | -------- | ---------
`text`  | `text?`  | `[text]`
`bool`  | `bool?`  | `[bool]`
`int`   | `int?`   | `[int]`
`float` | `float?` | `[float]`
`dict`  | `dict?`  | `[dict]`

**Optional** types (with a question mark) are not required — their definition
may not be present.

**Text** is a sequence of characters.  Because values cannot contain line
feeds, if one is needed a **text append separator** can be used on the next
line to *insert* one.  This is `:>` instead of the usual `: ` between the key
and value.  A line feed will be inserted between the text values.

When appending, a **blank** key can be used instead of repeating the key.
This is a sequence of spaces with the same length as the key.

```
lyric: Out in the garden
     :>There's half of a heaven
```

**Bool** is a *boolean*: either `true` or `false`.

**Int** is an *integer* (whole number) in one of three forms:

  * *Decimal*: sequence of digits `0`-`9` (not starting with `0`).  May have a
    sign prefix `+` or `-`
  * *Binary*: `0b` followed by sequence of digits `0` or `1`
  * *Hexadecimal*: `0x` followed by sequence of digits `0`-`9`, `A`-`F` or
    `a`-`f`

An underscore may be inserted between digits to improve readability.

```
a_decimal: 4
b_binary: 0b1000
c_hexadecimal: 0x0F
d_decimal: +16
e_binary: 0b01_0111
f_hexadecimal: 0x2a
```

A **float** is a
[floating point](https://en.wikipedia.org/wiki/IEEE_754) number, made up of
these parts:
  1. Whole number part (same as decimal int)
  2. Fractional part (decimal point followed by sequence of digits `0`-`9`)
  3. Exponent part (`e` followed by decimal int)

One or both of the whole or fractional parts must be present, but the exponent
part is not required.  As with ints, underscores may be included.

The values `inf` and `NaN` stand for *infinity* and *not a number*,
respectively.  Either can be prefixed with a `+` or `-` sign.

```
float0: -1.5
float1: .6931471805599453
float2: 100
float3: 6.626_070_15e-34
float4: 6.022_140_76e23
float5: +inf
```

A **dict** is a *record* containing indented mappings from keys to values.

```
:::
things: dict
  alpha: int
  beta: text
:::
things:
  alpha: 15
  beta: What have you
```

Since dictionaries do not have *values* themselves, they can be used as a short
cut for the first contained mapping.  The only restriction is that mapping must
not have a dictionary type.

```
things: 15
  beta: alpha is equal to 15
```

A **list** is a type `[`enclosed in square brackets`]`.  Values are sequences
of the contained type, separated by spaces.  Like optional types, the
definition should be omitted if the list is empty.

```
:::
flags: [bool]
checks: [bool]
:::
flags: true false true true
# checks list is empty
```

Like text, lists can be **appended**.  All items are added to the end of the
list.

```
numbers: 0 1 1 2 3
       : 5 8 13 21 34
# same as numbers: 0 1 1 2 3 5 8 13 21 34
```

To append to a **list of dictionaries**, since the lines are not consecutive,
the key must not be blank.
```
:::
person: [dict]
  name: text
  birthday: date
:::
person: George Washington
  birthday: 1732-02-22
person: Abraham Lincoln
  birthday: 1809-02-12
```

For a **list of text**, items are separated by spaces, just like other lists.
If spaces are needed, a **text value separator** `:=` can be used to treat the
entire value as a single text item.  The text append separator `:>` will also
behave the same way.

```
:::
to_do: [text]
:::
to_do: first second
     :=third item
     : fourth fifth sixth
     :>item
     : seventh
```
