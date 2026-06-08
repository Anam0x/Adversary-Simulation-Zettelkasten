---
aliases:
tags:
  - 📚Reference_Material
primary-categories:
  - "[[Training]]"
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary-categories:
  - "[[Active Directory]]"
type: Reference Material
resource-type: Documentation
authors:
  - Orange CyberDefense
publisher: Orange CyberDefense
publication-date: <!-- YYYY-MM-DD -->
covers-tradecraft:
  - "[[03 - Content/Kerberoasting]]"
covers-tools:
  - "[[GetUserSPNs.py]]"
covers-platforms:
  - Active Directory
note-status: ☑️ Ready
---
# [[AD - mindmap 2025 - 03]]

---
## Overview

*AD - mindmap 2025 - 03*[^1] is a comprehensive visual reference maintained by [Orange CyberDefense](https://github.com/Orange-Cyberdefense) that outlines key techniques and tools used in Active Directory (AD) enumeration. It organizes enumeration strategies into logical categories such as domain reconnaissance, user and group enumeration, trust relationships, and privilege escalation paths. This mindmap serves as a practical guide for red team operators and penetration testers to navigate the complexities of AD environments, supporting methodical exploration and identification of attack opportunities within Windows domains. To best navigate the mindmap, access the image from the [ocd-mindmaps](https://orange-cyberdefense.github.io/ocd-mindmaps/) landing page in a web browser.

## Summary

This reference is most useful as a visual checklist for broad Active Directory reconnaissance. It helps translate a large enumeration space into a structured set of categories, tools, and follow-on ideas.

## Embedded Reference/Notes

![](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg)

## Key Takeaways

- AD enumeration benefits from a repeatable visual framework
- A strong reference note can bridge tools, tradecraft, and attack-path thinking
- This mindmap is best used alongside more detailed procedural notes and playbooks

## When To Use This

- Early in an AD engagement to orient enumeration scope
- When building or reviewing an enumeration checklist
- When mapping related notes for domain reconnaissance and privilege escalation

---

## Related Notes

### Same Classification

#### Similar Reference Material
```dataview
LIST
FROM "03 - Content"
WHERE type = "Reference Material"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    resource-type = this.resource-type OR
    contains(covers-platforms, this.covers-platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Covers Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.covers-tradecraft, file.link)
SORT file.name ASC
```

#### Covers Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.covers-tools, file.link)
SORT file.name ASC
```

---

## Resources

| Reference | Info |
| --------- | ---- |
| [OCD Mindmaps, Orange Cyberdefense](https://orange-cyberdefense.github.io/ocd-mindmaps/) | Orange Cyberdefense mindmaps |

[^1]: AD - mindmap 2025 - 03, Orange CyberDefense, https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
