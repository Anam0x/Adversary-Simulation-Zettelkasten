---
aliases:
tags:
  - ⌛Case_Study
primary-categories:
  - "[[Training]]"
secondary-categories:
  - "[[Hack the Box]]"
type: Case Study
engagement-type: Training Exercise
target-environment: Ubuntu-based WordPress host with local MySQL backend
used-tradecraft:
  - <!-- [[Tradecraft Note]] -->
used-tools:
  - <!-- [[Tool Note]] -->
exploited-vulnerabilities:
  - <!-- [[Vulnerability Note]] -->
start-date: <!-- YYYY-MM-DD -->
end-date: <!-- YYYY-MM-DD -->
note-status: ☑️ Ready
---
# [[Example Hack The Box Retired Machine]]

---
## Overview

This retired Hack The Box Linux machine simulated a typical web-facing content management system misconfiguration, presenting opportunities to exercise web application exploitation, credential reuse, and Linux privilege escalation techniques.

This case study captures a compact end-to-end Linux compromise: exploit a public-facing application, extract reusable credentials, pivot through remote access, and abuse a local privilege escalation path.

## Scenario Summary

Initial access was achieved by exploiting a vulnerable WordPress plugin that permitted unauthenticated remote code execution (RCE), allowing the attacker to drop a reverse shell. Lateral movement was facilitated by enumerating local WordPress configuration files, extracting hashed user credentials, and successfully cracking a low-complexity password. Privilege escalation was obtained through a misconfigured `sudo` rule that permitted the compromised user to run `perl` with *root* privileges, enabling immediate shell access as root.

## Environment Overview

* **TEST.HTB (`192.168.X.Y`)**:
	* **Operating System**: Linux (Ubuntu-based distribution)
	* **Primary Service**: Apache web server hosting a WordPress blog
	* **Secondary Service(s)**: N/A
	* **Exposed Port(s)**: `80/tcp`, `22/tcp`
	* **Application Stack**:
		* WordPress CMS (vulnerable plugin enabled)
		* PHP backend
		* MySQL database (internally accessible)
	* **User Accounts**:
		* **WordPress**:
			* *admin* (password unknown)
			* *paul*:*qwerty123*
		* **TEST.HTB**:
			* *paul*:*qwerty123*
			* *root* (password unknown)
	* **Privilege Escalation Vector**:
		* Misconfigured `sudoers` file allowed password-less execution of `perl` as *root*:
			```bash
			perl -e 'exec "/bin/sh";'
			```

## Sequence Of Events

1. Enumerated exposed web services and identified WordPress attack surface
2. Exploited the vulnerable plugin to gain command execution
3. Extracted local application secrets and recovered reusable credentials
4. Pivoted into an interactive shell via SSH
5. Abused permissive `sudo` configuration to obtain root

## Operational Breakdown

### Phase Breakdown

| Phase                | Objective                                                        | Tradecraft/ATT&CK                                                                                    | Tools/Commands                                                                                  | Notes                                                                                         |
| -------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Reconnaissance       | Confirm exposed services and identify a likely application stack | Service discovery followed by targeted CMS enumeration                                               | `nmap -sCV -Pn`, WPScan, Gobuster                                                               | HTTP and SSH were the key exposed paths; WordPress quickly became the focus                   |
| Initial Access       | Convert a vulnerable plugin into code execution                  | Exploit Public-Facing Application (T1190), Command and Scripting Interpreter: Unix Shell (T1059.004) | Plugin exploit path, reverse shell one-liner, shell stabilization commands                      | Web-shell quality was limited, so the immediate goal was a more durable operator foothold     |
| Credential Access    | Recover reusable secrets from the application layer              | File and Directory Discovery (T1083), Brute Force: Password Cracking (T1110.002)                     | Review of `wp-config.php`, extraction of WordPress-linked credentials, offline cracking tooling | The WordPress/application context yielded credentials that were useful beyond the app itself  |
| Lateral Movement     | Pivot into a cleaner remote access channel                       | Remote Services: SSH (T1021.004)                                                                     | `ssh paul@<target>`                                                                             | SSH reduced fragility and made post-exploitation faster than staying inside the initial shell |
| Privilege Escalation | Escalate from the compromised user to root                       | Abuse Elevation Control Mechanism: Sudo (T1548.003)                                                  | `sudo -l`, `perl -e 'exec \"/bin/sh\";'`                                                        | The final escalation path was straightforward once the permissive sudo entry was identified   |

### Tooling/Configuration

| Tool | Role In Operation | Flags/Configs | Tradeoffs/Notes |
| ---- | ----------------- | ------------- | --------------- |
| Nmap | Service discovery and initial surface mapping | `-sCV -Pn` | Fast way to validate exposed services before deeper web enumeration |
| WPScan | WordPress and plugin enumeration | `--url http://<target> --enumerate ap,u,t --plugins-detection aggressive` | High-value when WordPress is confirmed, but can become noisy quickly |
| Gobuster | Directory and path discovery | `dir -u http://<target> -w /usr/share/wordlists/dirb/common.txt` | Useful for confirming reachable plugin paths and web content layout |
| John the Ripper/Hashcat | Offline credential recovery | WordPress-compatible cracking mode | Effective only if the recovered hash is weak enough to justify the effort |
| OpenSSH client | Interactive post-exploitation pivot | `ssh paul@<target>` | Cleaner shell stability than staying in a brittle web shell |

## Observations

- The machine is a good compact example of chaining web exploitation with straightforward Linux post-exploitation
- Credential reuse across application and system boundaries remains a practical escalation path
- Lightweight GTFOBins-style privilege escalation vectors are worth capturing in reusable playbooks

## Follow-up Items

* [x] Review additional `sudo` misconfigurations and GTFOBins related to `perl`
* [x] Explore automation for plugin-specific enumeration post-WPScan
* [x] Add SSH lateral movement checklist to internal Playbook template

---

## Related Notes

### Same Classification

#### Similar Case Studies
```dataview
LIST
FROM "03 - Content"
WHERE type = "Case Study"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND engagement-type = this.engagement-type
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Used Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.used-tradecraft, file.link)
SORT file.name ASC
```

#### Used Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.used-tools, file.link)
SORT file.name ASC
```

#### Exploited Vulnerabilities
```dataview
LIST
FROM "03 - Content"
WHERE type = "Vulnerability"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.exploited-vulnerabilities, file.link)
SORT file.name ASC
```

---

## Resources

| Reference | Info |
| --------- | ---- |
| [MITRE ATT&CK Framework, MITRE](https://attack.mitre.org/)                      | MITRE ATT&CK Framework           |
| [Example Machine, Hack The Box](https://www.hackthebox.com/hacker/hacking-labs) | Hack The Box online lab platform |

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
