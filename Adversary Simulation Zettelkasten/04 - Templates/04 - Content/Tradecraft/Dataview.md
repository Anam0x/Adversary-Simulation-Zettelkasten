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
LIST WITHOUT ID uses-tools
WHERE file = this.file
FLATTEN uses-tools
SORT uses-tools ASC
```

#### Bypasses Controls
```dataview
LIST WITHOUT ID bypasses-controls
WHERE file = this.file
FLATTEN bypasses-controls
SORT bypasses-controls ASC
```

#### Exploits Vulnerabilities
```dataview
LIST WITHOUT ID exploits-vulnerabilities
WHERE file = this.file
FLATTEN exploits-vulnerabilities
SORT exploits-vulnerabilities ASC
```

#### Uses Protocols
```dataview
LIST WITHOUT ID uses-protocols
WHERE file = this.file
FLATTEN uses-protocols
SORT uses-protocols ASC
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
