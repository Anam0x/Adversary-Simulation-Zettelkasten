---
aliases:
  - AD Initial Enumeration Cheat Sheet
  - AD Initial Enumeration Checklist
tags:
  - ✅Playbook
primary categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary categories:
  - "[[Active Directory]]"
  - "[[Post-Exploitation]]"
  - "[[Domain Enumeration]]"
type: Playbook
---
# [[AD Initial Enumeration Playbook]]

***

## Overview

This playbook provides a structured approach for initial Active Directory (AD) enumeration after gaining access to a domain-joined host. It focuses on gathering essential domain information to establish situational awareness and identify potential attack paths.
## Steps

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

***

## Resources

| Hyperlink                                                                                                              | Info                                                         |
| ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| [PowerView Cheat Sheet, HarmJ0y](https://github.com/HarmJ0y/CheatSheets/blob/master/PowerView.pdf)                     | PowerView 3.0 cheat sheet by PowerSploit co-creator          |
| [PowerView CheatSheet, Zach Fleming](https://zflemingg1.gitbook.io/undergrad-tutorials/powerview/powerview-cheatsheet) | Zach Fleming's (AKA _zflemingg1_) AD enumeration cheat sheet |
| [PowerView-3.0 tips and tricks, HarmJ0y](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)             | PowerView 3.0 sample commands by PowerSploit co-creator      |
| [PowerView, PowerSploit](https://powersploit.readthedocs.io/en/latest/Recon/)                                          | PowerView documentation                                      |

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
