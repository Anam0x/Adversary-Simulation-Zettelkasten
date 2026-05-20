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
LIST WITHOUT ID related-tradecraft
WHERE file = this.file
FLATTEN related-tradecraft
SORT related-tradecraft ASC
```

#### Related Vulnerabilities
```dataview
LIST WITHOUT ID related-vulnerabilities
WHERE file = this.file
FLATTEN related-vulnerabilities
SORT related-vulnerabilities ASC
```

#### Related Security Controls
```dataview
LIST WITHOUT ID related-controls
WHERE file = this.file
FLATTEN related-controls
SORT related-controls ASC
```

#### Related Tools
```dataview
LIST WITHOUT ID related-tools
WHERE file = this.file
FLATTEN related-tools
SORT related-tools ASC
```
