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
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.implements-tradecraft, file.link)
SORT file.name ASC
```

#### Targets Vulnerabilities
```dataview
LIST
FROM "03 - Content"
WHERE type = "Vulnerability"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.targets-vulnerabilities, file.link)
SORT file.name ASC
```

#### Uses Protocols
```dataview
LIST
FROM "03 - Content"
WHERE type = "Protocol"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.uses-protocols, file.link)
SORT file.name ASC
```
