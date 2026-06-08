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
# [[Obsidian - Properties]]

---
## Overview

This note documents the official approach to using Obsidian frontmatter properties in the Adversary Simulation Tradecraft Zettelkasten. Properties exist to support classification, navigation, and targeted Dataview queries without turning note creation into excessive data entry.

The vault therefore follows three design principles:

1. Properties should primarily support **discovery**, not replace thoughtful writing
2. Properties should remain **minimal by default** and only expand when they unlock clear query value
3. Properties should use **typed relationships** where possible so that links carry explicit meaning

## Property Types

Obsidian properties used in this vault may be any of the following:
- **Text**: plain text values, including single Obsidian links when appropriate
- **Lists**: multi-value fields, commonly used for links or controlled vocabulary
- **Numbers**: quantitative values such as scores
- **Booleans**: `true`/`false`
- **Dates**: calendar dates
- **Date & Time**: timestamps when more precision is useful

## Universal Content Properties

All content notes should preserve the following core properties:
- `aliases`
- `tags`
- `primary-categories`
- `secondary-categories`
- `type`

These fields are the universal classification layer for the vault. They support broad discovery across content types and keep every note connected to the larger knowledge graph.

## Classification First, Explicit Links Second

This vault is designed around **shared classification first**. Most Dataview queries should answer questions such as:
- Which notes share this content type?
- Which notes belong to the same primary or secondary categories?
- Which notes share the same operational classification?

Explicit note-to-note links are still encouraged, but they should usually appear in one of two places:
1. In the note body, when the link is contextual, explanatory, or part of the narrative
2. In a typed metadata property, when the link expresses a durable relationship that should power enriched Dataview queries

## Typed Relationship Properties

This vault avoids ambiguous catch-all relationship fields such as `related`. Instead, content templates may define **typed relationship properties** when a relationship is:
- Semantically clear
- Likely to recur across many notes of that type
- Valuable enough to justify dedicated queries

Examples:
- `known-bypasses`
- `detects-tradecraft`
- `implements-tradecraft`
- `required-tools`
- `required-tradecraft`
- `exploits-vulnerabilities`

Typed relationship properties make queries more meaningful than generic backlink collections because they explain *why* two notes are connected.

## Tradecraft Terminology

This vault is moving away from the generic label **Technique** in favor of **Tradecraft**.

The reason is semantic clarity. In adversary simulation, "technique" is often interpreted as a specific subcomponent of the broader TTP model. That can create confusion when a note may reasonably describe:
- A tactic
- A technique
- A procedure
- Or a hybrid of multiple TTP elements

`Tradecraft` is intentionally broader. It is the preferred label for content notes that document adversary behavior, operational methods, procedural knowledge, or ATT&CK-aligned content.

When a note needs more precise classification, that precision should be expressed through properties such as:
- `attack-id`
- `tactic`
- `platforms`
- `permissions-required`
- `data-sources`

Rather than by relying on the note type name alone.

## Metadata Budgeting

Metadata budgets are not uniform across all content types. Different note types provide different retrieval value, so some templates justify richer schemas than others.

### Default Budget

Most content types should remain within a **small metadata budget**. A reasonable default is:
- Universal content properties
- Plus **1-3** type-specific properties

This keeps the note lightweight and prevents templates from becoming forms.

### Expanded Budget

Some content types act as major retrieval hubs and may justify a **moderate metadata budget**:
- Universal content properties
- Plus **4-6** type-specific properties

Examples include:
- `Attack Surface`
- `Command`
- `Infrastructure`
- `Lab Setup`
- `Playbook`
- `Protocol`
- `Vulnerability`

### High-Value Budget

A small number of content types may justify a **larger metadata budget** when they serve as high-value operational indexes.

These include:
- `Tradecraft`
- `Tool`
- `Security Control`

For these, a richer property model is acceptable when the fields are:
- Stable over time
- Repeatedly useful in Dataview queries
- Operationally meaningful

Even for these notes, verbose prose fields such as long descriptions or detailed procedures should stay in the body rather than the metadata.

## When To Add A New Property

Before adding a new template property, apply the following test:

