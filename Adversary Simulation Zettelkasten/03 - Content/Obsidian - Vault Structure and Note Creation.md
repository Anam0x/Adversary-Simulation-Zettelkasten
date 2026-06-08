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
# [[Obsidian - Vault Structure and Note Creation]]

---
## Overview

This guide covers how to create and organize content in the Adversary Simulation Zettelkasten vault using the automated note creation system. You'll learn to add primary/secondary categories, content notes, and custom content types through the interactive workflow.

## Vault Structure

The vault uses a hierarchical organization system with four main directories:

```
.
├── Adversary Simulation Zettelkasten/
│   ├── 000 - Global Index (Start Here!).md
│   ├── 01 - Primary Categories/
│   │   ├── Primary Category 1.md
│   │   └── Primary Category 2.md
│   ├── 02 - Secondary Categories/
│   │   ├── Secondary Category 1.md
│   │   └── Secondary Category 2.md
│   ├── 03 - Content/
│   │   ├── Note 1.md
│   │   ├── Note 2.md
│   │   └── Note 3.md
│   └── 04 - Templates/
│       ├── Primary Category/
|       │   ├── Body.md
│       │   └── Metadata.md
│       ├── Secondary Category/
│       │   ├── Body.md
│       │   └── Metadata.md
│       └── Content/
│           ├── Basic/
│           │   ├── Body.md
│           │   ├── Dataview.md
│           │   ├── Footer.md
│           │   └── Metadata.md
│           ├── Custom Type 1/
│           │   ├── Body.md
│           │   ├── Dataview.md
│           │   ├── Footer.md
│           │   └── Metadata.md
│           └── Custom Type 2/
│               ├── Body.md
│               ├── Dataview.md
│               ├── Footer.md
│               └── Metadata.md

└── README.md
```

### Non-Rigid Hierarchical Organization

**Primary Categories** (🥇) represent high-level domains like "Penetration Testing," "Red Team Operations," or "Active Directory." These serve as the main organizational pillars.

**Secondary Categories** (🥈) provide mid-level organization within primary categories, such as "Metasploit," "PowerShell," or "Domain Enumeration."

**Content Notes** (⚛️) are atomic, actionable notes containing specific techniques, tools, payloads, or procedures. Each content note links back to relevant primary and secondary categories.

Whereas in traditional note-taking systems, notes can only have a parent-child relationship with a single parent node, the Zettelkasten system enables flexible many-to-many relationships. Content notes can link to multiple primary and secondary categories simultaneously, secondary categories can relate to several primary domains, and cross-category connections emerge naturally through shared content. This networked approach reflects the interconnected nature of adversary simulation knowledge, where techniques often span multiple domains and tactical procedures may apply across different operational contexts.

### Template System

The template system enables consistent note formatting and automated content generation. Each note type has dedicated templates in the `04 - Templates` directory:

* **Content notes** have four components:
	1. *Metadata*: frontmatter with tags and properties
	2. *Body*: main content structure
	3. *Dataview*: related-note discovery queries
	4. *Footer*: references and annotation footnotes
* **Primary and secondary categories** use *Metadata* and *Body* templates for standardized organization

Custom content types with personalized templates can be created on-demand at the content type selection phase of the note creation workflow.

## Note Creation Workflow

All notes are created using the automated Templater[^1] script. The system guides you through type selection, categorization, and content structuring. To trigger the note creation workflow, right-click any one of the following three directories and select the `New note` option.

1. `01 - Primary Categories`
2. `02 - Secondary Categories`
3. `03 - Content`

### Selecting Note Type

When creating notes, you must choose a note type to ensure consistent structure and organization.

![[vault-structure-01.gif|700]]

Note type selection is mandatory because it determines the note's template structure, search tags, and file system destination. This ensures every content note follows consistent formatting standards and maintains proper vault organization. The system prevents the creation of unstructured notes that could disrupt the knowledge management workflow.

### Primary Categories

Primary categories represent the highest level of organization in your vault. All new notes designated as primary categories will be automatically relocated to the `01 - Primary Categories` directory.

The user will be prompted to provide a valid title for new primary categories. All note titles (including secondary categories and content notes) must be non-empty, not contain illegal characters, be less than 100 characters in length, and not have the same title as another note in its destination directory.

![[vault-structure-02.gif|700]]

