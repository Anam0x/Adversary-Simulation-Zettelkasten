---
aliases:
tags:
  - 💲Command
primary-categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
  - "[[Development]]"
secondary-categories:
  - "[[Active Directory]]"
  - "[[PowerShell]]"
  - "[[Domain Enumeration]]"
  - "[[Post-Exploitation]]"
type: Command
execution-environments:
  - Windows
target-environments:
  - Active Directory
shell-environments:
  - PowerShell
permissions-required:
  - User
used-in-tradecraft:
  - "[[Kerberoasting]]"
used-by-tools:
  - <!-- [[Tool Note]] -->
supports-remote: true
note-status: ☑️ Ready
---
# [[PowerShell - FormatEnumerationLimit]]

---

## Overview

Sets the PowerShell `$FormatEnumerationLimit` variable to control how many items are displayed when outputting collections. Essential for PowerView enumeration to prevent output truncation of large results like group memberships or computer lists.

## Requirements/Assumptions

* [ ] PowerShell execution context
* [ ] Sufficient privileges to modify session variables

## Syntax

```powershell
# Display current limit
$FormatEnumerationLimit

# Set unlimited output (recommended for enumeration)
$FormatEnumerationLimit = -1

# Set specific limit
$FormatEnumerationLimit = 50

# Reset to default (4 items)
$FormatEnumerationLimit = 4
```

## Examples/Use Cases

Basic usage before PowerView enumeration:
```powershell
# Set unlimited output to see all results
$FormatEnumerationLimit = -1

# Now enumerate domain users without truncation
Get-DomainUser | Select Name, MemberOf
```

Mythic C2 integration:
```powershell
# Submit as single command in Mythic Apollo
powerpick $FormatEnumerationLimit = -1; Get-DomainGroupMember -Identity "Domain Admins"
```

## Notes

### Common Flags/Variants

- `$FormatEnumerationLimit = -1`
	- Use this when you want collection output to remain untruncated during enumeration-heavy commands such as PowerView or custom AD reporting
- `$FormatEnumerationLimit = <positive integer>`
	- Useful when you want to preserve readability while still showing more than the default four items in a property collection
- `$FormatEnumerationLimit = 4`
	- Resets the session to PowerShell's default display behavior after finishing broad enumeration work
- Inline use with another command
	- Common in C2 or remote execution contexts where you want to set the variable and run the enumeration in one line; e.g., `powerpick $FormatEnumerationLimit = -1; Get-DomainUser | Select Name, MemberOf`

### Failure Cases

- Forgetting to set the value before large enumeration can still leave important output truncated
- Adjusting the value affects display only; it does not improve access or query quality by itself

---

## Related Notes

### Same Classification

#### Commands On Similar Platforms
```dataview
LIST
FROM "03 - Content"
WHERE type = "Command"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND contains(execution-environments, this.execution-environments[0])
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Used In Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.used-in-tradecraft, file.link)
SORT file.name ASC
```

#### Used By Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.used-by-tools, file.link)
SORT file.name ASC
```

---

## Resources

| Reference                                                                                                                               | Info                                                                 |
| --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| [How to Use $FormatEnumerationLimit, DoctorDNS](https://devblogs.microsoft.com/powershell-community/how-to-use-formatenumerationlimit/) | Blog post detailing correct `$FormatEnumerationLimit` variable usage |

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
