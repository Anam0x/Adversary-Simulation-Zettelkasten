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
# [[Obsidian - Callouts]]

---

## Overview

As of [Obsidian v0.14.0](https://obsidian.md/changelog/2022-03-14-desktop-v0.14.0/), Obsidian natively supports callout boxes[^1] (also known as admonitions). Callouts allow you to create visually distinct blocks with different icons, colors, and styles directly in your notes without requiring community plugins.

Callouts are useful for:
- Highlighting important information, warnings, or tips
- Creating structured documentation with clear visual hierarchy
- Drawing attention to critical details within your notes
- Organizing content into categorized blocks

---

## Basic Syntax

The basic syntax for callouts uses blockquote notation with a special identifier:

```markdown
> [!callout-type]
> Your content here.
```

**Example:**

```markdown
> [!note]
> This is a basic note callout.
```

> [!note]
> This is a basic note callout.

---

## Supported Callout Types

Obsidian supports multiple built-in callout types, each with distinct colors and icons:

### Informational Callouts

```markdown
> [!note]
> General notes, references, or related information.

> [!info]
> Highlighting helpful information.

> [!todo]
> Tasks or items to complete.
```

> [!note]
> General notes, references, or related information.

> [!info]
> Highlighting helpful information.

> [!todo]
> Tasks or items to complete.

### Positive Callouts

```markdown
> [!success]
> Indicating completion, success, or confirmation messages.

> [!check]
> Checkpoints or validated information.

> [!done]
> Completed tasks or finished items.
```

> [!success]
> Indicating completion, success, or confirmation messages.

> [!check]
> Checkpoints or validated information.

> [!done]
> Completed tasks or finished items.

### Cautionary Callouts

```markdown
> [!question]
> Questions, FAQs, or prompts for further thinking.

> [!help]
> Help information or assistance needed.

> [!warning]
> Calling out cautions, risks, or important alerts.

> [!caution]
> Situations requiring careful attention.

> [!attention]
> Items requiring immediate focus.
```

> [!question]
> Questions, FAQs, or prompts for further thinking.

> [!help]
> Help information or assistance needed.

> [!warning]
> Calling out cautions, risks, or important alerts.

> [!caution]
> Situations requiring careful attention.

> [!attention]
> Items requiring immediate focus.

### Negative Callouts

```markdown
> [!failure]
> Noting missing items, failed tasks, or critical issues.

> [!fail]
> Failed operations or unsuccessful attempts.

> [!missing]
> Missing information or incomplete sections.

> [!danger]
> Highlighting severe problems, errors, or urgent issues.

> [!error]
> Error messages or problematic situations.

> [!bug]
> Tracking bugs, issues, or debugging notes.
```

> [!failure]
> Noting missing items, failed tasks, or critical issues.

> [!fail]
> Failed operations or unsuccessful attempts.

> [!missing]
> Missing information or incomplete sections.

> [!danger]
> Highlighting severe problems, errors, or urgent issues.

> [!error]
> Error messages or problematic situations.

> [!bug]
> Tracking bugs, issues, or debugging notes.

### Additional Callouts

```markdown
> [!abstract]
> Summarizing content or providing an overview.

> [!tip]
> Helpful tips, best practices, or important details.

> [!example]
> Code examples, demonstrations, or sample content.

> [!quote]
> Highlighting quotes, citations, or references.

> [!cite]
> Formal citations or references.
```

> [!abstract]
> Summarizing content or providing an overview.

> [!tip]
> Helpful tips, best practices, or important details.

> [!example]
> Code examples, demonstrations, or sample content.

> [!quote]
> Highlighting quotes, citations, or references.

> [!cite]
> Formal citations or references.

---

## Callout Titles

You can customize the title of any callout by adding text after the callout type:

```markdown
> [!tip] Pro Tip
> Custom titles help organize and label your callouts.
```

> [!tip] Pro Tip
> Custom titles help organize and label your callouts.

To create a callout without any content (essentially a title-only callout), only populate the title line:

```markdown
> [!warning] This callout is title-only
```

> [!warning] This callout is title-only

---

## Collapsible Callouts

Callouts can be made collapsible (foldable) using `+` or `-` modifiers:

### Expanded by Default

Use `+` to create a callout that is expanded by default:

```markdown
> [!note]+ Click to Collapse
> This callout starts expanded and can be collapsed by clicking the arrow.
> 
> It can contain multiple paragraphs and other content.
```

> [!note]+ Click to Collapse
> This callout starts expanded and can be collapsed by clicking the arrow.
> 
> It can contain multiple paragraphs and other content.

### Collapsed by Default

Use `-` to create a callout that is collapsed by default:

```markdown
> [!warning]- Click to Expand
> This callout starts collapsed and must be clicked to view its contents.
> 
> This is useful for hiding detailed information that isn't always needed.
```

> [!warning]- Click to Expand
> This callout starts collapsed and must be clicked to view its contents.
> 
> This is useful for hiding detailed information that isn't always needed.

---

## Nested Callouts

Callouts can be nested inside other callouts for creating hierarchical information structures:

```markdown
> [!note] Parent Callout
> This is the main callout.
> 
> > [!warning] Nested Warning
> > This warning is nested inside the note callout.
> > 
> > > [!tip] Doubly Nested Tip
> > > You can nest callouts multiple levels deep.
> 
> Back to the parent callout content.
```

> [!note] Parent Callout
> This is the main callout.
> 
> > [!warning] Nested Warning
> > This warning is nested inside the note callout.
> > 
> > > [!tip] Doubly Nested Tip
> > > You can nest callouts multiple levels deep.
> 
> Back to the parent callout content.

---

## Markdown Support in Callouts

Callouts support full Markdown formatting, including:

### Lists

```markdown
> [!tip] Red Team TTPs
> - Reconnaissance
> - Initial Access
> - Execution
> - Persistence
> - Privilege Escalation
```

> [!tip] Red Team TTPs
> - Reconnaissance
> - Initial Access
> - Execution
> - Persistence
> - Privilege Escalation

### Code Blocks

```markdown
> [!example] PowerShell Command
> ```powershell
> Get-ADUser -Filter * -Properties LastLogonDate | 
>   Select-Object Name, LastLogonDate
> ```
```

> [!example] PowerShell Command
> ```powershell
> Get-ADUser -Filter * -Properties LastLogonDate | 
>   Select-Object Name, LastLogonDate
> ```

### Links and Wikilinks

```markdown
> [!info] Related Topics
> See also: [[Active Directory]], [[Kerberoasting]], and [[PowerShell]]
> 
> External resource: [MITRE ATT&CK](https://attack.mitre.org/)
```

> [!info] Related Topics
> See also: [[Active Directory]], [[Kerberoasting]], and [[PowerShell]]
> 
> External resource: [MITRE ATT&CK](https://attack.mitre.org/)

### Tables

```markdown
> [!abstract] Port Summary
> | Port | Service | Protocol |
> | ---- | ------- | -------- |
> | 80   | HTTP    | TCP      |
> | 443  | HTTPS   | TCP      |
> | 22   | SSH     | TCP      |
```

> [!abstract] Port Summary
> | Port | Service | Protocol |
> | ---- | ------- | -------- |
> | 80   | HTTP    | TCP      |
> | 443  | HTTPS   | TCP      |
> | 22   | SSH     | TCP      |

---

## Callout Type Aliases

Many callout types have aliases that produce the same styling. Use whichever name best fits your context:

| Primary Type | Aliases                |
| ------------ | ---------------------- |
| `note`       | `seealso`              |
| `abstract`   | `summary`, `tldr`      |
| `info`       | —                      |
| `todo`       | —                      |
| `tip`        | `hint`, `important`    |
| `success`    | `check`, `done`        |
| `question`   | `help`, `faq`          |
| `warning`    | `caution`, `attention` |
| `failure`    | `fail`, `missing`      |
| `danger`     | `error`                |
| `bug`        | —                      |
| `example`    | —                      |
| `quote`      | `cite`                 |

---

## Limitations

### Footnotes

Footnotes cannot be embedded directly within callout blocks. They must be placed outside the callout:

```markdown
> [!note]
> This callout attempts to use a footnote[^2].
```

> [!note]
> This callout attempts to use a footnote[^2].

It is advised to use inline links instead:

```markdown
> [!note]
> This callout uses an [inline link](https://example.com) instead of a footnote.
```

> [!note]
> This callout uses an [inline link](https://example.com) instead of a footnote.

### Dataview Queries

Dataview queries within callouts may not render correctly in all contexts. Test thoroughly if using dynamic content inside callouts.

---

## Comparison with Admonition Plugin

| Feature               | Native Callouts | Admonition Plugin               |
| --------------------- | --------------- | ------------------------------- |
| Installation Required | No              | Yes                             |
| Syntax                | `> [!type]`     | ` ```ad-type `                  |
| Collapsible           | Yes (`+`, `-`)  | Yes (`collapse:`)               |
| Custom Colors         | Limited         | Yes (RGB values)                |
| Markdown Support      | Full            | Full                            |
| Custom Callout Types  | Via CSS         | Built-in UI                     |
| Performance           | Native (faster) | Plugin (slight overhead)        |
| Future Compatibility  | High            | Dependent on plugin maintenance |

It is recomended to use native callouts for new notes. The [[Admonition]] plugin remains available for legacy compatibility and advanced customization needs.

## Resources

| Hyperlink                                                                                                                                      | Info                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| [Callouts, Obsidian Help](https://help.obsidian.md/Editing+and+formatting/Callouts)                                                            | Official Obsidian documentation for callouts                         |
| [0.14.0 Desktop, Obsidian Changelog](https://obsidian.md/changelog/2022-03-14-desktop-v0.14.0/)                                                | Release notes introducing native callout support                     |
| [How to make Custom Callouts in Obsidian, Brianna Laird](https://briannalaird.com/content/blog-posts/2025-06-17-making-callouts-obsidian.html) | Community member-published guide on styling and customizing callouts |

[^1]: Callouts, Obsidian Help, https://help.obsidian.md/Editing+and+formatting/Callouts
[^2]: Example Page, Google, https://google.com/example

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
