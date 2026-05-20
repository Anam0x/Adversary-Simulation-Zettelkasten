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
LIST WITHOUT ID detects-tradecraft
WHERE file = this.file
FLATTEN detects-tradecraft
SORT detects-tradecraft ASC
```

#### Known Bypasses
```dataview
LIST WITHOUT ID known-bypasses
WHERE file = this.file
FLATTEN known-bypasses
SORT known-bypasses ASC
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
