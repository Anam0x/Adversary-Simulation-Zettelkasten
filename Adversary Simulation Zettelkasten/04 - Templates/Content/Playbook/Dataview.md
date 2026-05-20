## Related Notes

### Same Classification

#### Similar Playbooks
```dataview
LIST
FROM "03 - Content"
WHERE type = "Playbook"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    objective = this.objective OR
    scope = this.scope
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Required Tradecraft
```dataview
LIST WITHOUT ID required-tradecraft
WHERE file = this.file
FLATTEN required-tradecraft
SORT required-tradecraft ASC
```

#### Required Tools
```dataview
LIST WITHOUT ID required-tools
WHERE file = this.file
FLATTEN required-tools
SORT required-tools ASC
```
