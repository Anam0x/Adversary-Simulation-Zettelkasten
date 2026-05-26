---
aliases:
  - T1558.003
  - Kerberoasting
  - Steal or Forge Kerberos Tickets - Kerberoasting (T1558.003)
tags:
  - 📕Tradecraft
primary-categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary-categories:
  - "[[Active Directory]]"
  - "[[Credential Harvesting]]"
  - "[[Post-Exploitation]]"
type: Tradecraft
attack-id: 
  - T1558.003
tactic: Credential Access
platforms:
  - Windows
permissions-required:
  - Authenticated User
supports-remote: true
data-sources:
  - Active Directory Credential Request
  - Windows Event Logs
uses-tools:
  - "[[GetUserSPNs.py]]"
bypasses-controls:
  - <!-- [[Security Control Note]] -->
exploits-vulnerabilities:
  - <!-- [[Vulnerability Note]] -->
uses-protocols:
  - <!-- [[Protocol Note]] -->
note-status: ☑️ Ready
---
# [[Kerberoasting]]

---

## Overview

Kerberoasting is credential-access tradecraft that targets service accounts backed by Service Principal Names (SPNs). An operator with a valid domain user context can request Kerberos service tickets for those accounts, extract the crackable material, and attempt offline recovery of the underlying password[^1][^2].

The technique is most useful when the target environment still permits favorable encryption types or relies on weak, manually managed service-account passwords. A recovered service credential can quickly turn into lateral movement, privilege escalation, or persistent application access.

## Objective

Obtain crackable Kerberos service-ticket material from SPN-backed accounts in order to recover reusable credentials for follow-on access, escalation, or lateral movement.

## Preconditions

### Host/Identity Requirements

- A valid domain user context or equivalent ticket material capable of requesting service tickets
- At least one service account with an SPN and a password posture weak enough to justify cracking effort
- Tooling or scripts that can request, collect, or reformat TGS output for offline cracking

### Network/Environmental Requirements

- Reachability to a domain controller or another path that yields service-ticket material
- Sufficient environmental awareness to identify high-value SPNs worth targeting
- Defender monitoring, account policy, and encryption settings that do not fully neutralize the technique's payoff

## Procedure

1. Enumerate candidate SPNs and the user or service accounts they map to
2. Request the relevant TGS material from a domain controller or capture service tickets from the environment
3. Convert the resulting data into a cracking-friendly format for tools such as Hashcat or John the Ripper
4. Crack the recovered material offline and validate whether any recovered credential materially expands access

## Variants

- Request TGS tickets directly with domain credentials using tools such as `GetUserSPNs.py`, `Rubeus`, or `Invoke-Kerberoast`[^3][^4][^5]
- Recover service tickets from network traffic instead of generating the request yourself
- Narrow targeting to privileged or operationally useful service accounts instead of broad, noisy collection

## Operational Considerations

### OPSEC And Evasion

- High-volume service-ticket requests from unusual accounts are one of the easiest ways to make this technique conspicuous
- Narrow SPN targeting is generally less noisy than bulk collection across an entire domain
- The tradecraft blends better into routine activity when requests are aligned with realistic host, user, and service context

### Reliability/Failure Modes

- AES-only service accounts or strong managed passwords can make the cracking phase unproductive even when ticket collection succeeds
- Weak target selection can burn time on service accounts that do not materially improve access
- Cracking success depends on password quality, available compute, and the time window in which recovered credentials remain useful

### Success Indicators

- TGS material is collected and converted cleanly into a format suitable for offline cracking
- At least one service-account password or otherwise useful credential artifact is recovered
- The recovered credential unlocks new execution, privilege, or lateral movement options

## Detection And Response

### Host Signals

- Execution of common roasting utilities such as `GetUserSPNs.py`, `Rubeus`, or `Invoke-Kerberoast`[^3][^4][^5]
- Follow-on credential use that quickly transitions into remote service access, privilege escalation, or sensitive AD enumeration
- Operator workflows that gather large sets of SPNs and immediately stage cracking output

### Network Signals

- Unusual bursts of Kerberos TGS requests to a domain controller from a host or user that does not normally request many service tickets
- Repeated RC4-oriented service-ticket requests in environments that otherwise favor stronger encryption
- Correlation between service-ticket activity and subsequent SSH, WinRM, SMB, or other remote-authentication behavior

### Defensive Friction

- Prefer AES over RC4 where possible and retire legacy encryption paths that keep roasted material easy to crack[^2]
- Enforce strong, regularly rotated service-account credentials or use managed service account patterns to reduce cracking viability[^2]
- Minimize privileged service-account sprawl and investigate unusual Kerberos service-ticket request volumes aggressively

## References

### Procedure Examples

