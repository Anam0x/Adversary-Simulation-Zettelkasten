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
