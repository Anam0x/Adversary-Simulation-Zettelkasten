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
# [[Obsidian - Getting Started]]  

---
## Welcome to Anam0x's Obsidian Zettelkasten!

This vault was inspired by TrustedSec's research on using Obsidian[^1] for collaborative offensive security knowledge management.

> [!todo]
> Once the installation and configuration scripts have been added, introduce the following line to this page:
> 
> ``It is recommended that you run the installation and configuration script (`install.ps1` for Windows, `install.sh` for NIX-like systems) prior to using this vault; more instructions are available in the `README.md` file.``

If you're just getting started, be sure to check out the complete blog pertaining to Obsidian on TrustedSec's website[^2]. In addition, dig into a few of the resources cited below which go into substantially more detail surrounding many of the how-tos of Obsidian and the benefits of maintaining a digital [[Zettelkasten]].

Everyone's learning style is different. It is therefore encouraged that you make your own changes to this vault to best accommodate your preferences and workflow.

## Current Minimum Setup

Before creating notes in this vault, make sure you have:
- [ ] Installed the required community plugins in [[Obsidian - Plugins]]
- [ ] Configured [[Obsidian - Templater]] to run `0400 - Gen_Note` on new file creation
- [ ] Enabled Dataview JavaScript queries as described in [[Obsidian - Dataview]]

These are ***not optional*** for the current vault experience. Several note-generation and navigation features assume they are already enabled.

For instructions on how to add your own content and categories, navigate to the [[Obsidian - Vault Structure and Note Creation]] page. Be sure to check out [[Obsidian - Plugins]] along with the individual pages for the following required vault plugins:
* [[Obsidian - Dataview]]
* [[Obsidian - Emoji Toolbar]]
* [[Obsidian - Obsidian Git]]
* [[Obsidian - Templater]]

> [!tip] Native Callouts
> As of Obsidian v0.14.0, callout boxes (admonitions) are natively supported without requiring the [[Obsidian - Admonition]] community plugin. See [[Obsidian - Callouts]] for usage instructions and examples.

## What To Expect From New Notes

When you create new content notes through the Templater workflow:
- The note is created in `03 - Content/`
- Type-specific metadata is prompted during creation
- `note-status` is added automatically
- Dataview visibility is controlled by `note-status`

If a note is still in progress, it should normally stay at:
- `note-status: ✍️ Draft`

and only shift to:
- `note-status: ☑️ Ready`

when you want it to appear in most query-driven views.

___

## Resources

| Reference | Info |
| --------- | ---- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness) | Example implementation of Zettelkasten using Obsidian |

[^1]: Obsidian, Obsidian, https://obsidian.md/
[^2]: Obsidian, Taming a Collective Consciousness; Sam Link; https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness

---
*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
