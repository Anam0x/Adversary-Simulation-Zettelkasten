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
