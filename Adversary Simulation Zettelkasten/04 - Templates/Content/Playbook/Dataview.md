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
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.required-tradecraft, file.link)
SORT file.name ASC
```

#### Required Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.required-tools, file.link)
SORT file.name ASC
```
