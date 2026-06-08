---
aliases:
  - SMB Signing Not Required
tags:
  - 🕳️Vulnerability
primary-categories:
  - "[[Network Security]]"
  - "[[Penetration Test]]"
secondary-categories:
  - "[[Active Directory]]"
  - "[[Post-Exploitation]]"
type: Vulnerability
cve-id: N/A
cvss-score: 6.5
severity: Medium
affected-platforms:
  - Windows
  - Network
affects-attack-surfaces:
  - "[[Microsoft SQL Server]]"
exploited-by-tradecraft:
  - <!-- [[Tradecraft Note]] -->
prerequisites:
  - Attacker can reach the SMB service
  - SMB signing is not required on the target
note-status: ☑️ Ready
---
# [[SMB Signing Disabled]]

---

## Overview

SMB signing disabled is a misconfiguration rather than a software defect, but it creates a meaningful trust-boundary weakness by allowing tampering or relay opportunities in environments that still rely heavily on SMB-backed authentication and administrative workflows[^1]. It is especially relevant on Windows servers that participate in domain trust and expose administrative shares or service-management paths.

## Technical Summary

| Dimension | Notes |
| --------- | ----- |
| Affected Component | SMB service configuration on Windows hosts, appliances, or other SMB-speaking systems |
| Root Cause | Failure to require SMB message signing where trust in server/client authenticity matters |
| Impact | Relay opportunity, session tampering risk, and weaker integrity guarantees for SMB-authenticated workflows |

## Exploit Conditions

| Condition | Details |
| --------- | ------- |
| Preconditions | The attacker can reach the target over SMB and signing is disabled or not enforced |
| Required Access | Network position sufficient to coerce, capture, or relay SMB-backed authentication |
| Reachable Interfaces | SMB listener plus any downstream systems that can be reached through the relayed context |

## Exploitation Notes

### High-Level Abuse Path

1. Coerce or observe an SMB authentication attempt.
2. Relay or tamper with the authentication flow where signing enforcement is absent.
3. Use the resulting access to pivot into lateral movement, administrative action, or further credential abuse.

### Operational Constraints

| Constraint | Why It Matters |
| ---------- | -------------- |
| Broad signing enforcement | Can shut down the most practical relay paths entirely |
| Target quality | Relay value depends on which services and privileges the relayed identity can actually reach |
| Environmental chaining | The issue is most valuable when paired with other trust or authentication weaknesses |

### Example Commands/Requests

```powershell
Get-SmbServerConfiguration | Select RequireSecuritySignature, EnableSecuritySignature
```

```bash
nxc smb <target> --gen-relay-list relayable.txt
```

## Detection And Investigation

### Indicators

| Source | Indicator | Notes |
| ------ | --------- | ----- |
| Configuration review | Signing not required on server or client side | Fastest way to confirm exposure without active abuse |
| SMB/NTLM telemetry | Unusual authentication paths between hosts that do not normally interact | Helpful when relay or coercion is suspected |
| Process/network context | Relay-oriented tooling or coercion before unexpected SMB actions | Stronger when paired with timing and identity context |

### Verification

- [ ] Review effective SMB server and client signing configuration.
- [ ] Validate whether the target can be relayed to in the actual environment, not just in theory.

## Mitigation

### Remediation

| Option | Action | Notes |
| ------ | ------ | ----- |
| Configuration | Require SMB signing on relevant servers and clients | Highest-value direct fix for this misconfiguration |
| Exposure reduction | Reduce unnecessary SMB exposure and administrative-share reachability | Limits the number of viable relay targets |
| Defense-in-depth | Pair signing enforcement with broader relay-hardening measures | Important when legacy systems cannot be remediated immediately |

### Residual Risk

- Signing enforcement reduces a key abuse path, but other trust and authentication weaknesses may still enable lateral movement

---

## Related Notes

### Same Classification

#### Similar Vulnerabilities
```dataview
LIST
FROM "03 - Content"
WHERE type = "Vulnerability"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    severity = this.severity OR
    contains(affected-platforms, this.affected-platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Affects Attack Surfaces
```dataview
LIST
FROM "03 - Content"
WHERE type = "Attack Surface"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.affects-attack-surfaces, file.link)
SORT file.name ASC
```

#### Exploited By Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.exploited-by-tradecraft, file.link)
SORT file.name ASC
```

---

## Resources

| Reference                                                                                                                                                                | Info                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------- |
| [Control SMB signing behavior, Microsoft](https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-signing)                                              | Primary note-level source for the misconfiguration and remediation posture |
| [SMB Relay Attacks and How to Defend Against Them, Microsoft](https://techcommunity.microsoft.com/blog/filecab/smb-relay-attacks-and-how-to-defend-against-them/4235322) | Secondary note-level source for exploitation and mitigation context        |

[^1]: Control SMB signing behavior, Microsoft, https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-signing

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
