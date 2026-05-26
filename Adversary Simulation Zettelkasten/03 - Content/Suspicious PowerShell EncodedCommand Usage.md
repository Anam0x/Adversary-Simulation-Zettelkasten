---
aliases:
  - EncodedCommand IOC
  - PowerShell -EncodedCommand
tags:
  - 👣IOC
primary-categories:
  - "[[Detection Engineering]]"
  - "[[Threat Intelligence]]"
secondary-categories:
  - "[[PowerShell]]"
  - "[[Post-Exploitation]]"
type: IOC
ioc-type: Command Line
indicator-value: powershell.exe -EncodedCommand
confidence: Medium
associated-tools:
  - <!-- [[Tool Note]] -->
associated-tradecraft:
  - "[[Kerberoasting]]"
active: true
note-status: ☑️ Ready
---
# [[Suspicious PowerShell EncodedCommand Usage]]

---

## Overview

This IOC tracks PowerShell executions that rely on the `-EncodedCommand` flag to stage or obscure a payload. The pattern is not malicious by itself, but it is a strong triage signal in environments where encoded command lines are uncommon or where PowerShell should be tightly controlled[^1].

## Indicator Context

| Dimension | Notes |
| --------- | ----- |
| What It Is | A command-line pattern in which `powershell.exe` or `pwsh.exe` is launched with `-EncodedCommand` or an equivalent short form |
| Why It Matters | Operators often use encoded commands to reduce quoting issues, hide immediate intent, or transport larger PowerShell content through remote execution channels |
| Scope/Confidence Notes | Medium confidence because administrators, automation, and some commercial tooling can use the same pattern legitimately |

## Detection And Investigation

### Detection Logic

| Detection Source | What To Match | Caveats |
| ---------------- | ------------- | ------- |
| Process-creation telemetry | `powershell` combined with `-EncodedCommand`, `-enc`, or similarly abbreviated forms | The token alone is insufficient without parent-process and user context |
| PowerShell logging | Decoded script content tied to suspicious encoded launches | Only useful where script logging is enabled and retained |

### False Positive Considerations

| Scenario | Why It Can Be Benign | Triage Hint |
| -------- | -------------------- | ----------- |
| Administrative automation | Encoded PowerShell is sometimes used to avoid quoting issues in deployment workflows | Check whether the parent process and host match known automation systems |
| Trusted remote execution tooling | Commercial or internal management tooling may transport PowerShell this way | Compare execution source, user, and destination behavior to known-good baselines |

### Triage Notes

| Pivot | Why It Matters |
| ----- | -------------- |
| Decode the supplied command | Quickly distinguishes commodity admin behavior from staging or post-exploitation content |
| Check adjacent discovery/auth activity | Helps reveal whether the same host is moving into credential access or lateral movement |
| Review script block and parent-process telemetry | Adds context when the command line is too compressed or ambiguous to stand alone |

### YARA/Rule Snippet

```text
Process command line contains:
  powershell
  AND (-EncodedCommand OR -enc)
```

## Related Threats/Campaigns

| Threat Actor/Campaign | Description                                                                                                                                                                                | Reference/Report                                                                                                                                  |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| AvosLocker            | Threat actor advisory for a ransomware group targeting a Cisco Talos client; one of the IOCs included in the article is an encoded PowerShell command for executing a Cobalt Strike Beacon | [Avos ransomware group expands with new attack arsenal; Chris Neal, Guilherme Venere](https://blog.talosintelligence.com/avoslocker-new-arsenal/) |

---

## Related Notes

### Same Classification

#### Similar Indicators
```dataview
LIST
FROM "03 - Content"
WHERE type = "IOC"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    ioc-type = this.ioc-type OR
    confidence = this.confidence
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Associated Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.associated-tradecraft, file.link)
SORT file.name ASC
```

#### Associated Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.associated-tools, file.link)
SORT file.name ASC
```

---

## Resources

| Reference                                                                                                                                   | Info                                                          |
| ------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| [about_PowerShell_exe, Microsoft](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_powershell_exe) | Primary note-level source for `-EncodedCommand` behavior      |
| [Malicious PowerShell Process - Encoded Command, Splunk](https://research.splunk.com/endpoint/c4db14d9-7909-48b4-a054-aa14d89dbb19/)        | Secondary note-level source for detection-oriented enrichment |

[^1]: about_PowerShell_exe, Microsoft, https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_powershell_exe

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
