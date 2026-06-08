## Related Notes

### Same Classification

#### Similar Lab Setups
```dataview
LIST
FROM "03 - Content"
WHERE type = "Lab Setup"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    lab-purpose = this.lab-purpose OR
    difficulty = this.difficulty OR
    contains(platforms, this.platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Practices Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.practices-tradecraft, file.link)
SORT file.name ASC
```

#### Uses Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.uses-tools, file.link)
SORT file.name ASC
```