If the user fails to provide a valid title within five attempts, they will be prompted to 1) use a fallback option, 2) cancel the note creation workflow, or 3) make a final attempt at providing a valid title. If the user fails to provide a valid title on the final attempt then the fallback option will be used to allow the note creation workflow to continue.

![[vault-structure-03.gif|700]]

Secondary categories and content notes use the same title validation workflow.

#### Search Tag Emoji

Primary categories (and content types, which we will explore later) require emoji selection for visual identification and search tag creation.

![[vault-structure-04.gif|700]]

The emoji selection interface provides categorized options relevant to adversary simulation operations. Each emoji serves as both a visual identifier and part of the searchable tag system.

Users can choose a search tag emoji from the categorized options list. Alternatively, they can select the `"🎲 Random selection"` option to allow the system to handle search tag emoji selection. The user can also manually enter a search tag emoji by selecting the `"✏️ Enter emoji manually"` option.

> [!tip]
> The [[Obsidian - Emoji Toolbar]] keyboard will not be available to you when you are redirected to the manual emoji entry system prompt, so it is advised to have your desired emoji copied to your clipboard before this step.

#### Emoji Validation and Error Handling

The system validates manual emoji input to prevent tag display issues in Obsidian. Common validation issues include:
* **Zero-width joiner emoji sequences** (👨‍💻, 🕵️‍♂️) may cause tag rendering problems
* **Reserved emojis** (🥇, 🥈, ⚛️) are used for vault administration
* **Multiple characters** or **invisible characters** can break tag functionality

![[vault-structure-05.gif|700]]

The validation system provides specific error messages and suggestions for resolving issues, with options to override warnings for experienced users.

The manual emoji prompt uses a similar five-count retry loop and final attempt option as the title validation workflow. If the user fails to provide a valid emoji within five attempts, they will be prompted to 1) use a fallback option, 2) cancel the note creation workflow, or 3) make a final attempt at providing a valid title. If the user fails to provide a valid title on the final attempt then the fallback option will be used to allow the note creation workflow to continue.

![[vault-structure-06.gif|700]]

#### Search Tag Implementation

Primary category emojis become part of structured search tags that enable efficient vault navigation. For example, a "Web Application Testing" primary category with the 🌐 emoji creates the tag `🌐Web_Application_Testing`, allowing rapid identification and filtering of related notes throughout the vault.

Content types also implement search tag emojis. For example, a "Lab Setup" content type with the 🧪 emoji creates the tag `🧪Lab_Setup`, and all new notes created using the "Lab Setup" template will be embedded with that search tag.

#### Primary Category Description

After the emoji step, the workflow prompts for a short 1-2 sentence description of the primary category. This description is inserted directly under the `## Overview` section of the generated primary-category note so that each category starts with a concise statement of scope and purpose.

![[vault-structure-07.gif|700]]

The goal of this prompt is to make new categories immediately useful instead of leaving behind an empty shell. Even a short description helps establish what the category is meant to cover, what kinds of notes belong inside it, and how it differs from nearby primary categories.

#### Final Primary Category Result

Once the workflow finishes, the new primary category is moved into `01 - Primary Categories/` and rendered with:
- Frontmatter metadata
- An `## Overview` section containing the prompted description
- Category-specific Dataview views from the current primary-category body template
- Standard created/modified timestamps

![[vault-structure-08.gif|700]]

### Secondary Categories

Secondary categories serve as specialized hubs for related techniques, tools, or procedures within broader primary category contexts. They enable more granular organization without losing the higher-level categorical structure. All new notes designated as secondary categories will be automatically relocated to the `02 - Secondary Categories` directory.

![[vault-structure-09.gif|700]]

#### Linking to Primary Categories

New secondary categories establish relationships with existing primary categories through wiki-link references.

![[vault-structure-10.gif|700]]

This linking system creates a hierarchical knowledge graph where secondary categories inherit context from their primary categories. The multi-select interface allows secondary categories to relate to multiple primary domains when appropriate, supporting complex and non-rigid organizational relationships.

#### Secondary Category Description

After the user selects the relevant primary categories, the workflow prompts for a short 1-2 sentence description of the secondary category. Like the primary-category description prompt, this text is inserted under `## Overview` so the note begins with a clear explanation of what the category is for.

![[vault-structure-11.gif|700]]

#### Final Secondary Category Result

