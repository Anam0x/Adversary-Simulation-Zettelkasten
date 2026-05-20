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
LIST WITHOUT ID abused-by-tradecraft
WHERE file = this.file
FLATTEN abused-by-tradecraft
SORT abused-by-tradecraft ASC
```

#### Secured By Controls
```dataview
LIST WITHOUT ID secured-by-controls
WHERE file = this.file
FLATTEN secured-by-controls
SORT secured-by-controls ASC
```

#### Related Tools
```dataview
LIST WITHOUT ID related-tools
WHERE file = this.file
FLATTEN related-tools
SORT related-tools ASC
```
