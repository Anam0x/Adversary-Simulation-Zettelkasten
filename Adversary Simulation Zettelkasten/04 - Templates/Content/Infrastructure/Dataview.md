## Related Notes

### Same Classification

#### Similar Infrastructure
```dataview
LIST
FROM "03 - Content"
WHERE type = "Infrastructure"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    infrastructure-type = this.infrastructure-type OR
    contains(platforms, this.platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Supports Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.supports-tradecraft, file.link)
SORT file.name ASC
```

#### Supports Tools
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.supports-tools, file.link)
SORT file.name ASC
```

#### Supports Playbooks
```dataview
LIST
FROM "03 - Content"
WHERE type = "Playbook"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.supports-playbooks, file.link)
SORT file.name ASC
```