When the workflow completes, the note is moved into `02 - Secondary Categories/` and generated with:
- Frontmatter metadata
- The selected primary-category backlinks
- An `## Overview` section containing the prompted description
- Category-specific Dataview views from the current secondary-category body template
- Standard created/modified timestamps

![[vault-structure-12.gif|700]]

### Content

Content notes represent the atomic knowledge units in your vault (e.g., specific techniques, tools, procedures, or observations). All new notes designated as content notes will be automatically relocated to the `03 - Content` directory.

![[vault-structure-13.gif|700]]

#### Category Back-Linking

Content notes establish bidirectional relationships with both primary and secondary categories. The dual-linking system ensures content notes remain discoverable within the broader organizational framework while maintaining their atomic nature.

![[vault-structure-14.gif|700]]

Content notes should focus on a single actionable atomic idea (e.g., a specific command syntax, payload example, configuration steps, or tactical procedure) with bidirectional category relationships supporting both top-down navigation (from categories to content) and bottom-up discovery (from content to related categories).

If a category picker is left empty, the workflow can continue intentionally rather than forcing a brittle one-shot selection. This makes it easier to capture cross-domain or not-yet-categorized ideas without breaking the note-creation flow.

### Content Types

After back-linking a new content note with primary and secondary categories, the user will be prompted to select a content type for the new content note. New custom content types are created under `04 - Templates/Content/` in their own sub-directory; the sub-directory contains the structural elements `Metadata.md`, `Body.md`, `Dataview.md`, and `Footer.md`.

![[vault-structure-15.gif|700]]

Content types determine the template structure and formatting for content notes. The system supports both existing content types and on-demand creation of new types.

In addition to determining structure, content types also determine:
- Which metadata properties are prompted during creation
- Which controlled vocabularies are used
- Which typed relationship fields appear in frontmatter
- Which Dataview sections are assembled into the final note

#### Using Existing Content Types

Existing content types provide consistent template structures for common note formats.

![[vault-structure-16.gif|700]]

The "Basic" content type offers a general-purpose template suitable for most content, while specialized types provide focused structures for specific use cases (e.g., tool documentation, payload libraries, or procedure checklists).

For built-in content types, the workflow also prompts for the frontmatter properties defined in the current Templater schema. Depending on the selected content type, that can include:
- Controlled-value properties such as `severity`, `tactic`, or `resource-type`
- Controlled-list properties such as `platforms`
- Typed note relationships such as `uses-tools`, `related-tradecraft`, or `covers-tools`

![[vault-structure-17.gif|700]]

#### Creating New Content Types

Custom content type creation enables template personalization for specialized organizational needs. New content types use the same note title and search tag emoji validation loop as other notes.

![[vault-structure-18.gif|700]]

The template structure of a new content type is automatically generated from the "Basic" template. This can be modified after the note creation workflow to introduce new elements to any of the four structural components of the content type. This enables building a library of specialized templates tailored to your specific red team documentation needs.

Content types enable:
* Consistent information structure across similar notes
* Specialized templates for different data types
* Effective filtering and search capabilities
* Scalable organization as vault content grows

#### Dataview Query Selection For New Content Types

When creating a new custom content type, the workflow includes a Dataview query-tier selection step. This controls how much starter related-note scaffolding is generated inside the new `Dataview.md` file.

The current tiers are:
- `None`
	- Minimal placeholder only
- `Lightweight`
	- A restrained `## Related Notes` layout intended for simpler note types
- `Rich`
	- A fuller discovery layout for note types expected to benefit from broader Dataview support

This tier does not lock the content type permanently. It simply determines the initial scaffold, which you can edit later after the template exists.

In practice:
- Use `None` when the content type is highly specialized or unlikely to need automatic discovery
- Use `Lightweight` when you want a modest set of typed/same-classification queries
- Use `Rich` when the note type is meant to become a discovery hub

The starter queries are intentionally generic. They are meant to render immediately for a brand-new custom content type, while also making it obvious where you should later replace the placeholders with type-specific Dataview queries.

`Lightweight` creates:
- a `Typed Relationships` discovery block implemented in `dataviewjs`
- a `Same Classification` Dataview query that finds other published notes with the same `type`

````md
## Related Notes

### Typed Relationships

