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
LIST WITHOUT ID practices-tradecraft
WHERE file = this.file
FLATTEN practices-tradecraft
SORT practices-tradecraft ASC
```

#### Uses Tools
```dataview
LIST WITHOUT ID uses-tools
WHERE file = this.file
FLATTEN uses-tools
SORT uses-tools ASC
```
