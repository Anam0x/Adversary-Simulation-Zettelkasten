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
# [[Obsidian - Plugins]]  

---

## Overview

This vault uses Obsidian Community Plugins[^1] to automate the process of note creation, source control, data querying, and other essential functions.

> [!todo]
> Once the installation and configuration scripts have been added, introduce the following line to this page:
> 
> ``It is recommended that you run the installation and configuration script (`install.ps1` for Windows, `install.sh` for NIX-like systems) prior to using this vault; more instructions are available in the `README.md` file.``

## Required Plugins

The following plugins are required for the Adversary Simulation Tradecraft Zettelkasten.

### Obsidian Git

<iframe src="https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-git" width="150" height="300" style="overflow: auto; resize: both; aspect-ratio: 16 / 9; width: 100%; height: 100%;"></iframe>

The [[Obsidian - Obsidian Git]][^2] plugin provides seamless Git integration within Obsidian, enabling automatic version control and synchronization of your vault across multiple devices. It supports automated commits, push/pull operations, and conflict resolution, making cross-device knowledge management possible while maintaining a complete history of changes. This plugin is essential for backing up your research and maintaining synchronization on multiple devices or in team environments, if you choose to share your notes.

### Templater

<iframe src="https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/templater-obsidian" width="150" height="300" style="overflow: auto; resize: both; aspect-ratio: 16 / 9; width: 100%; height: 100%;"></iframe>

The [[Obsidian - Templater]][^3] plugin extends Obsidian's built-in template functionality with advanced scripting capabilities and dynamic content generation. It allows for the creation of sophisticated note templates with variables, JavaScript execution, and system integration. In this vault, Templater automates the note creation workflow, ensuring consistent formatting and metadata across all content types while reducing manual overhead.

### Dataview

<iframe src="https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/dataview" width="150" height="300" style="overflow: auto; resize: both; aspect-ratio: 16 / 9; width: 100%; height: 100%;"></iframe>

The [[Obsidian - Dataview]][^4] plugin transforms Obsidian into a powerful database by enabling SQL-like queries over your notes and their metadata. It can generate dynamic tables, lists, and charts based on tags, frontmatter, and content, making it possible to create automated indexes and dashboards. This plugin is crucial for organizing and discovering content within large knowledge bases.

### Emoji Toolbar

<iframe src="https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-emoji-toolbar" width="150" height="300" style="overflow: auto; resize: both; aspect-ratio: 16 / 9; width: 100%; height: 100%;"></iframe>

The [[Obsidian - Emoji Toolbar]][^5] plugin provides a user-friendly interface for inserting emojis and Unicode symbols into notes. Beyond basic emoji insertion, it supports custom emoji sets and can be configured to work with specific tagging systems. In this vault, it facilitates the consistent use of emoji-based categorization and visual organization of primary categories and content types.

## Optional Plugins

> [!todo]
Experiment more with suggested plugins and add to this section as-needed.

### Admonition

<iframe src="https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-admonition" width="150" height="300" style="overflow: auto; resize: both; aspect-ratio: 16 / 9; width: 100%; height: 100%;"></iframe>

> [!important] Native Callout Support
> As of [Obsidian v0.14.0](https://obsidian.md/changelog/2022-03-19-desktop-v0.14.0/), callout boxes (admonitions) are natively supported without requiring this plugin. For most use cases, native callouts are recommended.

The [[Obsidian - Admonition]][^6] plugin adds support for visually distinct code blocks that can highlight important information, warnings, tips, and other content types. It provides a variety of predefined styles and supports custom formatting, making documentation more readable and professionally formatted. These code blocks help organize information hierarchically and draw attention to critical details.

The Admonition plugin is now considered optional for this vault. While Obsidian's native callouts provide the core functionality for creating visually distinct information blocks, you may still choose to install the Admonition plugin for:

- **Legacy compatibility**: If working with vaults that use the older syntax (` ```ad-type `)
- **Advanced customization**: Access to additional styling options beyond native callouts
- **Custom callout types**: Creating specialized admonition types with unique styling
- **RGB color control**: Fine-grained color customization not available in native callouts

For new notes, it is recommended to use native callouts (`> [!type]` syntax) for better performance and future compatibility. See the [[Obsidian - Callouts]] page for a detailed comparison between the plugin and native callout features.

___

## Resources

| Reference | Info |
| --------- | ---- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness)                        | Example implementation of Zettelkasten using Obsidian; discusses the importance of multiple community Obsidian plugins |
| [Obsidian Git, Obsidian Hub](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-git)            | Obsidian Git wiki page maintained by Obsidian community                                                                |
| [Templater, Obsidian Hub](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/templater-obsidian)         | Templater wiki page maintained by Obsidian community                                                                   |
| [Dataview, Obsidian Hub](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/dataview)                    | Dataview wiki page maintained by Obsidian community                                                                    |
| [Admonition, Obsidian Hub](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-admonition)       | Admonition wiki page maintained by Obsidian community                                                                  |
| [Emoji Toolbar, Obsidian Hub](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-emoji-toolbar) | Emoji Toolbar wiki page maintained by Obsidian community                                                               |

[^1]: Obsidian Community Plugins, Obsidian, https://help.obsidian.md/community-plugins
[^2]: Obsidian Git Plugin, Denis Olehov, https://github.com/denolehov/obsidian-git
[^3]: Templater Plugin, SilentVoid13, https://github.com/SilentVoid13/Templater
[^4]: Dataview Plugin, Michael Brenan, https://github.com/blacksmithgu/obsidian-dataview
[^5]: Emoji Toolbar Plugin, oliveryh, https://github.com/oliveryh/obsidian-emoji-toolbar
[^6]: Admonition Plugin, Jeremy Valentine, https://github.com/javalent/admonitions

---
*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
