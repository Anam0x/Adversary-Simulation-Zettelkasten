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
# [[Admonition]]

***

## Description

Adds admonition block-styled content to Obsidian.md, styled after [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/reference/admonitions/).

![](https://raw.githubusercontent.com/javalent/admonitions/master/publish/gifs/all.gif)

## Installation

1. Admonition[^1] is a registered Obsidian plugin and can be installed directly from `Settings > Community Plugins > Browse`
	* [[Obsidian - Plugins#Additional]]

## Configuration

* Nothing required beyond installation for default admonition block element display
* Would recommend enabling the 'copy' button permitting the copying of content directly from the blocks

## Basic Usage

### Code Block Style

````markdown
```ad-<type>
title: <Optional Title>
collapse: <none|open|close>
icon: <FontAwesome or RPG Awesome icon name>
color: <R,G,B>
Your content here.
```
````

```ad-info
title: Custom Title
collapse: none
Your content here.
```

#### Supported Parameters

| Parameter   | Description                                                          |
| ----------- | -------------------------------------------------------------------- |
| `title:`    | Optional. Overrides default title; supports Markdown in block-style. |
| `collapse:` | `open`, `close`, or `none`. Controls collapsible behavior.           |
| `icon:`     | Sets a custom icon using FontAwesome or RPG Awesome names.           |
| `color:`    | RGB triad (e.g., `200,200,200`) to override default styling.         |

#### Supported Types

| Type       | Aliases                           | Use Case                                                       |
| ---------- | --------------------------------- | -------------------------------------------------------------- |
| `note`     | `note`, `seealso`                 | General notes, references, or related information.             |
| `abstract` | `abstract`, `summary`, `tldr`     | Summarizing content or providing an overview of a section.     |
| `info`     | `info`, `todo`                    | Highlighting helpful information or listing tasks to complete. |
| `tip`      | `tip`, `hint`, `important`        | Sharing helpful tips, best practices, or important details.    |
| `success`  | `success`, `check`, `done`        | Indicating completion, success, or confirmation messages.      |
| `question` | `question`, `help`, `faq`         | Posing questions, FAQs, or prompts for further thinking.       |
| `warning`  | `warning`, `caution`, `attention` | Calling out cautions, risks, or important alerts.              |
| `failure`  | `failure`, `fail`, `missing`      | Noting missing items, failed tasks, or critical issues.        |
| `danger`   | `danger`, `error`                 | Highlighting severe problems, errors, or urgent issues.        |
| `bug`      | `bug`                             | Tracking bugs, issues, or debugging notes.                     |
| `example`  | `example`                         | Providing code examples, demonstrations, or sample content.    |
| `quote`    | `quote`, `cite`                   | Highlighting quotes, citations, or references.                 |

## Callouts

As of [Obsidian v0.14.0](https://publish.obsidian.md/hub/01+-+Community/Obsidian+Roundup/2022-03-19+Better+Citations+Workflows+%26+Native+Callout+Boxes), Obsidian natively supports admonitions via "callout boxes". This allows you to create visually distinct blocks with different icons and colors directly in your notes without needing to install the community Admonitions plugin.

````markdown
> [!example]
> This is an example callout box.
````

> [!example]
> This is an example callout box.

While the older, plugin-based Admonition syntax with code blocks (` ```ad-note `) still functions, the newer Callout syntax is recommended for future compatibility.

### Nesting

For block-style admonitions, wrap with matching backtick levels.

`````markdown
````ad-note
title: Parent
```ad-warning
title: Nested Child
Nested content
```
Parent content
````

> [!note] Parent
> > [!warning] Nested Child
> > Nested content
> 
> Parent content 
`````

````ad-note
title: Parent
```ad-warning
title: Nested Child
Nested content.
```
Parent content
````

> [!note] Parent
> > [!warning] Nested Child
> > Nested content
> 
> Parent content 
## Constraints

It is not possible to embed a footnote in admonition-styled code blocks or callouts, illustrated below:

````markdown
```ad-example
Attempt to embed a footnote in this codeblock, it won't work[^2].
However, embedded hyperlinks do appear to work, like [this one](https://google.com/example).
```

> [!example]
> Attempt to embed a footnote in this callout, it won't work[^2].
> However, embedded hyperlinks do appear to work, like [this one](https://google.com/example).
````

```ad-example
Attempt to embed a footnote in this codeblock, it won't work[^2].
However, embedded hyperlinks do appear to work, like [this one](https://google.com/example).
```

> [!example]
> Attempt to embed a footnote into this callout, it won't work[^2].
> However, embedded hyperlinks do appear to work, like [this one](https://google.com/example).

It is therefore recommended to avoid embedding footnotes in Admonition code blocks and to use inline hyperlinks instead.

___

## Resources

| Hyperlink                                                                                                                                        | Info                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness)                  | Example implementation of Zettelkasten using Obsidian; showcases usage of Admonition-styled code blocks |
| [Admonition, Obsidian Hub](https://publish.obsidian.md/hub/02+-+Community+Expansions/02.05+All+Community+Expansions/Plugins/obsidian-admonition) | Admonition wiki page maintained by Obsidian community                                                   |

[^1]: Admonition Plugin, Jeremy Valentine, https://github.com/javalent/admonitions
[^2]: Example Page, Google, https://google.com/example

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
