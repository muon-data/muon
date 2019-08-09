## MuON v0.3.0alpha

*Micro Object Notation*

**MuON** is a format for configuration files and data interchange — as
expressive as other formats, but [much simpler](WHY.md).

```muon
# MuON example
movie: Alien
  director: Ridley Scott
  cast:=Sigourney Weaver
      :=Tom Skerritt
      :=John Hurt
  release: 1979-06-22
    region: USA
  release: 1979-09-06
    region: UK
  gross: 203_630_630
  emoji: 👽 👾
```

### Specification

**MuON** files are [UTF-8 encoded](https://en.wikipedia.org/wiki/UTF-8) with no
[byte-order mark](https://unicode.org/glossary/#byte_order_mark).  Since UTF-8
does not allow
[surrogate code points](https://unicode.org/glossary/#surrogate_code_point),
all characters are
[Unicode scalars](https://unicode.org/glossary/#unicode_scalar_value).

Every line feed (U+000A) marks the end of a **line**.  There are three line
types: *blank*, *comment* and *definition*.  **Blank** lines contain no
characters.  **Comments** begin with zero or more spaces followed by a number
sign:
```muon
# Example comment
```

A **definition** creates a mapping between a **key** and **value**, like so:
```muon
key: value
```

Each file is a tree of **branches** starting from a root record.  With no
**indents**, mappings are contained in the root.  When a mapping defines a
branch, subsequent definitions with an additional indent are contained by it.
```muon
key_in_root: value in root
branch:
  key_in_branch: value in branch
```

Every indent in a file must contain the same number of spaces, typically 2 to 4.
Only spaces (U+0020) are allowed, not tabs or other whitespace.  For nested
branches, multiple indents are used.
```muon
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
```muon
# "skeleton" key must be quoted
"""skeleton"" key": value
```

Also, a key *should* be quoted if it contains
[homoglyphs](https://en.wikipedia.org/wiki/Homoglyph#Unicode_homoglyphs)
of colon, contains
[control characters](https://en.wikipedia.org/wiki/Unicode_control_characters)
or begins with [whitespace](https://en.wikipedia.org/wiki/Whitespace_character).

It is common to use keys directly within programming languages as
[identifiers](https://en.wikipedia.org/wiki/Identifier#In_computer_languages).
In this case, they should contain only ASCII alphanumeric or underscore
characters.

A **value** is a sequence of characters.  With the exception of *line feed*, any
unicode character is allowed.  If a value is empty, the space after the colon in
the definition is not required.

### Schema

A **schema** is a template with all values containing *types*.  It can either be
separate or prepended to a MuON file.  In either case, it begins and ends with a
line of three colons.

```muon
:::
# Example MuON schema
movie: list record
  title: text
  director: text Alan Smithee
  cast: list text
  release: list record
    release_date: date
    region: text
  gross: int
  emoji: optional text
:::
```

A **type** is one of nine values: `text`, `bool`, `int`, `float`, `datetime`,
`date`, `time`, `record` or `dictionary`.  Any type may be preceded by a
**modifier**, either `optional` or `list`.  A **default** value can follow the
type — used when the definition is not present.  Default values are not allowed
for `record`, `dictionary` or `optional` types.  Modifiers and defaults are
separated from the type with a single space.

**Text** is a sequence of characters.
```muon
:::
greeting: text Hello!
farewell: text Goodbye!
:::
# greeting is Hello!
farewell: Be seeing you.
```

Because values cannot contain line feeds, text definitions must be **appended**
to represent them.  This is done with a **text append** separator, which is `:>`
instead of the usual `: ` between the key and value.  A line feed will be
inserted before each appended value.

When appending, a **blank** key should be used — a sequence of spaces with the
same number of characters as the key.

```muon
lyric: Out in the garden
     :>There's half of a heaven
```

**Bool** is a *boolean*: either `true` or `false`.
```muon
earth_is_flat: false
```

**Int** is an *integer* (whole number) in one of three forms:

  * *Decimal*: sequence of digits `0`-`9` (not starting with `0`).  May have a
    `+` or `-` sign prefix
  * *Binary*: `0b` followed by sequence of digits `0` or `1`
  * *Hexadecimal*: `0x` followed by sequence of digits `0`-`9`, `A`-`F` or
    `a`-`f`

An underscore may be inserted between digits to improve readability.

```muon
locke: 4
reyes: 0b1000
ford: 0x0F
jarrah: +16
shephard: 0b01_0111
kwon: 0x2a
```

A **float** is a
[floating point](https://en.wikipedia.org/wiki/IEEE_754) number, made up of
these parts:
  1. *Whole number* part (same as decimal int)
  2. *Fractional* part (decimal point followed by sequence of digits `0`-`9`)
  3. *Exponent* part (`e` followed by decimal int)

One or both of the whole or fractional parts must be present, but the exponent
part is not required.  As with ints, underscores may be included.

The values `inf` and `NaN` stand for *infinity* and *not a number*,
respectively.  Either can be prefixed with a `+` or `-` sign.

```muon
prime: 37
log_e_2: .6931471805599453
mercury: -38.83440
planck: 6.626_070_15e-34
buzz: +inf
avogadro: 6.022_140_76e23
```

**Datetime** is *date*, *time* and *offset*, as specified by `date-time`
from [RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6).  The date and
time must be separated by an upper-case `T`.  If the offset is represented by
`Z`, it must also be uppercase.
```muon
moonwalk: 1969-07-21T02:56:00Z
```

**Date** is *year*, *month* and *day*, as specified by `full-date` from
[RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6).
```muon
birthday: 2019-08-01
```

**Time** is *hour*, *minute* and *second*, as specified by `partial-time` from
[RFC 3339](https://tools.ietf.org/html/rfc3339#section-5.6).
```muon
start: 08:00:00
end: 15:58:14.593849001
```

A **record** is a *branch* containing definitions on subsequent indented lines.
```muon
:::
book: record
  title: text
  author: text
  year: int
:::
book:
  title: If on a winter's night a traveler
  author: Italo Calvino
  year: 1979
```

Since records do not use the values from their definitions, those values
can **substitute** for the first contained mapping, which must then be left out.
Substitution is not allowed for `record`, `dictionary` or `optional` mappings.
```muon
book: The Left Hand of Darkness
  author: Ursula K. Le Guin
  year: 1969
```

A **dictionary** is a *branch* type for associative arrays.  The schema must
contain a single mapping with types for both key and value.  The key type can be
`text`, `bool`, `int`, `float`, `datetime`, `date` or `time`.
```muon
:::
num_word: dictionary
  text: int
:::
num_word:
  fifty: 50
  one: 1
  thirteen: 13
```

**Optional** types are not required — the absence of a definition represents
a `None` or `null` value.
```muon
:::
name: text
occupation: optional text
:::
name: Surfer Joe
# no occupation
```

A **list** is a sequence of *items*, separated by spaces.  If there are no
items in a list, its definition should be omitted.
```muon
:::
show_times: list time
healthy_snacks: list text
:::
show_times: 15:40:00 18:00:00 20:20:00
# no healthy_snacks
```

Like text, lists can be *appended*.  All items are added to the end of the
list.
```muon
fibonacci: 0 1 1 2 3
         : 5 8 13 21 34
# same as fibonacci: 0 1 1 2 3 5 8 13 21 34
```

To append to **list record** or **list dictionary**, since the definitions are
not consecutive, the key cannot be blank.
```muon
person: George Washington
  birthday: 1732-02-22
person: Abraham Lincoln
  birthday: 1809-02-12
```

For **list text**, items are separated by spaces, just like other lists.  If the
text contains spaces, a **text value** separator `:=` can be used to treat an
entire value as a single item.  The *text append* separator `:>` will also
append the value as one item.
```muon
shopping: avocado banana
        :=cream cheese
        : cucumber
        :=ice cream
        : raw
        :>burger! (mmmm)
```