1. Does the property describe a relationship or classification that recurs often?
2. Will the property support at least one query that is likely to be used repeatedly?
3. Is the property concise enough to stay accurate over time?
4. Would this information be worse if it only lived in the body text?

If the answer to most of these questions is "no," the information likely belongs in the note body instead.

## Recommended Property Strategy

In practice, content templates in this vault should follow this order of priority:
1. Use **core classification properties** to support broad discovery
2. Add **typed relationship properties** only when they enable clearly better queries
3. Keep **descriptive prose** in the note body
4. Treat larger schemas as an exception for high-value operational note types, not the default

This approach keeps the vault queryable without sacrificing readability, flexibility, or thoughtful linking.

## Required By Default Policy

The Templater workflow treats a property as **required by default** when it materially improves one or more of the following:
- first-pass classification
- high-value Dataview retrieval
- separation from nearby content types

The workflow treats a property as **optional by default** when it is primarily:
- enrichment
- descriptive context
- operational nuance
- something likely to be filled in after initial capture

In short:
- **Required** = needed to answer "what kind of note is this?"
- **Optional** = useful detail that improves the note after creation

## Creation Workflow Flags

The Templater schema now separates three different concerns:

- `required`
  - The property is part of a semantically complete note model.
- `promptOnCreate`
  - The property should be prompted during note creation.
- `allowEmptyOnCreate`
  - The property may still be left as an empty list or placeholder during creation if no suitable value exists yet.

This separation is intentional. A property can now be:
- required and prompted
- optional but still prompted for consistency
- prompted but allowed to remain empty during initial capture

In practice:
- most controlled-vocabulary properties use `promptOnCreate: true`
- typed `list[link]` properties often use `promptOnCreate: true` and `allowEmptyOnCreate: true`
- `required` no longer acts as a hidden proxy for prompt behavior

## Publication Status

Content notes may also carry a `note-status` property to distinguish working drafts from notes that are ready to appear in the main knowledge views.

- `✍️ Draft`
  - The note is still in progress and should be hidden from most Dataview discovery queries.
- `☑️ Ready`
  - The note is ready to appear in Dataview-driven note discovery.

This vault now favors **status-only filtering** over automated file promotion. In practice, Dataview queries should:
- include notes with `note-status: ☑️ Ready`
- include older notes that do not yet have a `note-status`
- exclude notes explicitly marked `✍️ Draft`

This keeps publication logic simple and avoids coupling note visibility to a promotion script or directory move.
New content notes are not given a separate draft disclaimer block automatically; `note-status` is the source of truth for note visibility.

When the generator adds `note-status` automatically, it currently appends it as the last metadata property if the template did not already define it. If a template explicitly includes `note-status`, the workflow updates that existing line in place rather than moving it.

## Failure Recovery

The Templater workflow now auto-generates `note-status` for content notes so visibility can be controlled consistently during note creation.

If note creation is cancelled or fails after the workflow has already started, the generator now leaves behind a **safe recovery draft** instead of malformed frontmatter. In practice this means:
- content notes fall back to a minimal draft with `note-status`
- the note explains that the workflow was cancelled or failed
- users can repair the note manually instead of losing the work entirely

The creation workflow also surfaces clearer notices when:
- a prompted property is filled successfully
- a prompted link-list property is left as a template placeholder
- a fallback value is used during validation
- the final destination of the new note is determined

`schema-version` is still a reasonable future addition, but it is being deferred until the vault adopts a more formal external schema or migration mechanism.

## Property Input Modes

The property tables below use the following input-mode labels:
- `Controlled value`: choose one value from a predefined list
- `Controlled list`: choose one or more values from a predefined list
- `Note picker`: choose one or more existing notes from a filtered set of content types
- `Free-write`: manually enter any value
- `Free-write list`: manually enter one or more values
- `Boolean`: `true`/`false`
- `Date`: date entry
- `Number`: numeric entry

As a workflow rule, properties that use `Controlled value` or `Controlled list` input modes usually set `promptOnCreate: true`. Even when they are not semantically required for long-term note quality, the creation workflow still prompts for them so classification stays consistent.

## Maintaining Controlled Values

Controlled values are now centralized in [[0400 - Gen_Note.md]] under the `CONTROLLED_VALUES` object.

