---
aliases:
  - AD Initial Enumeration Cheat Sheet
  - AD Initial Enumeration Checklist
tags:
  - ✅Playbook
primary-categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary-categories:
  - "[[Active Directory]]"
  - "[[Post-Exploitation]]"
  - "[[Domain Enumeration]]"
type: Playbook
objective: Establish initial Active Directory situational awareness and identify follow-on attack paths
scope: Internal
required-tools:
  - "[[PowerShell - FormatEnumerationLimit]]"
required-tradecraft:
  - <!-- [[Tradecraft Note]] -->
required-access: Domain-joined user context
opsec-risk: Medium
tested: true
note-status: ☑️ Ready
---
# [[AD Initial Enumeration Playbook]]

---
## Overview

This playbook provides a structured approach for initial Active Directory (AD) enumeration after gaining access to a domain-joined host. It focuses on gathering essential domain information to establish situational awareness and identify potential attack paths.

## Objectives

- Establish enough domain context to identify high-value accounts, trust relationships, and quick-win attack paths for follow-on tradecraft
- Capture enumeration output in a documentable format to facilitate easier report writing and reviewing of operational notes

## Prerequisites

- Authenticated access to a domain-joined Windows host
- A PowerShell execution context that allows basic enumeration commands
- At least one operator-controlled session where output can be reviewed without truncation

## Procedure

* [ ] Establish PowerView session
	* [ ] Load PowerView module
	* [ ] Set format enumeration limit: 
		* [[PowerShell - FormatEnumerationLimit]]
	* [ ] Test basic connectivity with `Get-Domain`
* [ ] Enumerate current domain context
	* [ ] Identify current domain and forest
	* [ ] Document domain functional level
	* [ ] Map domain controllers and their roles
* [ ] Identify high-value targets
	* [ ] Enumerate Domain Admins group membership
	* [ ] Check for users with AdminCount=1
	* [ ] Locate computer objects configured for delegation
 * [ ] Map trust relationships
	 * [ ] Document inbound and outbound domain trusts
	 * [ ] Identify forest trusts and their directions
	 * [ ] Note any trust attributes (SID filtering, etc.)
 * [ ] Check for quick wins
	 * [ ] Search for users with SPNs (kerberoasting candidates)
	 * [ ] Find users with DONT_REQ_PREAUTH (ASREP roasting candidates)
	 * [ ] Look for computer accounts with unconstrained delegation
 * [ ] Document findings
	 * [ ] Create target list prioritized by risk/impact
	 * [ ] Note any misconfigurations or weaknesses
	 * [ ] Plan next enumeration phase based on discoveries

## Validation

- Confirm the current domain, forest, and domain controllers are documented
- Confirm at least one prioritized target list or attack path hypothesis exists
- Confirm trust relationships and quick-win opportunities were recorded for follow-on action

## Cleanup/Rollback

- Remove any imported tooling if the operation requires minimizing in-memory artifacts
- Sanitize operator notes before sharing them outside the engagement context

---

## Related Notes

### Same Classification

#### Similar Playbooks
```dataview
LIST
FROM "03 - Content"
WHERE type = "Playbook"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    objective = this.objective OR
    scope = this.scope
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Required Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.required-tradecraft, file.link)
SORT file.name ASC
```

#### Required Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.required-tools, file.link)
SORT file.name ASC
```

---

## Resources

| Reference | Info |
| --------- | ---- |
| [PowerView Cheat Sheet, HarmJ0y](https://github.com/HarmJ0y/CheatSheets/blob/master/PowerView.pdf)                     | PowerView 3.0 cheat sheet by PowerSploit co-creator          |
| [PowerView CheatSheet, Zach Fleming](https://zflemingg1.gitbook.io/undergrad-tutorials/powerview/powerview-cheatsheet) | Zach Fleming's (AKA _zflemingg1_) AD enumeration cheat sheet |
| [PowerView-3.0 tips and tricks, HarmJ0y](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)             | PowerView 3.0 sample commands by PowerSploit co-creator      |
| [PowerView, PowerSploit](https://powersploit.readthedocs.io/en/latest/Recon/)                                          | PowerView documentation                                      |

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
