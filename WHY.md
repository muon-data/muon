## Why MuON?

MuON is a format for configuration files and data interchange.

### Features

* Ease of reading and *writing*
  * Short specification
* Simplicity
  * Regular syntax
  * No seldom-used escaping rules
  * Only 5 special characters (plus 2 digraphs): *line feed*, *space*, `"`, `#`,
    `:`, `:>` and `:=`
  * Smaller file size
* Schema support
  * All common types supported
  * Default values
  * Optional and list types

### Comparision

A list of books in various formats:

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
