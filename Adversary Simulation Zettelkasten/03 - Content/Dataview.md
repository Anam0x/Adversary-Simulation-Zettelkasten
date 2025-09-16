---
aliases:
tags:
  - 📝Basic
primary categories:
  - "[[Vault Administration]]"
secondary categories:
  - "[[Obsidian]]"
type: Basic
---
# [[Dataview]]

***

## Description

Adds dynamic views, tables, lists, and calendars by querying your notes' metadata, tags, links, and content.

## Installation

1. Dataview[^1] is a registered Obsidian plugin and can be installed directly from `Settings > Community Plugins > Browse`
	* [[Obsidian - Plugins#Required Plugins]]

## Configuration

* Enable JavaScript queries if you need advanced scripting capabilities (disabled by default for security) by navigating to `Settings > Community Plugins > Dataview` and toggling *Enable JavaScript queries*
* Set inline query prefix (default: =) for inline Dataview expressions by navigating to `Settings > Community Plugins > Dataview > Codeblocks` and setting the *Inline query prefix* value
* Enable/disable automatic task completion date tracking by navigating to `Settings > Community Plugins > Dataview > Tasks` and toggling *Automatic task completion tracking*

## Basic Usage

Every Dataview query consists of:
* Exactly one [Query Type]([[Dataview#Query Types]])
* Zero or one `FROM` data commands with one to many sources
* Zero to many other [Data Commands]([[Dataview#Data Commands]]) with one to many expressions and/or other fields depending on the data command

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

***

## Resources

| Hyperlink                                                                                                                       | Info                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness) | Example implementation of Zettelkasten using Obsidian; demonstrates leveraging Dataview for dynamic query results |
| [Dataview Wiki, Michael Brenan](https://blacksmithgu.github.io/obsidian-dataview/)                                              | Dataview official documentation and syntax guide                                                                  |

[^1]: Dataview Plugin, Michael Brenan, https://github.com/blacksmithgu/obsidian-dataview

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
