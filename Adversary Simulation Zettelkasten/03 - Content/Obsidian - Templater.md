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
# [[Obsidian - Templater]]

---
## Overview
Templater[^1] is a template language that lets you insert **variables** and **functions** results into your Obsidian[^2] notes. 

![templater_demo](https://github.com/SilentVoid13/Templater/blob/561ac7bb30dbc2aff6aeab0dd7aa9883bef4fca8/imgs/templater_demo.gif?raw=true)

## How We'll Use Templater

- Run the vault's main note/category creation workflow through `0400 - Gen_Note`
- Prompt for note type, title, category links, content type, and type-specific properties
- Apply controlled vocabularies and typed-link selection during creation
- Generate frontmatter, body, Dataview, and footer sections from the active templates
- Set `note-status` automatically for new content notes
- Move newly created notes into the appropriate vault location
- Leave behind a safe recovery draft if the workflow is cancelled or fails late

## Installation

Templater is a registered Obsidian plugin and can be installed directly from `Settings > Community Plugins > Browse` (see [[Obsidian - Plugins#Required]] for more details).

## Configuration

> [!info] Templater Configuration Instructions
> ### Required Values
> 1. *Template Folder Location*: `04 - Templates`
> 2. *Trigger Templater on New File Creation*: **True**
> 3. *Empty File Template*: `04 - Templates/0400 - Gen_Note`

### Recommended Supporting Settings

- `Settings > Files & Links > Folder to create new notes in`
	- Set this to a temporary holding location if you want incomplete or cancelled notes isolated from your main content flow
- `Settings > Editor > Properties in document`
	- Use the mode that best fits your editing style, but remember the vault expects frontmatter-backed properties

## Vault-Specific Workflow

The main `0400 - Gen_Note` workflow currently handles:
1. Note type selection
2. Title validation and fallback handling
3. Emoji selection for primary categories and content types
4. Category back-linking
5. Content-type selection or custom content-type creation
6. Prompting for controlled values and typed relationships
7. Final assembly of `Metadata`, `Body`, `Dataview`, and `Footer`

For content notes, the workflow now defaults new notes to:
- `note-status: ✍️ Draft`
- Direct creation in `03 - Content/`

Draft visibility is controlled through `note-status`, not through a dedicated drafts directory or promotion script.

## Failure Recovery

If the workflow is cancelled or fails after it has already started, Templater now leaves behind a safe recovery draft instead of malformed frontmatter.

In practice this means:
- Partially created content notes keep a minimal valid frontmatter block
- `note-status` is preserved as a draft signal
- Placeholder metadata can remain in place for later cleanup

___

# Resources

| Reference | Info |
| --------- | ---- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness)                | Example implementation of Zettelkasten using Obsidian; showcases usage of Templater-automated note creation |
| [Templater, Obsidian Hub](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/templater-obsidian) | Templater wiki page maintained by Obsidian community                                                        |

[^1]: Templater Plugin, SilentVoid13, https://github.com/SilentVoid13/Templater
[^2]: Obsidian, Obsidian, https://obsidian.md/

---
*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
