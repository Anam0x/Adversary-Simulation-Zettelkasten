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
