## Related Notes

### Same Classification

#### Similar Reference Material
```dataview
LIST
FROM "03 - Content"
WHERE type = "Reference Material"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    resource-type = this.resource-type OR
    contains(covers-platforms, this.covers-platforms[0])
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
