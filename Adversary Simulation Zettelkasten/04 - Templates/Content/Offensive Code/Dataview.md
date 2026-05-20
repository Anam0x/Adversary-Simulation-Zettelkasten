## Related Notes

### Same Classification

#### Similar Offensive Code
```dataview
LIST
FROM "03 - Content"
WHERE type = "Offensive Code"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    contains(languages, this.languages[0]) OR
    contains(platforms, this.platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Implements Tradecraft
```dataview
LIST WITHOUT ID implements-tradecraft
WHERE file = this.file
FLATTEN implements-tradecraft
SORT implements-tradecraft ASC
```

#### Targets Vulnerabilities
```dataview
LIST WITHOUT ID targets-vulnerabilities
WHERE file = this.file
FLATTEN targets-vulnerabilities
SORT targets-vulnerabilities ASC
```

#### Uses Protocols
```dataview
LIST WITHOUT ID uses-protocols
WHERE file = this.file
FLATTEN uses-protocols
SORT uses-protocols ASC
```
