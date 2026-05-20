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
# [[Emoji Toolbar]]

***

## Description

The Emoji Toolbar plugin for Obsidian adds a customizable hotkey that opens a searchable emoji picker, letting you quickly filter, preview (with Twitter-style formatting), and insert emojis, including skin tone variants and recently used icons, directly into your editor.

## Installation

1. Emoji Toolbar is a registered Obsidian plugin and can be installed directly from `Settings > Community Plugins > Browse`
	* [[Obsidian - Plugins#Required Plugins]]

## Configuration

* Ensure that the plugin is enabled by navigating to `Settings > Community Plugins > Emoji Toolbar` and toggling the single option to on
* After enabling the plugin, navigate to `Settings > Hotkeys > Emoji Toolbar: Open emoji picker` to confirm/modify the hotkey associate with launching the emoji keyboard; the default value is <kbd>Win</kbd> + <kbd>Alt</kbd> + <kbd>.</kbd>

## Basic Usage

Use the hotkey specified at `Settings > Hotkeys > Emoji Toolbar: Open emoji picker` to launch the emoji keyboard.

![](https://raw.githubusercontent.com/oliveryh/obsidian-emoji-toolbar/main/demo/demo.gif)

The following emojis are currently used as special search tags for notes:

```dataviewjs
const RESERVED_CATEGORY_TAGS = [
  { emoji: "🥇", label: "Primary Category" },
  { emoji: "🥈", label: "Secondary Category" },
];

const extractEmojiTag = (page, excludedTags = []) => {
  const tags = (page.tags ?? []).filter(tag => !excludedTags.includes(tag));
  return tags.length > 0 ? tags[0] : null;
};

const extractLeadingEmoji = (tagValue) => {
  if (!tagValue) {
    return "?";
  }

  const normalized = String(tagValue).replace(/^#/, "");
  const emojiMatch = normalized.match(/^[^A-Za-z0-9_]+/u);
  return emojiMatch ? emojiMatch[0] : normalized;
};

const renderBulletList = (title, items) => {
  const lines = [
    `- ${title}:`,
    ...items.map(item => `  - ${item.emoji} - ${item.label}`),
  ];
  dv.paragraph(lines.join("\n"));
};

const primaryCategoryItems = dv.pages('"01 - Primary Categories"')
  .sort(p => p.file.name, "asc")
  .map(page => ({
    emoji: extractLeadingEmoji(extractEmojiTag(page, ["🥇Primary_Category"])),
    label: page.file.name,
  }))
  .array();

const contentTypeItems = Array.from(
  new Map(
    dv.pages('"04 - Templates/Content"')
      .where(page => page.type)
      .sort(page => page.type, "asc")
      .map(page => {
        const tag = extractEmojiTag(page);
        return [
          page.type,
          {
            emoji: extractLeadingEmoji(tag),
            label: page.type,
          }
        ];
      })
      .array()
  ).values()
).sort((a, b) => a.label.localeCompare(b.label));

renderBulletList("Categories", RESERVED_CATEGORY_TAGS);
renderBulletList("Primary Categories", primaryCategoryItems);
renderBulletList("Content", contentTypeItems);
```

> [!info]
> Currently, there is no restriction on using the same emoji as a search tag for new primary categories or content types (e.g., you can have a primary category with the search tag "💯New_Category" and a content type with the search tag "💯New_Content_Type"). This is a deliberate design choice to account for scenarios where the list of compatible emojis has been exhausted (an unlikely scenario given Emoji Toolbar theoretically supports [at least 3,790 emojis as of September 2024](https://emojipedia.org/faq#how-many)) and where users create loosely related content types (e.g., two content types with the search tags "⛏️Offensive_Tool" and "⛏️Defensive_Tool").

The following emojis are always reserved for vault administration and cannot be used to create new search tags via the [[Templater]] plugin script:

* 🥇 - Primary Category
* 🥈 - Secondary Category
* ⚛️ - Atomic Note/Content

Although the note creation workflow script automatically prohibits this behavior, users must avoid manually introducing search tags using these emojis. Failure to do so may result in unexpected behavior from the Obsidian search tag feature.

***

## Resources

| Hyperlink                                                                                                                       | Info                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| [Obsidian, Taming a Collective Consciousness; Sam Link](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness) | Example implementation of Zettelkasten using Obsidian; discusses the importance of "special tags" |
| [Dataview, GitHub](https://github.com/blacksmithgu/obsidian-dataview)                                                           | Main repository for the Dataview community plugin for Obsidian                                    |

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
