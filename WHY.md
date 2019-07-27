## Why MuON?

MuON is a format for configuration files or text data interchange.
But we already have **XML**, **JSON**, **YAML**, **TOML** and dozens of less
popular formats.  Do we really need another one?  Let's compare with a sample
containing a list of books.

----
### XML:
```xml
<book title="Pale Fire"
      author="Vladimir Nabokov"
      year="1962">
  <character name="John Shade"
             location="New Wye" />
  <character name="Charles Kinbote"
             location="Zembla" />
</book>
<book title="The Curious Incident of the Dog in the Night-Time"
      author="Mark Haddon"
      year="2003">
  <character name="Christopher Boone"
             location="Swindon" />
  <character name="Siobhan" />
</book>
```
----
### JSON:
```json
{"book": [
  {
    "title": "Pale Fire",
    "author": "Vladimir Nabokov",
    "year": 1962,
    "character": [
      {"name": "John Shade", "location": "New Wye"},
      {"name": "Charles Kinbote", "location": "Zembla"}
    ]
  },
  {
    "title": "The Curious Incident of the Dog in the Night-Time",
    "author": "Mark Haddon",
    "year": 2003,
    "character": [
      {"name": "Christopher Boone", "location": "Swindon"},
      {"name": "Siobhan"}
    ]
  }
]}
```
----
### YAML:
```yaml
---
book:
- title: Pale Fire
  author: Vladimir Nabokov
  year: '1962'
  character:
  - name: John Shade
    location: New Wye
  - name: Charles Kinbote
    location: Zembla
- title: The Curious Incident of the Dog in the Night-Time
  author: Mark Haddon
  year: '2003'
  character:
  - name: Christopher Boone
    location: Swindon
  - name: Siobhan
```
----
### TOML:
```toml
[[book]]
  title = "Pale Fire"
  author = "Vladimir Nabokov"
  year = 1962

  [[character]]
    name = "John Shade"
    location = "New Wye"

  [[character]]
    name = "Charles Kinbote"
    location = "Zembla"

[[book]]
  title = "The Curious Incident of the Dog in the Night-Time"
  author = "Mark Haddon"
  year = 2003

  [[character]]
    name = "Christopher Boone"
    location = "Swindon"

  [[character]]
    name = "Siobhan"
```
----
### MuON:
```
:::
book: [dict]
  title: text
  author: text
  year: integer?
  character: [dict]
    name: text
    location: text?
:::
book: Pale Fire
  author: Vladimir Nabokov
  year: 1962
  character: John Shade
    location: New Wye
  character: Charles Kinbote
    location: Zembla
book: The Curious Incident of the Dog in the Night-Time
  author: Mark Haddon
  year: 2003
  character: Christopher Boone
    location: Swindon
  character: Siobhan
```
----

The data can be modeled in each of these formats with no problems.  They are all
*readable* by someone with minimal experience in programming.  The difficulty of
*writing* is a bit subjective, but the general trend from hard to easy follows
the order presented: XML, JSON, YAML, TOML, then MuON.  The hardest part about
MuON is writing a schema, which should be a relatively rare task.

Format | Bytes | Minimum | Comments | Schema | Simplicity
-------|-------|---------|----------|--------|-----------
XML    |   441 |     366 |      Yes |    Yes |        4th
JSON   |   467 |     356 |       No |  Draft |         🥉
YAML   | *351* |     335 |      Yes |     No |        5th
TOML   |   433 |     385 |      Yes |     No |         🥈
MuON   |   439 |   *303* |      Yes |  *Yes* |         🥇

The bytes listed is for the nicely formatted versions, as seen above.  The
minimum bytes was obtained by stripping out extraneous whitespace, and in the
case of MuON, the schema.  For TOML and MuON, these minimal versions are still
quite readable — the others less so.

The main point about comments is that they're not supported by JSON.  They *are*
supported in XML, but with an `<!-- obtuse -->` syntax.

Schemas are helpful to validate data types.  Most of these formats have inline
type information, such as JSON's [square brackets] for arrays and {curly braces}
for objects.  But these hints are incomplete without a schema, and redundant
when one is available.  There is a [specification](https://json-schema.org) for
adding schemas to JSON, but it's unlikely to be supported soon by most
implementations.  XML has had schemas for ages, and it's not universally
supported.  MuON was designed from the start to take advantage of schemas.

* XML has fallen out of favor, but it is not as bad as some people claim.  That
  said, it only works well for things like Open Document Format.

* JSON is not a good configuration format, and it's not a great data interchange
  format either.  Its biggest plus is universal support in programming
  languages.

* YAML looks similar to MuON at first glance, but this example does not scratch
  the surface of the
  [complexity hidden in YAML](https://arp242.net/yaml-config.html).  Also, the
  use of hyphens for array items makes it easy to get lost in long files.  MuON
  does not have this problem, since the key is repeated for each list item.

* TOML is good for *some* types of configuration files.  Once data structures
  become nested more than two levels, its "flat" layout becomes more of a
  hinderance, reducing readability.

Simplicity is important to reduce bugs and help with interoperability.  This
ranking may be biased, but it is based on experience working with the formats
as well as the length of each specification.
