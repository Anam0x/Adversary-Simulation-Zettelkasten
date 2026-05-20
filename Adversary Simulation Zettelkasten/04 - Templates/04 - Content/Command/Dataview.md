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
LIST WITHOUT ID used-in-tradecraft
WHERE file = this.file
FLATTEN used-in-tradecraft
SORT used-in-tradecraft ASC
```

#### Used By Tools
```dataview
LIST WITHOUT ID used-by-tools
WHERE file = this.file
FLATTEN used-by-tools
SORT used-by-tools ASC
```
