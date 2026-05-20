## Description

> [!todo] Todo: DELETE ME
> > [!tip]
> > Document offensive techniques, tactics, and procedures (TTPs) from established frameworks or custom tradecraft.

## Access Requirements

### Host

- [ ] Prerequisite 1
	- [ ] Prerequisite 1.1
- [ ] Prerequisite 2
	- [ ] Prerequisite 2.1

### Network

- [ ] Prerequisite 1
	- [ ] Prerequisite 1.1
- [ ] Prerequisite 2
	- [ ] Prerequisite 2.1

## Procedure

1. <!-- Step 1 -->
2. <!-- Step 2 -->

## Related Techniques

- Add link(s) [[]] back to related TECHNIQUE content notes

## Operational Considerations

<!-- Add any concerns related to live deployment of this technique -->
<!-- Delete any unnecessary sections -->

### Denial of Service Risk

- 

### OPSEC and Evasion

- 

### Common Issues

- 

### Success Indicators

- 

## Detections

### Host

### Network

---

## Connected Notes

### Related Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND contains(implements-techniques, this.file.link)
SORT tool-category ASC, file.name ASC
```

### Related Commands
```dataview
LIST
FROM "03 - Content"
WHERE type = "Command"
  AND contains(related-techniques, this.file.link)
SORT platform ASC, file.name ASC
```

### Related Security Controls
```dataview
TABLE control-type AS "Type", bypass-difficulty AS "Bypass Difficulty"
FROM "03 - Content"
WHERE type = "Security Control"
  AND contains(file.outlinks, this.file.link)
SORT bypass-difficulty DESC, file.name ASC
```

### Related Case Studies
```dataview
LIST
FROM "03 - Content"
WHERE type = "Case Study"
  AND contains(techniques-used, this.file.link)
SORT engagement-date DESC
```

### Related Playbooks
```dataview
LIST
FROM "03 - Content"
WHERE type = "Playbook"
  AND contains(required-techniques, this.file.link)
SORT complexity ASC
```

### Related Offensive Code
```dataview
LIST
FROM "03 - Content"
WHERE type = "Offensive Code"
  AND contains(implements-technique, this.file.link)
SORT language ASC
```

## Procedure Examples

| Source | Actor/Tool/Campaign | Description | Reference |
| ------ | ------------------- | ----------- | --------- |

## Mitigations

| Mitigation | Description | Framework Reference |
| ---------- | ----------- | ------------------- |

## Detection

| Data Source | Data Component | Detection Logic | Framework Reference |
| ----------- | -------------- | --------------- | ------------------- |

## Framework Mappings

> [!todo] 
> Cross-reference to multiple frameworks when applicable, otherwise delete this section.

| Framework        | ID  | Name | URL |
| ---------------- | --- | ---- | --- |
| MITRE ATT&CK     |     |      |     |
| Kill Chain Phase |     |      |     |
| Other            |     |      |     |