```dataviewjs
const relationshipPrefixes = [
  "related-", "uses-", "supports-", "covers-", "required-",
  "detects-", "implements-", "targets-", "abused-by-",
  "secured-by-", "associated-", "exploited-", "practices-"
];
const current = dv.current();
const relationshipRows = Object.entries(current)
  .filter(([key, value]) => relationshipPrefixes.some(prefix => key.startsWith(prefix)))
  .map(([key, value]) => {
    const values = Array.isArray(value) ? value.filter(Boolean) : (value ? [value] : []);
    return [key, values.length ? values.map(item => dv.fileLink(String(item).replace(/^\[\[|\]\]$/g, ""))) : ["(empty)"]];
  });
if (relationshipRows.length) {
  dv.table(["Relationship Property", "Current Links"], relationshipRows);
} else {
  dv.paragraph("No typed relationship properties have been populated yet.");
}
```

> [!todo]
> Replace this summary table with dedicated typed-relationship queries after you finalize this content type's metadata schema.

### Same Classification

```dataview
LIST
FROM "03 - Content"
WHERE type = this.type
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
SORT file.name ASC
LIMIT 10
```
````

`Rich` creates everything in `Lightweight`, plus a `Reverse Relationships` query that finds published notes already linking back to the current note:

````md
## Related Notes

### Same Classification

```dataview
LIST
FROM "03 - Content"
WHERE type = this.type
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

```dataviewjs
const relationshipPrefixes = [
  "related-", "uses-", "supports-", "covers-", "required-",
  "detects-", "implements-", "targets-", "abused-by-",
  "secured-by-", "associated-", "exploited-", "practices-"
];
const current = dv.current();
const relationshipRows = Object.entries(current)
  .filter(([key, value]) => relationshipPrefixes.some(prefix => key.startsWith(prefix)))
  .map(([key, value]) => {
    const values = Array.isArray(value) ? value.filter(Boolean) : (value ? [value] : []);
    return [key, values.length ? values.map(item => dv.fileLink(String(item).replace(/^\[\[|\]\]$/g, ""))) : ["(empty)"]];
  });
if (relationshipRows.length) {
  dv.table(["Relationship Property", "Current Links"], relationshipRows);
} else {
  dv.paragraph("No typed relationship properties have been populated yet.");
}
```

> [!todo]
> Replace this summary table with dedicated typed-relationship queries after you finalize this content type's metadata schema.

### Reverse Relationships

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  type AS "Content Type",
  file.mtime AS "Modified"
FROM "03 - Content"
WHERE (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND contains(file.outlinks, this.file.link)
SORT file.mtime DESC
LIMIT 10
```
````

`Lightweight` is a good fit when you mainly want a starter note with self-discovery and schema reminders. `Rich` is a better fit when the new content type is expected to become a stronger hub in the vault graph.

### Draft Visibility

New content notes are created directly in `03 - Content/` and use:
- `note-status: ✍️ Draft`

as the initial workflow state.

Dataview visibility is controlled through `note-status`, not through a draft directory or promotion script. Notes should generally move to:
- `note-status: ☑️ Ready`

when they are ready to appear in most query-driven views.

## Best Practices

A structured approach ensures your Adversary Simulation vault scales effectively while maintaining organizational clarity and supporting efficient knowledge retrieval.

### Category Design

* **Primary categories** should represent major operational domains
* **Secondary categories** should focus on sub-domains that can be further broken down into granular notes
* Maintain **logical and non-rigid hierarchical relationships** between primary and secondary categories

### Content Creation

* Keep content notes **atomic and focused** on single concepts
* Use **descriptive titles** that clearly indicate content purpose
* Establish **new categories and relationships** when content spans domains
* Alternatively, **create categories first** and flesh them out with **new content notes**

### Template Management

* Create **specialized content types** for recurring note formats
* Use **self-explanatory emoji selection** that aids visual navigation
* Maintain **template consistency** across related content types

### Search and Organization

* Leverage **emoji-based search tags** for rapid content filtering and querying
* Use **wiki-link relationships** for knowledge graph navigation
* Maintain **consistent naming conventions** across categories and content

---
## Resources

| Reference | Info |
| --------- | ---- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness) | Example implementation of Zettelkasten using Obsidian; deep dive into vault structure and automation procedures |

[^1]: Templater Plugin, SilentVoid13, https://github.com/SilentVoid13/Templater

---
*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
