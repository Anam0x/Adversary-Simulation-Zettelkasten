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
# [[Obsidian Git]]

***

## Description

Simple plugin that allows you to backup your [Obsidian.md](https://obsidian.md) vault to a remote git repository (e.g. private repo on GitHub). This plugin assumes you have existing git repository initialized locally and credentials are setup. This is the mechanism by which all your notes are sync'd to the Offpipe repository and shared between consultants. 

## Installation

1. Obsidian Git is a registered Obsidian plugin and and can be installed directly from `Settings > Community Plugins > Browse`
	* [[Obsidian - Plugins#Required]]

## Configuration

* Navigate to `Settings > Community Plugins > Git` and ensure the following properties are set:
	1. *Auto commit-and-sync interval (minutes)*: `60`
	2. *Auto pull interval (minutes)*: `10`
	3. *Commit message on auto commit-and-sync*: `[hostname OR FLast] {{date}}`
	4. *{{date}} placeholder format*: `MM-DD-YYYY HH:mm:ss`
	5. *Pull on commit-and-synce*: Enabled (default)
	6. *Push on commit-and-synce*: Enabled (default)

## Commands

All associated commands specific to Obsidian git can be reviewed from the Command Pallete (<kbd>CTRL</kbd> + <kbd>P</kbd> to open)

![[Pasted image 20210907112215.png]]

These commands can be added to hotkey combinations which greatly simply the addons use. 

![[Pasted image 20210907112642.png]]

___

## Resources

| Hyperlink                                                                                                                                                          | Info                                                                                                                               |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness)                                    | Example implementation of Zettelkasten using Obsidian; demonstrates how Obsidian Git can be leveraged for vault team collaboration |
| [The Easiest Way to Setup Obsidian Git (to backup notes), Obsidian Forum](https://forum.obsidian.md/t/the-easiest-way-to-setup-obsidian-git-to-backup-notes/51429) | Example implementation of setting up an Obsidian GitHub repository and automating source control with Obsidian Git                 |

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
