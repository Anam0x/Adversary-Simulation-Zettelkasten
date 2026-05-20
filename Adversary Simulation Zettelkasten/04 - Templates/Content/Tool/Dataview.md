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
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.implements-tradecraft, file.link)
SORT file.name ASC
```

#### Used In Playbooks
```dataview
LIST
FROM "03 - Content"
WHERE type = "Playbook"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.used-in-playbooks, file.link)
SORT file.name ASC
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
