## Related Notes

### Same Classification

#### Similar Study Resources
```dataview
LIST
FROM "03 - Content"
WHERE type = "Study Resources"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    provider = this.provider OR
    status = this.status
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Covers Tradecraft
```dataview
LIST WITHOUT ID covers-tradecraft
WHERE file = this.file
FLATTEN covers-tradecraft
SORT covers-tradecraft ASC
```

#### Covers Tools
```dataview
LIST WITHOUT ID covers-tools
WHERE file = this.file
FLATTEN covers-tools
SORT covers-tools ASC
```
