---
aliases:
  - Kerberos Authentication
tags:
  - 📡Protocol
primary-categories:
  - "[[Cryptography]]"
  - "[[Penetration Test]]"
secondary-categories:
  - "[[Active Directory]]"
type: Protocol
protocol-family: Authentication
ports:
  - TCP/88
authentication-methods:
  - Kerberos
  - Certificates
abused-by-tradecraft:
  - "[[Kerberoasting]]"
secured-by-controls:
  - <!-- [[Security Control Note]] -->
related-tools:
  - "[[GetUserSPNs.py]]"
note-status: ☑️ Ready
---
# [[Kerberos]]

---

## Overview

Kerberos is an authentication protocol built around ticket issuance, mutual trust, and shared-key cryptography. In enterprise Windows environments it underpins domain authentication, service access, delegation workflows, and many of the trust assumptions that matter most during adversary simulation[^1].

## Specification Details

| Topic | Notes |
| ----- | ----- |
| Primary Purpose | Let clients authenticate to services without repeatedly transmitting reusable credentials |
| Security-Relevant Properties | Ticket lifetimes, encryption types, SPN mappings, delegation behavior, and how tickets are requested or reused |
| Common Environments | Active Directory domains, Windows-integrated applications, and service-to-service enterprise authentication |

## Message Flow

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/68/Kerberos_protocol.svg/1280px-Kerberos_protocol.svg.png" style="background-color:white;">

- **Initial exchange**
	- A client requests a Ticket Granting Ticket from the KDC through the `AS-REQ`/`AS-REP` flow
- **Core request/response**
	- Once the client has a TGT, it asks the KDC for a service ticket through `TGS-REQ`/`TGS-REP`, using the target SPN to identify the service
- **Service/action stage**
	- The client presents the service ticket to the target service through `AP-REQ`
	- The service may reply with `AP-REP` if mutual authentication is used
- **Assessment notes**
	- The TGS stage is especially important during adversary simulation because it is where SPN-backed service tickets are issued and where Kerberoasting-style tradecraft derives its offline-cracking material

## Authentication/Security

| Trust Element | How It Works | Security Implication |
| ------------- | ------------ | -------------------- |
| Key Distribution Center | Central authority issues TGTs and service tickets | KDC visibility and policy matter enormously for both attack and defense |
| Service Principal Names | Map services to identities | Weak SPN-backed service accounts can become credential-access targets |
| Ticket Encryption | Protects ticket contents with account-derived material | Legacy or weak encryption choices can materially improve attacker odds |

## Common Implementations

- **Active Directory**
	- The most common enterprise Kerberos environment and the place where most operator interest in ticket issuance, SPNs, and service-account behavior comes from
- **Windows service accounts**
	- SPN-backed identities that make Kerberos practical for services
	- Creates high-value targets for service-ticket abuse
- **Enterprise applications**
	- Integrated-authentication consumers that inherit Kerberos trust assumptions without always making those relationships obvious to defenders or administrators

## Traffic Analysis

| Signal | Normal Pattern | Suspicious Pattern |
| ------ | -------------- | ------------------ |
| TGS request volume | Requests align with known user/application behavior | Bursts of service-ticket requests from unusual hosts or identities |
| Encryption usage | Matches environment policy and expected service behavior | Unexpected RC4-oriented requests or unusual etype selection |
| Authentication origin | Requests originate from expected systems | Ticketing activity from operator workstations or pivot hosts that do not fit normal use |

## Exploitation Vectors

| Abuse Path | Preconditions | Common Outcome |
| ---------- | ------------- | -------------- |
| Kerberoasting | Valid domain context plus roastable SPN-backed account | Recover crackable ticket material for offline credential attack |
| Delegation abuse | Misconfigured constrained/unconstrained delegation paths | Privilege escalation or lateral movement through trust misuse |
| Ticket theft/reuse | Access to tickets or ticketing context on a compromised host | Impersonation or access reuse without original plaintext credentials |

## Detection Signatures

| Telemetry Source | What To Look For | Caveats |
| ---------------- | ---------------- | ------- |
| Domain controller Kerberos logs | High-volume or unusual TGS request activity | Requires retention and context to distinguish admin workflows from abuse |
| Host/process telemetry | Ticketing tools, unusual PowerShell, or credential-access utilities | Tool choice varies and some workflows use native APIs rather than obvious binaries |
| Authentication correlation | Kerberos activity followed by remote admin or privilege escalation | Strong context signal, but not every suspicious request leads to immediate follow-on action |

## Related Protocols

- **NTLM**:
	- Common authentication fallback or alternative worth comparing when evaluating downgrade, relay, or mixed-auth environments
- **LDAP**:
	- Often queried alongside Kerberos workflows to discover identities, SPNs, and trust relationships
- **SMB**:
	- A frequent downstream consumer of Kerberos-authenticated access and a useful comparison point for lateral movement
- **DNS**:
	- A naming dependency that affects service discovery
	- Can materially change Kerberos behavior when resolution is broken or manipulated

---

## Related Notes

### Same Classification

#### Similar Protocols
```dataview
LIST
FROM "03 - Content"
WHERE type = "Protocol"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    protocol-family = this.protocol-family OR
    contains(authentication-methods, this.authentication-methods[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Abused By Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.abused-by-tradecraft, file.link)
SORT file.name ASC
```

#### Secured By Controls
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.secured-by-controls, file.link)
SORT file.name ASC
```

#### Related Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.related-tools, file.link)
SORT file.name ASC
```

---

## Resources

| Reference                                                                                                                                          | Info                                                                   |
| -------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| [Kerberos Authentication Overview, Microsoft](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview) | Primary note-level reference for protocol role in Windows environments |
| [Kerberos, NIST](https://csrc.nist.gov/glossary/term/kerberos)                                                                                     | Secondary note-level reference for baseline protocol terminology       |

[^1]: Kerberos Authentication Overview, Microsoft, https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview
[^2]: Kerberoasting, MITRE ATT&CK, https://attack.mitre.org/techniques/T1558/003/

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