This means most schema maintenance should follow this pattern:
1. Update the reusable vocabulary list in `CONTROLLED_VALUES`
2. Leave the property schema entry alone unless the property should point to a different vocabulary entirely

Examples of reusable controlled vocabularies include:
- Platform lists
- Severity levels
- ATT&CK tactics
- OPSEC risk levels
- Protocol families
- Resource types

This refactor separates **configuration** from **prompting logic**. Users making routine taxonomy changes should usually only need to edit `CONTROLLED_VALUES`, not the property prompt workflow itself.

> [!todo]
> A medium-term goal for this project is to move schema/config details out of the script into a dedicated config file or note. This will likely be pursued concurrently with the installation script goals.

## Content Type Property Matrix

### Attack Surface

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `platforms` | `list[text]` | Controlled list | Yes | Core classifier for discovery and peer queries. |
| `deployment-models` | `list[text]` | Controlled list | No | Useful context, but not needed for initial placement. |
| `authentication-methods` | `list[text]` | Controlled list | No | Enrichment for later analysis and filtering. |
| `related-controls` | `list[link]` | Note picker | Yes | Required so attack surface notes immediately connect to protective context. |
| `related-tradecraft` | `list[link]` | Note picker | Yes | Required so attack surface notes immediately connect to adversary behavior. |
| `related-vulnerabilities` | `list[link]` | Note picker | Yes | Required so attack surface notes immediately connect to exploitation paths. |
| `related-tools` | `list[link]` | Note picker | Yes | Required so attack surface notes immediately connect to supporting tooling. |

### Basic

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| None beyond universal content properties | n/a | n/a | n/a | Intended to stay minimal and prose-first. |

### Biography

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `organizations` | `list[text\|link]` | Free-write list | No | Helpful context, but not essential for initial classification. |
| `roles` | `list[text]` | Free-write list | No | Supporting detail for later curation. |
| `active-from` | `date` | Date | No | Timeline enrichment. |
| `active-to` | `date` | Date | No | Timeline enrichment. |
| `related-tradecraft` | `list[link]` | Note picker | Yes | Required so biography notes anchor to relevant operational content immediately. |
| `related-tools` | `list[link]` | Note picker | Yes | Required so biography notes anchor to relevant operational content immediately. |

### Case Study

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `engagement-type` | `text` | Controlled value | Yes | Primary classifier for case study retrieval. |
| `target-environment` | `text` | Free-write | Yes | Distinguishes scope and operational context early. |
| `used-tradecraft` | `list[link]` | Note picker | No | Valuable, but often curated after capture. |
| `used-tools` | `list[link]` | Note picker | No | Valuable, but often curated after capture. |
| `exploited-vulnerabilities` | `list[link]` | Note picker | No | Valuable, but often curated after capture. |
| `start-date` | `date` | Date | No | Timeline enrichment. |
| `end-date` | `date` | Date | No | Timeline enrichment. |

### Command

| Property                 | Type         | Input Mode      | Required | Justification                                                                |
| ------------------------ | ------------ | --------------- | -------- | ---------------------------------------------------------------------------- |
| `execution-environments` | `list[text]` | Controlled list | Yes      | Commands need an execution-environment classifier describing where they run. |
| `target-environments`    | `list[text]` | Controlled list | No       | Separates what the command operates against from where it executes.          |
| `shell-environments`     | `list[text]` | Controlled list | No       | Useful execution detail, but not a minimum classifier.                       |
| `permissions-required`   | `list[text]` | Controlled list | No       | Important nuance, but not required for initial indexing.                     |
| `used-in-tradecraft`     | `list[link]` | Note picker     | No       | Relationship enrichment.                                                     |
| `used-by-tools`          | `list[link]` | Note picker     | No       | Relationship enrichment.                                                     |
| `supports-remote`        | `boolean`    | Boolean         | No       | Helpful nuance, but not required to create a usable note.                    |

### Idea

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `status` | `text` | Controlled value | No | Helpful workflow hint, not core classification. |
| `related-tradecraft` | `list[link]` | Note picker | Yes | Required so idea notes connect to actionable tradecraft from the start. |
| `related-tools` | `list[link]` | Note picker | Yes | Required so idea notes connect to actionable tooling from the start. |
| `related-playbooks` | `list[link]` | Note picker | Yes | Required so idea notes connect to executable workflows from the start. |

