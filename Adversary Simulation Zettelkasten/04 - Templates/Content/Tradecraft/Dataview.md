## Related Notes

### Same Classification

#### Same Tactic
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND tactic = this.tactic
SORT file.name ASC
LIMIT 10
```

#### Same Platforms
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND contains(platforms, this.platforms[0])
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Uses Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.uses-tools, file.link)
SORT file.name ASC
```

#### Bypasses Controls
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.bypasses-controls, file.link)
SORT file.name ASC
```

#### Exploits Vulnerabilities
```dataview
LIST
FROM "03 - Content"
WHERE type = "Vulnerability"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.exploits-vulnerabilities, file.link)
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

### Reverse Relationships

#### Implemented By Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(implements-tradecraft, this.file.link)
SORT file.name ASC
```

#### Detected By Controls
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(detects-tradecraft, this.file.link)
SORT file.name ASC
```
