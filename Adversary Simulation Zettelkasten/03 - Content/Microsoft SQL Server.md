---
aliases:
  - MSSQL
  - SQL Server
tags:
  - 🎯Attack_Surface
primary-categories:
  - "[[Network Security]]"
  - "[[Penetration Test]]"
secondary-categories:
  - "[[Active Directory]]"
type: Attack Surface
platforms:
  - Windows
deployment-models:
  - On-Premises
  - Cloud
authentication-methods:
  - Kerberos
  - NTLM
  - Certificates
related-controls:
  - <!-- [[Security Control Note]] -->
related-tradecraft:
  - "[[Kerberoasting]]"
related-vulnerabilities:
  - <!-- [[Vulnerability Note]] -->
related-tools:
  - <!-- [[Tool Note]] -->
note-status: ☑️ Ready
---
# [[Microsoft SQL Server]]

---

## Overview

Microsoft SQL Server is a high-value enterprise attack surface because it blends data access, privileged service execution, domain authentication, and remote administrative interfaces into one platform. In real environments it often sits at the intersection of application trust, service-account management, and lateral-movement opportunities[^1].

## Architecture Summary

> [!todo]- Diagram/Visual Aid
> Add a simple architecture diagram showing the SQL listener, SQL Agent, service identity, and key trust relationships to adjacent systems when this note is expanded further.

| Component | Role | Exposure/Trust | Why It Matters |
| --------- | ---- | -------------- | -------------- |
| Database Engine Service | Processes queries, stores data, and exposes the primary SQL listener | Reachable by applications, administrators, and sometimes broad internal user populations over TDS | It is the main control point for query execution, credential exposure, and direct interaction with the attack surface |
| SQL Agent | Runs scheduled jobs, maintenance tasks, and operator workflows | Trusted by administrators and often inherits host/service-level execution paths | It can turn database-level access into code execution, persistence, or indirect privilege escalation |
| Domain-Integrated Service Identity | Authenticates SQL Server to domain resources and may register SPNs | Trusted by Active Directory, remote services, and supporting enterprise infrastructure | It creates identity-based attack paths such as Kerberoasting, delegation abuse, or lateral movement through reusable service credentials |

## Trust Boundaries

| Boundary Type | What Crosses It | Why It Matters |
| ------------- | --------------- | -------------- |
| Identity Boundary | SQL logins, Windows-integrated authentication, service accounts, delegated application identities | Determines who can request access and which credentials can be reused or abused |
| Administrative Boundary | `sysadmin`, SQL Agent operators, linked-server administrators, host-level administrators | Defines where operational control can turn into code execution or persistence |
| Data Boundary | Databases, backup files, linked servers, secrets stored in tables/jobs/connection strings | Often contains the actual business value or credential material defenders care about |
| Network Boundary | Client connections, management traffic, cluster communication, outbound access from the SQL host | Creates the paths used for administration, pivoting, and service-to-service trust |

## Key Components

### Database Engine Service

> [!todo]- Diagram/Visual Aid
> Add an architecture diagram or other visual aid when it materially improves understanding of the attack surface component.

**Purpose**: Process queries, manage storage, and expose the main SQL listener

**Security Relevance**:
- Often runs under a domain-backed or local service account
- May expose command-execution paths through extended procedures or integrated features

**Notable Interfaces**:
- TDS listener on `1433/tcp`
- Windows service context
- Management tooling such as SSMS or `sqlcmd`

**Assumptions/Weak Points**:
- Weak service-account hygiene can make the host relevant to credential-access tradecraft
- Overprivileged SQL roles often collapse application/data separation

### SQL Agent


> [!todo]- Diagram/Visual Aid
> Add an architecture diagram or other visual aid when it materially improves understanding of the attack surface component.

**Purpose**: Execute scheduled jobs, maintenance routines, and operator-defined workflows

**Security Relevance**:
- Jobs can become an execution and persistence mechanism if an operator reaches the right role set
- Proxy accounts or job-step credentials may expose additional privilege

**Notable Interfaces**:
- Job scheduling
- CmdExec/PowerShell job steps
- Proxy and credential objects

**Assumptions/Weak Points**:
- Maintenance jobs are often trusted and under-monitored
- Operators sometimes inherit more power through job configuration than through direct query access

