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
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.covers-tradecraft, file.link)
SORT file.name ASC
```

#### Covers Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.covers-tools, file.link)
SORT file.name ASC
```
