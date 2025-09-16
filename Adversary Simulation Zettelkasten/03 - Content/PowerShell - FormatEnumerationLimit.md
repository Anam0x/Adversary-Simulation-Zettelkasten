---
aliases: 
tags:
  - 💲Command
primary categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
  - "[[Development]]"
secondary categories:
  - "[[Active Directory]]"
  - "[[PowerShell]]"
  - "[[Domain Enumeration]]"
  - "[[Post-Exploitation]]"
type: Command
---
# [[PowerShell - FormatEnumerationLimit]]

---

## Overview

Sets the PowerShell `$FormatEnumerationLimit` variable to control how many items are displayed when outputting collections. Essential for PowerView enumeration to prevent output truncation of large results like group memberships or computer lists.

## Requirements

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

## Examples & Use Cases

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

---

## Resources

| Hyperlink                                                                                                                               | Info                                                                 |
| --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| [How to Use $FormatEnumerationLimit, DoctorDNS](https://devblogs.microsoft.com/powershell-community/how-to-use-formatenumerationlimit/) | Blog post detailing correct `$FormatEnumerationLimit` variable usage |

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
