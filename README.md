# Adversary Simulation Obsidian Zettelkasten

## Overview

This vault adapts TrustedSec’s approach to using Obsidian for offensive security knowledge management, as described in their blog post [*Obsidian: Taming a Collective Consciousness*](https://www.trustedsec.com/blog/obsidian-taming-a-collective-consciousness/).

I have modified the vault to better reflect my personal methodology, professional interests, and learning style. Additionally, certain features intended for team collaboration have been removed or minimized, as this template repository is primarily intended for personal use.

This repository builds on the original [*Obsidian-Vault-Structure*](https://github.com/trustedsec/Obsidian-Vault-Structure) vault with the following new features:
* **Enhanced Templates**: Expanded template library with automatic back-linking
* **Dynamic Organization**: Auto-categorization and custom search tag assignment
* **Personal Focus**: Streamlined for individual use (team features removed/minimized)
* **Refactored Note Creation Workflow**: Creation of new content types, back-linking categories, and search tag assignment are now integrated into the note creation workflow

## Prerequisites

* [Obsidian](https://obsidian.md/) (latest version)
* `git` (for version control and synchronization)

## Installation

### 1. Create Your Own Repository From Template

This vault is designed as a template. Create your own copy using one of the following methods:

<details>
<summary><strong>Option A: GitHub Web UI (Recommended)</strong></summary>
<br />
1. Navigate to this vault's repository on GitHub

2. Expand the green "Use this template" option in the top right corner

3. Select "Create a new repository"

![](/img/installation-github-01.png)

4. Choose your repository name, description, and visibility options

5. Click the green "Create repository" button

![](/img/installation-github-02.png)

6. Clone your new repository locally:

```
git clone git@github.com:[your-username]/[your-repo-name]
cd [your-repo-name]
```

</details>

<details>
<summary><strong>Option B: GitHub CLI</strong></summary>
<br />
1. Ensure you have successfully authenticated to your GitHub account via the GitHub CLI client utility:

```
gh auth login
```

2. Create a new repository from this template repository:

```
gh repo create [your-repo-name] --template [template-repo-url] [`--public`, `--private`, or `--internal`]
```

3. Clone your new repository locally:

```
gh repo clone [your-username]/[your-repo-name]
cd [your-repo-name]
```

</details>

<details>
<summary><strong>Option C: Manual Clone</strong></summary>
<br />
1. If you prefer to work locally without creating a GitHub repository or want to use a different Git platform (e.g., GitLab), first clone the template repository locally:

```
git clone [this-repository-url]
cd adversary-simulation-zettelkasten
```

2. Remove the original remote and reinitialize:

```
rm -rf .git
git init
```

3. Continue working locally, or push your changes to a new repository on your preferred platform (e.g., GitLab)

</details>

### 2. Open Vault in Obsidian

1. Launch Obsidian

2. Click the "Open folder as a vault" button

![](/img/open-vault-01.png)

3. Navigate to and open the cloned repository folder

### 3. Enable Community Plugins

1. Click the gear icon in the bottom left corner of the screen

![](/img/enable-plugins-01.png)

3. Navigate to `Settings > Community plugins`

4. Click the "Turn on community plugins" button

![](/img/enable-plugins-02.png)

## Required Plugin Installation and Configuration

> [!NOTE]
> In future releases of this template repository I hope to automate the installation and configuration of the required Obsidian community plugins via Bash and PowerShell scripts. For now, the user will have to manually install and configure these.

The vault requires five community plugins for full functionality.

| Plugin        | Key Settings                                        |
| ------------- | --------------------------------------------------- |
| [Obsidian Git](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-git) | *Auto commit-and-sync interval (minutes)*: `60`<br />*Auto pull interval (minutes)*: `10`<br />*Commit message on auto commit-and-sync*: `[hostname OR FirstLast] {{date}}`<br />*{{date}} placeholder format*: `MM-DD-YYYY HH:mm:ss`<br />*Push on commit-and-sync*: ON<br />*Pull on commit-and-sync*: ON |
| [Templater](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/templater-obsidian) | *Template folder location*: `04 - Templates`<br />*Trigger Templater on new file creation*: ON<br />*Enable folder templates*: ON<br />Add new folder templates for `01 - Primary Categories`, `02 - Secondary Categories`, and `03 - Content`<br />Set the folder templates' script to `04 - Templates/0400 - Gen_Note.md` |
| [Dataview](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/dataview) | *Enable inline queries*: ON<br /> *Inline query prefix*: `=`                     |
| [Admonition](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-admonition) | *Add Copy Button*: ON                                     |
| [Emoji Toolbar](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-emoji-toolbar) | Hotkey: <kbd>Win</kbd>+<kbd>Alt</kbd>+<kbd>.</kbd> (default)                         |

### Detailed Plugin Setup Instructions

<details>
<summary><strong>1. Obsidian Git</strong></summary>

##### Obsidian Git Installation

You may skip Obsidian Git installation and configuration if you are not working with remote repositories.

1. In the community plugins settings page of Obsidian, click the "Browse" button

![](/img/obsidian-git-01.png)

2. Search for "Git" in the community plugins browser and select the plugin by Denis Olehov

![](/img/obsidian-git-02.png)

3. Click the "Install" button

![](/img/obsidian-git-03.png)

4. After installation, click the "Enable" button

![](/img/obsidian-git-04.png)

#### Obsidian Git Configuration

1. Click the "Options" button

![](/img/obsidian-git-05.png)

2. Set the "Auto commit-and-sync interval (minutes)" option to `60`

3. Set the "Auto pull interval (minutes)" option to `10`

4. Set the "Commit message on auto commit-and-sync" option to `[hostname OR FirstLast] {{date}}`

5. Set the "{{date}} placeholder format" option to `MM-DD-YYYY HH:mm:ss` or your preferred date format

6. Toggle the "Push on commit-and-sync" and "Pull on commit-and-sync" options to on

![](/img/obsidian-git-06.png)

7. Leave the remaining options to their default values

</details>

<details>
<summary><strong>2. Templater</strong></summary>

#### Templater Installation

1. Search for "Templater" in the community plugins browser and select the plugin by SilentVoid

![](/img/templater-01.png)

2. Click the "Install" button

![](/img/templater-02.png)

3. After installation, click the "Enable" button

![](/img/templater-03.png)

#### Templater Configuration

1. Click the "Options" button

2. Set the "Template folder location" option to `04 - Templates`

3. Toggle the "Trigger Templater on new file creation" option to on

4. Toggle the "Enable folder templates" option to on (a new set of options should appear below)

![](/img/templater-04.png)

5. Click the "Add new folder template" button, search for the following directory, set its Templater script to `04 - Templates/0400 - Gen_Note.md`, and repeat for the remaining directories:

* `01 - Primary Categories`
* `02 - Secondary Categories`
* `03 - Content`

![](/img/templater-05.png)

</details>

<details>
<summary><strong>3. Dataview</strong></summary>

#### Dataview Installation

1. Search for "Dataview" in the community plugins browser and select the plugin by Michael Brenan

![](/img/dataview-01.png)

2. Click the "Install" button

![](/img/dataview-02.png)

3. After installation, click the "Enable" button

![](/img/dataview-03.png)

#### Dataview Configuration

1. Click the "Options" button

2. Toggle "Enable inline queries" option to on

3. If you need advanced scripting capabilities (disabled by default for security), set the "Enable JavaScript queries" option to on

4. Set the "Inline query prefix" option to `=`

5. Enable/disable automatic task completion date tracking by toggling the "Automatic task completion tracking" option to on or off

</details>

<details>
<summary><strong>4. Admonition</strong></summary>

#### Admonition Installation

1. Search for "Admonition" in the community plugins browser and select the plugin by Jeremy Valentine

![](/img/admonition-01.png)

2. Click the "Install" button

![](/img/admonition-02.png)

3. After installation, click the "Enable" button

![](/img/admonition-03.png)

#### Admonition Configuration

1. Click the "Options" button

2. Toggle the "Add Copy Button" option to on for convenience

</details>

<details>
<summary><strong>5. Emoji Toolbar</strong></summary>

#### Emoji Toolbar Installation

1. Search for "Emoji Toolbar" in the community plugins browser and select the plugin by oliveryh

![](/img/emoji-toolbar-01.png)

2. Click the "Install" button

![](/img/emoji-toolbar-02.png)

3. After installation, click the "Enable" button

![](/img/emoji-toolbar-03.png)

#### Emoji Toolbar Configuration

1. Click the "Hotkeys" button

![](/img/emoji-toolbar-04.png)

2. Click the plus icon for "Emoji Toolbar: Open emoji picker" and confirm/modify the hotkey associate with launching the emoji keyboard (the default value is <kbd>Win</kbd>+<kbd>Alt</kbd>+<kbd>.</kbd>)

![](/img/emoji-toolbar-05.png)

</details>

## Verification Checklist

* [ ] All 5 plugins enabled in Community plugins settings
* [ ] Template files present in the `04 - Templates` directory
* [ ] Git status visible in Obsidian status bar (if using Git)
* [ ] Emoji toolbar responds to hotkey
* [ ] New notes auto-apply templates when created in structured folders

## Getting Started

1. Read the `[[Obsidian - Getting Started]]` note in the vault
2. Review the `[[Vault Structure and Note Creation]]` guide
3. Explore example content in the `03 - Content` directory
4. Create your first note using the templates

## Contributing

This vault is structured for personal use but can be adapted for team collaboration. Feel free to modify templates and configurations to match your workflow.

***

**Based on**: [TrustedSec's Obsidian Vault Structure](https://github.com/trustedsec/Obsidian-Vault-Structure)