| Source    | Actor/Campaign        | Description                                                                                                                | Reference                                                                                                                                                                         |
| --------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Reporting | FIN7                  | Used Kerberoasting-oriented PowerShell commands to support credential access and lateral movement[^6][^7].               | [FIN7 Activity Reporting](https://www.crowdstrike.com/blog/carbon-spider-embraces-big-game-hunting-part-1/)                                                                       |
| Reporting | SolarWinds Compromise | Obtained TGS tickets for Active Directory service principals and cracked them offline during follow-on operations[^8].    | [SolarWinds Compromise Writeup](https://www.microsoft.com/security/blog/2021/01/20/deep-dive-into-the-solorigate-second-stage-activation-from-sunburst-to-teardrop-and-raindrop/) |
| Reporting | Wizard Spider         | Used Kerberoasting-capable tooling during post-exploitation to recover crackable ticket material[^9][^10][^11][^12][^13]. | [Wizard Spider/FIN12 Reporting](https://www.mandiant.com/sites/default/files/2021-10/fin12-group-profile.pdf)                                                                     |

### Tool Examples

| Tool                                                                                       | Role In Tradecraft                                                                              | Notes                                                                                                                        | Reference                                                                                          |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| [GetUserSPNs.py](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py)   | Enumerate SPN-backed accounts and request crackable tickets from a non-Windows operator context | Strong fit for Linux-based operators and easy handoff into Hashcat/John workflows[^3].                                      | [Impacket Example Script](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py)  |
| [Invoke-Kerberoast](https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/) | Request crackable service tickets from a PowerShell-capable Windows context                     | Useful when the operator already has an in-domain Windows foothold and wants to stay inside native admin workflows[^4][^14]. | [PowerSploit Documentation](https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/) |
| [Rubeus](https://github.com/GhostPack/Rubeus)                                              | Request or manipulate Kerberos ticket material from a Windows/CLR-heavy operator workflow       | Common in mature post-exploitation chains where deeper Kerberos handling is needed beyond simple ticket requests[^5].       | [GhostPack Rubeus](https://github.com/GhostPack/Rubeus)                                            |

### Framework Mappings

| Framework           | ID        | Name                                           | URL                                               |
| ------------------- | --------- | ---------------------------------------------- | ------------------------------------------------- |
| MITRE ATT&CK        | T1558.003 | Steal or Forge Kerberos Tickets: Kerberoasting | https://attack.mitre.org/techniques/T1558/003/    |
| MITRE ATT&CK Tactic | TA0006    | Credential Access                              | https://attack.mitre.org/tactics/TA0006/          |
| CAPEC               | CAPEC-509 | Kerberoasting                                  | https://capec.mitre.org/data/definitions/509.html |

---

## Related Notes

### Same Classification

#### Same Tactic
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND tactic = this.tactic
SORT file.name ASC
LIMIT 10
```

#### Same Platforms
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND contains(platforms, this.platforms[0])
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Uses Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.uses-tools, file.link)
SORT file.name ASC
```

#### Bypasses Controls
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.bypasses-controls, file.link)
SORT file.name ASC
```

#### Exploits Vulnerabilities
```dataview
LIST
FROM "03 - Content"
WHERE type = "Vulnerability"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.exploits-vulnerabilities, file.link)
SORT file.name ASC
```

#### Uses Protocols
```dataview
LIST
FROM "03 - Content"
WHERE type = "Protocol"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.uses-protocols, file.link)
SORT file.name ASC
```

### Reverse Relationships

#### Implemented By Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(implements-tradecraft, this.file.link)
SORT file.name ASC
```

#### Detected By Controls
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(detects-tradecraft, this.file.link)
SORT file.name ASC
```

---

## Resources

| Reference | Info |
| --------- | ---- |
| [Steal or Forge Kerberos Tickets: Kerberoasting](https://attack.mitre.org/techniques/T1558/003/) | MITRE ATT&CK entry for Kerberoasting |
| [Steal or Forge Kerberos Tickets, Technique T1558](https://attack.mitre.org/techniques/T1558/) | Parent ATT&CK technique for Kerberos ticket abuse |

[^1]: Invoke-Kerberoast.ps1, Empire Project, https://github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/Invoke-Kerberoast.ps1
[^2]: Cracking Kerberos TGS Tickets Using Kerberoast - Exploiting Kerberos to Compromise the Active Directory Domain, Sean Metcalf, https://adsecurity.org/?p=2293
[^3]: Impacket, SecureAuth, https://www.secureauth.com/labs/open-source-tools/impacket
[^4]: Invoke-Kerberoast, PowerSploit, https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/
[^5]: Rubeus, GhostPack, https://github.com/GhostPack/Rubeus
[^6]: Carbon Spider Embraces Big Game Hunting Part 1, CrowdStrike, https://www.crowdstrike.com/blog/carbon-spider-embraces-big-game-hunting-part-1/
[^7]: FIN7 Power Hour: Adversary Archaeology and the Evolution of FIN7, Google Cloud, https://www.mandiant.com/resources/evolution-of-fin7
[^8]: Deep Dive Into the Solorigate Second-Stage Activation, Microsoft Security, https://www.microsoft.com/security/blog/2021/01/20/deep-dive-into-the-solorigate-second-stage-activation-from-sunburst-to-teardrop-and-raindrop/
[^9]: Ryuk's Return, The DFIR Report, https://thedfirreport.com/2020/10/08/ryuks-return/
[^10]: Unhappy Hour Special: KEGTAP and SINGLEMALT With a Ransomware Chaser, Google Cloud, https://www.fireeye.com/blog/threat-research/2020/10/kegtap-and-singlemalt-with-a-ransomware-chaser.html
[^11]: Cybersecurity Advisory AA20-302A, CISA, https://us-cert.cisa.gov/ncas/alerts/aa20-302a
[^12]: Ryuk Speed Run 2, Hours to Ransom, The DFIR Report, https://thedfirreport.com/2020/11/05/ryuk-speed-run-2-hours-to-ransom/
[^13]: FIN12 Group Profile, Mandiant, https://www.mandiant.com/sites/default/files/2021-10/fin12-group-profile.pdf
[^14]: Kerberoasting Without Mimikatz, Will Schroeder, https://blog.harmj0y.net/powershell/kerberoasting-without-mimikatz/

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
