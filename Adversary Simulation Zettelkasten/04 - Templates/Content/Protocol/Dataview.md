## Related Notes

### Same Classification

#### Similar Protocols
```dataview
LIST
FROM "03 - Content"
WHERE type = "Protocol"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    protocol-family = this.protocol-family OR
    contains(authentication-methods, this.authentication-methods[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Abused By Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.abused-by-tradecraft, file.link)
SORT file.name ASC
```

#### Secured By Controls
```dataview
LIST
FROM "03 - Content"
WHERE type = "Security Control"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.secured-by-controls, file.link)
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
