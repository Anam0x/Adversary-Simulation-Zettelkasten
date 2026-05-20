## Related Notes

### Same Classification

#### Similar Attack Surfaces
```dataview
LIST
FROM "03 - Content"
WHERE type = "Attack Surface"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    contains(platforms, this.platforms[0]) OR
    contains(deployment-models, this.deployment-models[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Related Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.related-tradecraft, file.link)
SORT file.name ASC
```

#### Related Vulnerabilities
```dataview
LIST
FROM "03 - Content"
WHERE type = "Vulnerability"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.related-vulnerabilities, file.link)
SORT file.name ASC
```

#### Related Security Controls
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.related-controls, file.link)
SORT file.name ASC
```

#### Related Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.related-tools, file.link)
SORT file.name ASC
```