### Infrastructure

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `infrastructure-type` | `text` | Controlled value | Yes | Primary classifier for this note type. |
| `platforms` | `list[text]` | Controlled list | No | Useful, but not every infrastructure note needs it up front. |
| `components` | `list[text\|link]` | Free-write list | No | Better captured after initial note creation. |
| `supports-playbooks` | `list[link]` | Note picker | No | Relationship enrichment. |
| `supports-tools` | `list[link]` | Note picker | No | Relationship enrichment. |
| `supports-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |

### IOC

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `ioc-type` | `text` | Controlled value | Yes | Fundamental classifier for detection content. |
| `indicator-value` | `text` | Free-write | Yes | The note is not meaningful without the actual indicator. |
| `confidence` | `text` | Controlled value | No | Useful assessment metadata, but not required to capture the IOC. |
| `associated-tools` | `list[link]` | Note picker | No | Relationship enrichment. |
| `associated-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `active` | `boolean` | Boolean | No | Useful state flag, but not mandatory for first capture. |

### Lab Setup

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `lab-purpose` | `text` | Controlled value | Yes | Primary classifier for the lab's role. |
| `platforms` | `list[text]` | Controlled list | No | Helpful context, but not always necessary initially. |
| `difficulty` | `text` | Controlled value | No | Learning/support metadata rather than core indexing. |
| `estimated-build-time` | `text` | Free-write | No | Workflow aid, not classification. |
| `practices-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `uses-tools` | `list[link]` | Note picker | No | Relationship enrichment. |

### Offensive Code

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `languages` | `list[text]` | Controlled list | Yes | Strong first-pass classifier for code notes. |
| `entry-points` | `list[text]` | Controlled list | No | Useful technical detail, but not a required first classifier. |
| `implements-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `targets-vulnerabilities` | `list[link]` | Note picker | No | Relationship enrichment. |
| `uses-protocols` | `list[link]` | Note picker | No | Relationship enrichment. |
| `opsec-risk` | `text` | Controlled value | No | Operational nuance best added later. |
| `platforms` | `list[text]` | Controlled list | No | Helpful context, but not strictly required if language is known. |

### Playbook

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `objective` | `text` | Free-write | Yes | Defines what the playbook is for. |
| `scope` | `text` | Controlled value | Yes | Core classifier for operational context. |
| `required-tools` | `list[link]` | Note picker | No | Often curated after the playbook skeleton exists. |
| `required-tradecraft` | `list[link]` | Note picker | No | Often curated after the playbook skeleton exists. |
| `required-access` | `list[text]` | Free-write list | No | Helpful prerequisite detail, not required for initial creation. |
| `opsec-risk` | `text` | Controlled value | No | Operational nuance. |
| `tested` | `boolean` | Boolean | No | Lifecycle/workflow metadata. |

### Protocol

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `protocol-family` | `text` | Controlled value | Yes | Primary classifier for protocol discovery. |
| `ports` | `list[text]` | Free-write list | No | Useful technical detail, but not always required immediately. |
| `authentication-methods` | `list[text]` | Controlled list | No | Valuable nuance, but not essential for first-pass capture. |
| `abused-by-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `secured-by-controls` | `list[link]` | Note picker | No | Relationship enrichment. |
| `related-tools` | `list[link]` | Note picker | Yes | Required so protocol notes connect to concrete tool usage immediately. |

### Reference Material

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `resource-type` | `text` | Controlled value | Yes | Defines the note's basic nature. |
| `authors` | `list[text]` | Free-write list | No | Useful bibliographic enrichment. |
| `publisher` | `text` | Free-write | No | Useful bibliographic enrichment. |
| `publication-date` | `date` | Date | No | Useful bibliographic enrichment. |
| `covers-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `covers-tools` | `list[link]` | Note picker | No | Relationship enrichment. |
| `covers-platforms` | `list[text]` | Free-write list | No | Helpful discovery detail, but not essential. |

