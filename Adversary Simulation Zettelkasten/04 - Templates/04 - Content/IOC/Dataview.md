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
LIST WITHOUT ID associated-tradecraft
WHERE file = this.file
FLATTEN associated-tradecraft
SORT associated-tradecraft ASC
```

#### Associated Tools
```dataview
LIST WITHOUT ID associated-tools
WHERE file = this.file
FLATTEN associated-tools
SORT associated-tools ASC
```
