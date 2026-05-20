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
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.detects-tradecraft, file.link)
SORT file.name ASC
```

#### Known Bypasses
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.known-bypasses, file.link)
SORT file.name ASC
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
