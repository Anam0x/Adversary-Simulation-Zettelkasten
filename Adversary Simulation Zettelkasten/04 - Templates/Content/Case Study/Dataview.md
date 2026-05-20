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
LIST WITHOUT ID used-tradecraft
WHERE file = this.file
FLATTEN used-tradecraft
SORT used-tradecraft ASC
```

#### Used Tools
```dataview
LIST WITHOUT ID used-tools
WHERE file = this.file
FLATTEN used-tools
SORT used-tools ASC
```

#### Exploited Vulnerabilities
```dataview
LIST WITHOUT ID exploited-vulnerabilities
WHERE file = this.file
FLATTEN exploited-vulnerabilities
SORT exploited-vulnerabilities ASC
```
