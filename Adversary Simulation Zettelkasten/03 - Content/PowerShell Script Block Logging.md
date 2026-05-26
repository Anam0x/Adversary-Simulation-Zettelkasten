---
aliases:
  - PS Script Block Logging
tags:
  - 🧱Security_Control
primary-categories:
  - "[[Detection Engineering]]"
secondary-categories:
  - "[[PowerShell]]"
type: Security Control
control-category: Telemetry
protects-platforms:
  - Windows
detects-tradecraft:
  - "[[Kerberoasting]]"
known-bypasses:
telemetry-sources:
  - Windows PowerShell Event ID 4104
  - PowerShell Operational Log
enforcement-points:
  - PowerShell logging policy
  - Endpoint telemetry pipeline
note-status: ☑️ Ready
---
# [[PowerShell Script Block Logging]]

---

## Overview

PowerShell Script Block Logging captures the contents of PowerShell code as it is interpreted, giving defenders much stronger visibility into command content than process-command-line logging alone. It is especially useful in environments where operators rely on native PowerShell tradecraft, encoded commands, or in-memory script execution[^1].

## Control Summary

| Dimension | Notes |
| --------- | ----- |
| Primary Purpose | Detection and visibility |
| Coverage Model | Captures interpreted PowerShell script content on Windows hosts where the feature is enabled and logs are collected |
| Operational Dependencies | PowerShell policy configuration, log forwarding/retention, and analyst ability to query Event ID 4104 content at scale |

## How It Works

### Enforcement/Detection Logic

| Check/Mechanism | What It Does | Why It Matters |
| --------------- | ------------ | -------------- |
| Script block capture | Records PowerShell code as it is interpreted by the engine | Gives visibility into decoded or dynamically assembled content |
| Event emission | Sends content into Event ID 4104 and related logging channels | Makes downstream hunting, alerting, and retrospective review possible |
| Correlation with process context | Pairs script content with parent process and user/session activity | Helps distinguish benign administration from suspicious operator behavior |

### Deployment Considerations

| Consideration | Notes |
| ------------- | ----- |
| Placement | Must be enabled on Windows endpoints or servers where PowerShell activity matters |
| Tuning | Collection pipelines need to handle potentially high event volume |
| Trust Assumptions | Analysts need process, host, and user context; script text alone is rarely enough |

## Assessment Notes

| Scenario | Strengths | Blind Spots | Bypass Considerations |
| -------- | --------- | ----------- | --------------------- |
| Encoded or obfuscated PowerShell usage | Exposes decoded or reconstructed content better than command-line logging alone | Still depends on collection, retention, and analyst review quality | Operators may switch to non-PowerShell tooling or alternate runtimes |
| Administrative-heavy environments | Provides rich script context for investigation | Can be noisy if the environment automates heavily with PowerShell | Overly broad exclusions can create blind spots exactly where visibility is needed |

## Adversary Tradeoffs

### OPSEC Considerations

- [ ] PowerShell-heavy tradecraft becomes far riskier when the operator expects decoded script content to be logged
- [ ] Defenders gain disproportionate value against encoded or in-memory PowerShell compared to environments that only log process command lines

### Validation Ideas

- [ ] Run a benign encoded PowerShell command and confirm the decoded content appears in Event ID 4104
- [ ] Test whether detection content catches common AD enumeration or credential-access modules without overwhelming analysts with noise

## Detection/Response Notes

### Useful Telemetry

| Source | What It Gives You | Investigative Value |
| ------ | ----------------- | ------------------- |
| Event ID 4104 | Script block contents | Best source for seeing decoded or reconstructed PowerShell behavior |
| Parent process context | Launch source and process ancestry | Helps separate administrative automation from suspicious staging |
| User/session context | Identity and session scope | Useful for prioritizing execution from unexpected users or hosts |
| Process creation telemetry | Command-line, binary, and timing context | Strong correlation source when script blocks alone are ambiguous |

### Tuning Notes

| Tuning Area | Guidance | Risk If Mis-Tuned |
| ----------- | -------- | ----------------- |
| Administrative automation | Filter known-good workflows carefully rather than disabling logging broadly | Over-filtering can erase the most realistic operator emulation signals |
| Detection scope | Focus first on encoded commands, download behavior, and credential-access patterns | Broad untuned alerting can create fatigue and reduce analyst trust |

---

## Related Notes

### Same Classification

#### Controls In Same Category
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    control-category = this.control-category OR
    contains(protects-platforms, this.protects-platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Detects Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.detects-tradecraft, file.link)
SORT file.name ASC
```

#### Known Bypasses
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.known-bypasses, file.link)
SORT file.name ASC
```

### Reverse Relationships

#### Referenced By Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(bypasses-controls, this.file.link)
SORT file.name ASC
```

---

## Resources

| Reference                                                                                                                                     | Info                                                              |
| --------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| [about_Logging_Windows, Microsoft](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows) | Primary note-level source for control behavior and deployment     |
| [Malicious PowerShell Process - Encoded Command, Splunk](https://research.splunk.com/endpoint/c4db14d9-7909-48b4-a054-aa14d89dbb19/)          | Secondary note-level source for detection-oriented tuning context |

[^1]: about_Logging_Windows, Microsoft, https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