### Domain-Integrated Service Identity


> [!todo]- Diagram/Visual Aid
> Add an architecture diagram or other visual aid when it materially improves understanding of the attack surface component.

**Purpose**: Lets SQL Server authenticate to remote systems, register SPNs, and participate in Windows domain workflows

**Security Relevance**:
- Service accounts with SPNs can become targets for Kerberoasting
- Delegation, constrained delegation, and linked-server trust may widen access paths beyond the database itself

**Notable Interfaces**:
- SPNs
- Kerberos authentication
- Outbound access to file shares, backup targets, or linked infrastructure

**Assumptions/Weak Points**:
- Service accounts are frequently long-lived and manually managed
- SQL administrators may not own or review the surrounding AD identity posture closely enough

## Assessment Notes

### Common Attack Paths

| Attack Path | Preconditions | Likely Outcome |
| ----------- | ------------- | -------------- |
| Abuse SQL authentication and reach execution features | Valid credentials or overexposed SQL access | Query execution, host access, or job-based persistence |
| Target the SQL service account or SQL Agent context | SPN-backed or overprivileged service identity | Credential theft, lateral movement, or escalation |
| Use linked-server/backups/integrated trust paths | Remote access from the SQL host to other systems | Pivoting beyond the database tier |

### Security Model

| Control Area | Notes |
| ------------ | ----- |
| Authentication | Can be SQL-native, Windows-integrated, or mixed |
| Authorization | Layered across server roles, database roles, application identities, and host privileges |
| Isolation/Tenancy | Real exposure often depends more on surrounding trust relationships than on the listener alone |

### Common Weaknesses

| Weakness | Why It Recurs | What It Enables |
| -------- | ------------- | --------------- |
| Overprivileged service accounts | Service identities are often long-lived and hard to rotate | Credential access, lateral movement, and trust abuse |
| Unreviewed SQL Agent jobs and proxy accounts | Maintenance workflows are often trusted by default | Job-based execution, persistence, or privilege escalation |
| Linked servers and backup paths | Convenience features expand trust quietly over time | Unexpected remote access and data exposure |
| Weak auditing/TLS/rotation practices | Database administration maturity varies widely | Harder detection and easier operator persistence |

## Defensive Notes

### High-Value Controls

| Control | Why It Matters |
| ------- | -------------- |
| Strong service-account management | Reduces the value of SQL-linked credential access and Kerberoasting-style abuse |
| Restrictive role assignment | Limits which users can turn database access into execution or persistence |
| Monitoring around jobs/logins/outbound activity | Surfaces the control points most likely to show attacker behavior |

### Logging/Telemetry Considerations

| Source | What It Shows | Investigative Use |
| ------ | ------------- | ----------------- |
| Login telemetry | Failed and successful SQL authentication | Spot suspicious access patterns and brute-force behavior |
| SQL Agent job history | Job creation, modification, and execution | Detect execution and persistence attempts |
| AD service-account/SPN visibility | Identity context around the SQL service | Correlate database exposure with credential-access tradecraft[^2] |

---

## Related Notes

### Same Classification

#### Similar Attack Surfaces
```dataview
LIST
FROM "03 - Content"
WHERE type = "Attack Surface"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    contains(platforms, this.platforms[0]) OR
    contains(deployment-models, this.deployment-models[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Related Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.related-tradecraft, file.link)
SORT file.name ASC
```

#### Related Vulnerabilities
```dataview
LIST
FROM "03 - Content"
WHERE type = "Vulnerability"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.related-vulnerabilities, file.link)
SORT file.name ASC
```

#### Related Security Controls
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.related-controls, file.link)
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

| Reference                                                                                                        | Info                                                                       |
| ---------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| [What is SQL Server?, Microsoft](https://learn.microsoft.com/en-us/sql/sql-server/what-is-sql-server)            | Primary note-level reference for platform purpose and deployment context   |
| [Service Principal Names, Microsoft](https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names) | Secondary note-level reference for identity/service-account considerations |

[^1]: What is SQL Server?, Microsoft, https://learn.microsoft.com/en-us/sql/sql-server/what-is-sql-server
[^2]: Service Principal Names, Microsoft, https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
