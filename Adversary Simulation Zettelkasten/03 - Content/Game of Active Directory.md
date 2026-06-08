---
aliases:
  - GOAD
tags:
  - 🧪Lab_Setup
primary-categories:
  - "[[Training]]"
  - "[[Penetration Test]]"
secondary-categories:
  - "[[Active Directory]]"
  - "[[OffSec]]"
  - "[[Post-Exploitation]]"
type: Lab Setup
lab-purpose: Training
platforms:
  - Windows
  - Cloud
difficulty: Advanced
estimated-build-time: 2-4 hours
practices-tradecraft:
  - "[[Kerberoasting]]"
uses-tools:
  - "[[GetUserSPNs.py]]"
note-status: ☑️ Ready
---
# [[Game of Active Directory]]

---

## Overview

Game of Active Directory is a purpose-built training lab for practicing realistic Active Directory assessment, escalation, and post-exploitation workflows. It is a strong representative lab because it includes multiple trust relationships, seeded weaknesses, and enough operational depth to exercise both tradecraft and defensive thinking[^1].

## Requirements

### Hardware

- A host system with enough RAM and CPU to run multiple Windows servers/workstations reliably
- Adequate disk space for VM images, snapshots, and tooling
- Networking support for an isolated internal lab segment

### Software

- Hypervisor support such as VMware, VirtualBox, or a compatible lab platform
- Terraform/Ansible or the supported GOAD deployment workflow
- Attacker tooling for AD enumeration, Kerberos abuse, and post-exploitation validation

### Accounts/Services

- Required cloud or virtualization credentials if building the environment through supported automation
- Seeded lab users, domain admins, and service accounts created by the lab deployment
- An operator workstation or attack VM with network reachability into the environment

## Configuration Steps

### Step 1

Prepare the host, clone the GOAD project, review the supported deployment workflow, and verify that the required hypervisor or cloud dependencies are available.

> [!todo] Step Screenshot
> Add a screenshot here if this step benefits from a visual checkpoint, UI reference, or architecture snapshot.

### Step 2

Deploy the lab controllers, member systems, and seeded misconfigurations. Confirm that the domain, trust relationships, and host naming all match the expected build output.

> [!todo] Step Screenshot
> Add a screenshot here if this step benefits from a visual checkpoint, UI reference, or architecture snapshot.

### Step 3

Snapshot the finished environment, stage the attack workstation, and validate that core scenarios such as Kerberos ticket requests, LDAP enumeration, and remote administrative paths are reachable.

> [!todo] Step Screenshot
> Add a screenshot here if this step benefits from a visual checkpoint, UI reference, or architecture snapshot.

## Validation/Testing

> [!todo] Validation Output
> Add code blocks, screenshots, or short command output samples when they clearly show the lab is working as intended.

- Confirm the lab domain resolves correctly and that domain users can authenticate to expected systems
- Test at least one target workflow such as SPN enumeration with `GetUserSPNs.py`
- Verify snapshots or rebuild workflows before making destructive changes during practice

## Reset/Maintenance Notes

- Maintain a clean post-build snapshot before running high-noise chains or destructive experiments
- Rebuild or roll back after major privilege-escalation or persistence exercises
- Track any local changes to hosts, service accounts, or policies so future sessions remain repeatable

---

## Related Notes

### Same Classification

#### Similar Lab Setups
```dataview
LIST
FROM "03 - Content"
WHERE type = "Lab Setup"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    lab-purpose = this.lab-purpose OR
    difficulty = this.difficulty OR
    contains(platforms, this.platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Practices Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.practices-tradecraft, file.link)
SORT file.name ASC
```

#### Uses Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.uses-tools, file.link)
SORT file.name ASC
```

---

## Resources

| Reference                                                                      | Info                                                                                 |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| [GOAD, Orange Cyberdefense](https://github.com/Orange-Cyberdefense/GOAD)       | Primary note-level reference for the lab project and deployment workflow             |
| [GOAD-Light, Orange Cyberdefense](https://orange-cyberdefense.github.io/GOAD/) | Secondary note-level reference for walkthroughs, documentation, and setup validation |

[^1]: GOAD, Orange Cyberdefense, https://github.com/Orange-Cyberdefense/GOAD

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
