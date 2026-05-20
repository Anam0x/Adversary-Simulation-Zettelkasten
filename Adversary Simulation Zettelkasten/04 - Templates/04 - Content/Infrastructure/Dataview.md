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
LIST WITHOUT ID supports-tradecraft
WHERE file = this.file
FLATTEN supports-tradecraft
SORT supports-tradecraft ASC
```

#### Supports Tools
```dataview
LIST WITHOUT ID supports-tools
WHERE file = this.file
FLATTEN supports-tools
SORT supports-tools ASC
```

#### Supports Playbooks
```dataview
LIST WITHOUT ID supports-playbooks
WHERE file = this.file
FLATTEN supports-playbooks
SORT supports-playbooks ASC
```
