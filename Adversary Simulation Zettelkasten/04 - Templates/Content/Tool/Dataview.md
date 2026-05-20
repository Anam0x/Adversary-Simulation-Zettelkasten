## Related Notes

### Same Classification

#### Tools In Same Category
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tool"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    tool-category = this.tool-category OR
    contains(operating-platforms, this.operating-platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Implements Tradecraft
```dataview
LIST WITHOUT ID implements-tradecraft
WHERE file = this.file
FLATTEN implements-tradecraft
SORT implements-tradecraft ASC
```

#### Used In Playbooks
```dataview
LIST WITHOUT ID used-in-playbooks
WHERE file = this.file
FLATTEN used-in-playbooks
SORT used-in-playbooks ASC
```

### Reverse Relationships

#### Referenced By Other Notes
```dataview
LIST
FROM "03 - Content"
WHERE file.name != this.file.name
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND (
    contains(required-tools, this.file.link) OR
    contains(uses-tools, this.file.link) OR
    contains(related-tools, this.file.link) OR
    contains(associated-tools, this.file.link)
  )
SORT file.name ASC
```
