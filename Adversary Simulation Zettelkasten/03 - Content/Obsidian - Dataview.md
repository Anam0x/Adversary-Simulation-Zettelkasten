---
aliases:
tags:
  - 📝Basic
primary-categories:
  - "[[Vault Administration]]"
secondary-categories:
  - "[[Obsidian]]"
type: Basic
note-status: ☑️ Ready
---
# [[Obsidian - Dataview]]

---
## Overview
Adds dynamic views, tables, lists, and calendars by querying your notes' metadata, tags, links, and content.

## Installation

1. Dataview[^1] is a registered Obsidian plugin and can be installed directly from `Settings > Community Plugins > Browse`
	* [[Obsidian - Plugins#Required Plugins]]

## Configuration

* Enable JavaScript queries if you need advanced scripting capabilities (disabled by default for security) by navigating to `Settings > Community Plugins > Dataview` and toggling *Enable JavaScript queries*
* Set inline query prefix (default: =) for inline Dataview expressions by navigating to `Settings > Community Plugins > Dataview > Codeblocks` and setting the *Inline query prefix* value
* Enable/disable automatic task completion date tracking by navigating to `Settings > Community Plugins > Dataview > Tasks` and toggling *Automatic task completion tracking*

> [!important]
> JavaScript queries are required for parts of this vault, most notably the dynamic emoji inventory in [[Obsidian - Emoji Toolbar]].

## Basic Usage

Every Dataview query consists of:
* Exactly one [Query Type]([[Obsidian - Dataview#Query Types]])
* Zero or one `FROM` data commands with one to many sources
* Zero to many other [Data Commands]([[Obsidian - Dataview#Data Commands]]) with one to many expressions and/or other fields depending on the data command

### Query Types

* `TABLE`: Display data in tabular format
* `LIST`: Show results as bulleted lists
* `TASK`: Display tasks with completion status
* `CALENDAR`: Show dates in calendar view

### Data Commands

* `FROM`: Source to collect pages from
* `WHERE`: Filter pages on fields
* `SORT`: Sorts results by one or more fields
* `GROUP BY`: Groups results on a field
* `FLATTEN`: Flatten an array in every row
* `LIMIT N`: Limit results to at most `N` values

## Vault Conventions

This vault uses Dataview in three main ways:
1. `Related Notes` sections inside content notes
2. Category dashboards and discovery views
3. Small utility notes such as [[Obsidian - Emoji Toolbar]]

When writing new queries for this vault, prefer these conventions:
- Filter content visibility with `note-status`
- Use typed relationship fields instead of generic `related` lists where possible
- Query template content types from `04 - Templates/Content` when the goal is taxonomy or schema discovery
- Use `dataviewjs` only when plain DQL is too limited for the task

### Publication Filter Pattern

Most content-note discovery queries should treat published content like this:
````
```dataview
LIST
WHERE (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
LIMIT 5
```
````

```dataview
LIST
WHERE (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
LIMIT 5
```

This includes:
- Explicitly ready notes
- Older notes that predate the status model

and excludes:
- Notes marked `✍️ Draft`

## Example Queries

List all notes in the `01 - Primary Categories` directory:
````
```dataview
LIST
FROM "01 - Primary Categories" 
```
````

```dataview
LIST
FROM "01 - Primary Categories" 
```

List all notes tagged with the "🥇Primary_Category" tag not including files in the `04 - Templates` directory in ascending alphabetical order (should be equivalent to the query results above):
````
```dataview
LIST
FROM #🥇Primary_Category and -"04 - Templates"
SORT file.name ASC
```
````

```dataview
LIST
FROM #🥇Primary_Category and -"04 - Templates"
SORT file.name ASC
```

Table of primary categories with last-created and -modified dates:
````
```dataview
TABLE file.ctime as "Created", file.mtime as "Modified"
FROM "01 - Primary Categories"
SORT file.ctime DESC
```
````

```dataview
TABLE file.ctime as "Created", file.mtime as "Modified"
FROM "01 - Primary Categories"
SORT file.ctime DESC
```

List published content notes only:
````
```dataview
TABLE type, file.mtime AS "Modified"
FROM "03 - Content"
WHERE (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
SORT file.mtime DESC
```
````

```dataview
TABLE type, file.mtime AS "Modified"
FROM "03 - Content"
WHERE (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
SORT file.mtime DESC
```

---
## Resources

| Reference | Info |
| --------- | ---- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness) | Example implementation of Zettelkasten using Obsidian; demonstrates leveraging Dataview for dynamic query results |
| [Dataview Wiki, Michael Brenan](https://blacksmithgu.github.io/obsidian-dataview/)                                              | Dataview official documentation and syntax guide                                                                  |

[^1]: Dataview Plugin, Michael Brenan, https://github.com/blacksmithgu/obsidian-dataview

---
*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
