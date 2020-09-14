## Why MuON?

MuON is a format for configuration files and data interchange.

### Features

* Ease of reading and *writing*
  * Short specification
  * Comments!
* Simplicity
  * Regular syntax
  * No seldom-used escaping rules
  * Only a few special characters: *line feed*, *space*, `"`, `#` and `:`
  * Smaller typical file size
* First-class schema support
  * Common types supported
  * Constraints
  * Default values
  * Optional and list types
  * Choice values (sum types)
  * _Any_ type for schemaless branches

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
```muon
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

### Implementations

* Rust: [muon-rs](https://github.com/muon-data/muon-rs)

If you know of any others, please make a PR with a link!
