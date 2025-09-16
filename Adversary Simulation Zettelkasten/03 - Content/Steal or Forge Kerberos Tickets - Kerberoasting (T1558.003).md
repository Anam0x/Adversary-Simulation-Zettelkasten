---
aliases:
  - T1558.003
  - Kerberoasting
tags:
  - 📕TTP
primary categories:
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary categories:
  - "[[Active Directory]]"
  - "[[Credential Harvesting]]"
  - "[[Kerberoasting]]"
  - "[[Post-Exploitation]]"
type: TTP
---
# [[Steal or Forge Kerberos Tickets - Kerberoasting (T1558.003)]]

***
## Overview

```ad-info
The content in this page is directly adapted from the [MITRE ATT&CK](https://attack.mitre.org/techniques/T1558/003/) framework for personal reference. Please consult the original MITRE ATT&CK entry for the most up-to-date and complete information.
```

Adversaries may abuse a valid Kerberos ticket-granting ticket (TGT) or sniff network traffic to obtain a ticket-granting service (TGS) ticket that may be vulnerable to [Brute Force](https://attack.mitre.org/techniques/T1110)[^1][^2].

Service principal names (SPNs) are used to uniquely identify each instance of a Windows service. To enable authentication, Kerberos requires that SPNs be associated with at least one service logon account (an account specifically tasked with running a service[^3])[^4][^5][^6][^7].

Adversaries possessing a valid Kerberos ticket-granting ticket (TGT) may request one or more Kerberos ticket-granting service (TGS) service tickets for any SPN from a domain controller (DC)[^1][^2]. Portions of these tickets may be encrypted with the RC4 algorithm, meaning the Kerberos 5 TGS-REP etype 23 hash of the service account associated with the SPN is used as the private key and is thus vulnerable to offline Brute Force attacks that may expose plaintext credentials[^2][^1][^7].

This same behavior could be executed using service tickets captured from network traffic[^2].

Cracked hashes may enable Persistence, Privilege Escalation, and Lateral Movement via access to Valid Accounts[^6].

## Procedure Examples

| ID                                                | Type                                             | Name                                                                        | Description                                                                                                                                                                                                                                                                            |
| ------------------------------------------------- | ------------------------------------------------ | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [S1063](https://attack.mitre.org/software/S1063)  | [Software](https://attack.mitre.org/software/)   | [Brute Ratel C4](https://attack.mitre.org/software/S1063)                   | [Brute Ratel C4](https://attack.mitre.org/software/S1063) can decode Kerberos 5 tickets and convert it to hashcat format for subsequent cracking[^8].                                                                                                                                  |
| [S0363](https://attack.mitre.org/software/S0363)  | [Software](https://attack.mitre.org/software/)   | [Empire](https://attack.mitre.org/software/S0363)                           | [Empire](https://attack.mitre.org/software/S0363) uses [PowerSploit](https://attack.mitre.org/software/S0194)'s `Invoke-Kerberoast` to request service tickets and return crackable ticket hashes[^9].                                                                                 |
| [G0046](https://attack.mitre.org/groups/G0046)    | [Groups](https://attack.mitre.org/groups/)       | [FIN7](https://attack.mitre.org/groups/G0046)                               | [FIN7](https://attack.mitre.org/groups/G0046) has used Kerberoasting PowerShell commands such as, `Invoke-Kerberoast` for credential access and to enable lateral movement[^10][^11].                                                                                                  |
| [S0357](https://attack.mitre.org/software/S0357)  | [Software](https://attack.mitre.org/software/)   | [Impacket](https://attack.mitre.org/software/S0357)                         | [Impacket](https://attack.mitre.org/software/S0357) modules like `GetUserSPNs` can be used to get Service Principal Names (SPNs) for user accounts. The output is formatted to be compatible with cracking tools like John the Ripper and Hashcat[^12].                                |
| [G0119](https://attack.mitre.org/groups/G0119)    | [Groups](https://attack.mitre.org/groups/)       | [Indrik Spider](https://attack.mitre.org/groups/G0119)                      | [Indrik Spider](https://attack.mitre.org/groups/G0119) has conducted Kerberoasting attacks using a module from GitHub[^13].                                                                                                                                                            |
| [C0049](https://attack.mitre.org/campaigns/C0049) | [Campaigns](https://attack.mitre.org/campaigns/) | [Leviathan Australian Intrusions](https://attack.mitre.org/campaigns/C0049) | [Leviathan](https://attack.mitre.org/groups/G0065) used Kerberoasting techniques during [Leviathan Australian Intrusions](https://attack.mitre.org/campaigns/C0049)[^14].                                                                                                              |
| [C0014](https://attack.mitre.org/campaigns/C0014) | [Campaigns](https://attack.mitre.org/campaigns/) | [Operation Wocao](https://attack.mitre.org/campaigns/C0014)                 | During [Operation Wocao](https://attack.mitre.org/campaigns/C0014), threat actors used [PowerSploit](https://attack.mitre.org/software/S0194)'s `Invoke-Kerberoast` module to request encrypted service tickets and bruteforce the passwords of Windows service accounts offline[^15]. |
| [S0194](https://attack.mitre.org/software/S0194)  | [Software](https://attack.mitre.org/software/)   | [PowerSploit](https://attack.mitre.org/software/S0194)                      | [PowerSploit](https://attack.mitre.org/software/S0194)'s `Invoke-Kerberoast` module can request service tickets and return crackable ticket hashes[^16][^7].                                                                                                                           |
| [S1071](https://attack.mitre.org/software/S1071)  | [Software](https://attack.mitre.org/software/)   | [Rubeus](https://attack.mitre.org/software/S1071)                           | [Rubeus](https://attack.mitre.org/software/S1071) can use the `KerberosRequestorSecurityToken.GetRequest` method to request kerberoastable service tickets[^17].                                                                                                                       |
| [S0692](https://attack.mitre.org/software/S0692)  | [Software](https://attack.mitre.org/software/)   | [SILENTTRINITY](https://attack.mitre.org/software/S0692)                    | [SILENTTRINITY](https://attack.mitre.org/software/S0692) contains a module to conduct Kerberoasting[^18].                                                                                                                                                                              |
| [C0024](https://attack.mitre.org/campaigns/C0024) | [Campaigns](https://attack.mitre.org/campaigns/) | [SolarWinds Compromise](https://attack.mitre.org/campaigns/C0024)           | During the [SolarWinds Compromise](https://attack.mitre.org/campaigns/C0024), [APT29](https://attack.mitre.org/groups/G0016) obtained Ticket Granting Service (TGS) tickets for Active Directory Service Principle Names to crack offline[^19].                                        |
| [G0102](https://attack.mitre.org/groups/G0102)    | [Groups](https://attack.mitre.org/groups/)       | [Wizard Spider](https://attack.mitre.org/groups/G0102)                      | [Wizard Spider](https://attack.mitre.org/groups/G0102) has used Rubeus, MimiKatz Kerberos module, and the `Invoke-Kerberoast` cmdlet to steal AES hashes[^20][^21][^22][^23][^24].                                                                                                     |

## Mitigations

| ID                                                  | Mitigation                                                                  | Description                                                                                                                                                                                                                                               |
| --------------------------------------------------- | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [M1041](https://attack.mitre.org/mitigations/M1041) | [Encrypt Sensitive Information](https://attack.mitre.org/mitigations/M1041) | Enable AES Kerberos encryption (or another stronger encryption algorithm), rather than RC4, where possible[^2].                                                                                                                                           |
| [M1027](https://attack.mitre.org/mitigations/M1027) | [Password Policies](https://attack.mitre.org/mitigations/M1027)             | Ensure strong password length (ideally 25+ characters) and complexity for service accounts and that these passwords periodically expire. Also consider using Group Managed Service Accounts or another third party product such as password vaulting[^2]. |
| [M1026](https://attack.mitre.org/mitigations/M1026) | [Privileged Account Management](https://attack.mitre.org/mitigations/M1026) | Limit service accounts to minimal required privileges, including membership in privileged groups such as Domain Administrators[^2].                                                                                                                       |

## Detection

| ID                                                    | Data Source                                                     | Data Component                                                                                                                | Detects                                                                                                                                                                                                                                                                                                                                                  |
| ----------------------------------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [DS0026](https://attack.mitre.org/datasources/DS0026) | [Active Directory](https://attack.mitre.org/datasources/DS0026) | [Active Directory Credential Request](https://attack.mitre.org/datasources/DS0026/#Active%20Directory%20Credential%20Request) | Monitor for anomalous Kerberos activity, such as enabling Audit Kerberos Service Ticket Operations to log Kerberos TGS service ticket requests. Particularly investigate irregular patterns of activity (ex: accounts making numerous requests, Event ID 4769, within a small time frame, especially if they also request RC4 encryption \[Type 0x17\]). |

***
## Resources

| Hyperlink                                                                                        | Info                                                     |
| ------------------------------------------------------------------------------------------------ | -------------------------------------------------------- |
| [Steal or Forge Kerberos Tickets: Kerberoasting](https://attack.mitre.org/techniques/T1558/003/) | MITRE ATT&CK entry for "Kerberoasting"                   |
| [Steal or Forge Kerberos Tickets, Technique T1558](https://attack.mitre.org/techniques/T1558/)   | MITRE ATT&CK entry for "Steal or Forge Kerberos Tickets" |

[^1]: Invoke-Kerberoast.ps1, Empire Project, https://github.com/EmpireProject/Empire/blob/master/data/module_source/credentials/Invoke-Kerberoast.ps1
[^2]: Cracking Kerberos TGS Tickets Using Kerberoast – Exploiting Kerberos to Compromise the Active Directory Domain, Sean Metcalf, https://adsecurity.org/?p=2293
[^3]: Detecting Kerberoasting activity using Azure Security Center, Microsoft, https://learn.microsoft.com/en-us/archive/blogs/motiba/detecting-kerberoasting-activity-using-azure-security-center
[^4]: Service Principal Names, Microsoft, https://msdn.microsoft.com/library/ms677949.aspx
[^5]: Service Principal Names (SPNs) SetSPN Syntax (Setspn.exe), Microsoft TechNet, https://social.technet.microsoft.com/wiki/contents/articles/717.service-principal-names-spns-setspn-syntax-setspn-exe.aspx
[^6]: Attacking Kerberos, Red Siege, https://redsiege.com/kerberoast-slides
[^7]: Kerberoasting Without Mimikatz, Will Schroeder, https://blog.harmj0y.net/powershell/kerberoasting-without-mimikatz/
[^8]: When Pentest Tools Go Brutal: Red-Teaming Tool Being Abused by Malicious Actors, Unit 42, https://unit42.paloaltonetworks.com/brute-ratel-c4-tool/
[^9]: PowerShell Empire, Empire Project, https://github.com/PowerShellEmpire/Empire
[^10]: Carbon Spider Embraces Big Game Hunting Part 1, CrowdStrike, https://www.crowdstrike.com/blog/carbon-spider-embraces-big-game-hunting-part-1/
[^11]: FIN7 Power Hour: Adversary Archaeology and the Evolution of FIN7, Google Cloud, https://www.mandiant.com/resources/evolution-of-fin7
[^12]: Impacket, SecureAuth, https://www.secureauth.com/labs/open-source-tools/impacket
[^13]: To HADES and Back: UNC2165 Shifts to LOCKBIT to Evade Sanctions, Google Cloud, https://cloud.google.com/blog/topics/threat-intelligence/unc2165-shifts-to-evade-sanctions/
[^14]: Cybersecurity Advisory AA24-190A, CISA, https://www.cisa.gov/news-events/cybersecurity-advisories/aa24-190a
[^15]: Operation Wocao, Fox-IT, https://www.fox-it.com/media/kadlze5c/201912_report_operation_wocao.pdf
[^16]: Invoke-Kerberoast, PowerSploit, https://powersploit.readthedocs.io/en/latest/Recon/Invoke-Kerberoast/
[^17]: Rubeus, GhostPack, https://github.com/GhostPack/Rubeus
[^18]: SILENTTRINITY, byt3bl33d3r, https://github.com/byt3bl33d3r/SILENTTRINITY/tree/master/silenttrinity/core/teamserver/modules/boo
[^19]: Deep Dive Into the Solorigate Second-Stage Activation, Microsoft Security, https://www.microsoft.com/security/blog/2021/01/20/deep-dive-into-the-solorigate-second-stage-activation-from-sunburst-to-teardrop-and-raindrop/
[^20]: Ryuk's Return, The DFIR Report, https://thedfirreport.com/2020/10/08/ryuks-return/
[^21]: Unhappy Hour Special: KEGTAP and SINGLEMALT With a Ransomware Chaser, Google Cloud, https://www.fireeye.com/blog/threat-research/2020/10/kegtap-and-singlemalt-with-a-ransomware-chaser.html
[^22]: Cybersecurity Advisory AA20-302A, CISA, https://us-cert.cisa.gov/ncas/alerts/aa20-302a
[^23]: Ryuk Speed Run 2, Hours to Ransom; The DFIR Report; https://thedfirreport.com/2020/11/05/ryuk-speed-run-2-hours-to-ransom/
[^24]: FIN12 Group Profile, Mandiant, https://www.mandiant.com/sites/default/files/2021-10/fin12-group-profile.pdf

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
