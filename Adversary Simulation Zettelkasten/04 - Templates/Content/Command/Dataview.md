## Related Notes

### Same Classification

#### Commands On Similar Platforms
```dataview
LIST
FROM "03 - Content"
WHERE type = "Command"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND contains(execution-environments, this.execution-environments[0])
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Used In Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.used-in-tradecraft, file.link)
SORT file.name ASC
```

#### Used By Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.used-by-tools, file.link)
SORT file.name ASC
```