### Security Control

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `control-category` | `text` | Controlled value | Yes | Primary classifier for control discovery. |
| `protects-platforms` | `list[text]` | Controlled list | No | Useful scope detail, but not always needed at creation time. |
| `detects-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `known-bypasses` | `list[link]` | Note picker | No | Relationship enrichment. |
| `telemetry-sources` | `list[text]` | Free-write list | No | Valuable operational detail, but not a minimum classifier. |
| `enforcement-points` | `list[text]` | Free-write list | No | Valuable operational detail, but not a minimum classifier. |

### Study Resources

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `resource-type` | `text` | Controlled value | Yes | Defines the note's basic nature. |
| `provider` | `text` | Free-write | No | Useful source context. |
| `status` | `text` | Controlled value | No | Workflow/lifecycle metadata. |
| `covers-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `covers-tools` | `list[link]` | Note picker | No | Relationship enrichment. |
| `covers-platforms` | `list[text]` | Free-write list | No | Helpful discovery detail, but not essential. |
| `completed-on` | `date` | Date | No | Lifecycle metadata. |

### Tool

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `tool-category` | `text` | Free-write | Yes | Primary classifier for tool discovery. |
| `operating-platforms` | `list[text]` | Controlled list | No | Valuable context, but not always known immediately. |
| `target-platforms` | `list[text]` | Controlled list | No | Valuable context, but not always known immediately. |
| `implements-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `used-in-playbooks` | `list[link]` | Note picker | No | Relationship enrichment. |
| `opsec-risk` | `text` | Controlled value | No | Operational nuance. |
| `detection-difficulty` | `text` | Controlled value | No | Operational nuance. |
| `tested` | `boolean` | Boolean | No | Lifecycle/workflow metadata. |

### Tradecraft

| Property                   | Type         | Input Mode       | Required | Justification                                             |
| -------------------------- | ------------ | ---------------- | -------- | --------------------------------------------------------- |
| `attack-id`                | `list[text]` | Free-write       | No       | Helpful external mapping(s), but not every note needs it. |
| `tactic`                   | `text`       | Controlled value | Yes      | Core classifier and major Dataview dimension.             |
| `platforms`                | `list[text]` | Controlled list  | Yes      | Core classifier and major Dataview dimension.             |
| `permissions-required`     | `list[text]` | Controlled list  | No       | Important nuance, but not required for first capture.     |
| `supports-remote`          | `boolean`    | Boolean          | No       | Helpful nuance, not a minimum classifier.                 |
| `data-sources`             | `list[text]` | Free-write list  | No       | Important defensive context, but can be filled later.     |
| `uses-tools`               | `list[link]` | Note picker      | No       | Relationship enrichment.                                  |
| `bypasses-controls`        | `list[link]` | Note picker      | No       | Relationship enrichment.                                  |
| `exploits-vulnerabilities` | `list[link]` | Note picker      | No       | Relationship enrichment.                                  |
| `uses-protocols`           | `list[link]` | Note picker      | No       | Relationship enrichment.                                  |

### Vulnerability

| Property | Type | Input Mode | Required | Justification |
| --- | --- | --- | --- | --- |
| `cve-id` | `text` | Free-write | No | Useful identifier, but not every vulnerability note is CVE-backed. |
| `cvss-score` | `number` | Number | No | Helpful severity detail, but can be added later. |
| `severity` | `text` | Controlled value | Yes | Primary classifier for vulnerability retrieval. |
| `affected-platforms` | `list[text]` | Controlled list | No | Helpful scoping context, but not strictly required for first capture. |
| `affects-attack-surfaces` | `list[link]` | Note picker | No | Relationship enrichment. |
| `exploited-by-tradecraft` | `list[link]` | Note picker | No | Relationship enrichment. |
| `prerequisites` | `list[text]` | Free-write list | No | Useful exploit context, but not required initially. |

---
## Resources

| Reference | Info |
| --------- | ---- |
| [Properties, Obsidian Help](https://help.obsidian.md/properties)                   | Official Obsidian documentation for frontmatter properties |
| [Dataview Wiki, Michael Brenan](https://blacksmithgu.github.io/obsidian-dataview/) | Official Dataview documentation and query reference        |

---
*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
