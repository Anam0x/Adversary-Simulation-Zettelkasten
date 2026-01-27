---
aliases: 
tags:
  - 🥇Primary_Category
  - 🔴Red_Team
type: Primary Category
---
# [[Red Team]]

***

## Overview

Adversary simulation and full-scope security operations emulating real-world threat actors. Encompasses operational security, command and control, persistence, evasion techniques, and objective-based engagements.

---

## Statistics

```dataviewjs
const primaryLink = dv.current().file.link;
const secondaries = dv.pages('"02 - Secondary Categories"')
  .where(p => p.file.outlinks.includes(primaryLink));
const content = dv.pages('"03 - Content"')
  .where(p => p.file.outlinks.includes(primaryLink));

// Helper: Count only connections within 01/02/03 directories
const countKnowledgeLinks = (note) => {
  const knowledgeInlinks = note.file.inlinks.filter(l => 
    l.path.startsWith('01 - Primary Categories') ||
    l.path.startsWith('02 - Secondary Categories') ||
    l.path.startsWith('03 - Content')
  ).length;
  
  const knowledgeOutlinks = note.file.outlinks.filter(l => 
    l.path.startsWith('01 - Primary Categories') ||
    l.path.startsWith('02 - Secondary Categories') ||
    l.path.startsWith('03 - Content')
  ).length;
  
  return knowledgeInlinks + knowledgeOutlinks;
};

// Connectivity metrics
const totalInlinks = content.array().reduce((sum, p) => sum + p.file.inlinks.length, 0);
const totalOutlinks = content.array().reduce((sum, p) => sum + p.file.outlinks.length, 0);
const avgConnections = content.length > 0 ? ((totalInlinks + totalOutlinks) / content.length).toFixed(1) : '0';

// Find note with most incoming links
const mostIncoming = content.length > 0
  ? content.array().sort((a, b) => b.file.inlinks.length - a.file.inlinks.length)[0]
  : null;

// Find note with most outgoing links
const mostOutgoing = content.length > 0
  ? content.array().sort((a, b) => b.file.outlinks.length - a.file.outlinks.length)[0]
  : null;

// Isolated notes
const isolatedNotes = content.where(p =>
  p.file.inlinks.length === 0 ||
  (p.file.outlinks.length <= 2 && p.file.inlinks.length <= 1)
).length;

const types = {};
content.forEach(p => {
  const type = p.type || 'Uncategorized';
  types[type] = (types[type] || 0) + 1;
});
const topType = Object.entries(types).sort((a, b) => b[1] - a[1])[0];

const now = new Date();
const recentContent = content.where(p => {
  const daysSince = (now - p.file.mtime) / (1000 * 60 * 60 * 24);
  return daysSince <= 30;
}).length;

const oldContent = content.where(p => {
  const daysSince = (now - p.file.mtime) / (1000 * 60 * 60 * 24);
  return daysSince > 180;
}).length;

const contentPerSecondary = secondaries.length > 0 ? (content.length / secondaries.length).toFixed(1) : '0';
const mostPopulatedSecondary = secondaries.array()
  .map(s => ({
    name: s.file.name,
    count: content.where(c => c.file.outlinks.map(l => l.path).includes(s.file.path)).length
  }))
  .sort((a, b) => b.count - a.count)[0];

const stubNotes = content.where(p => p.file.size < 500).length;
const wellDeveloped = content.where(p => p.file.size > 2000).length;

// Health Score
const contentScore = Math.min((content.length / 50) * 100, 100);
const avgConn = content.length > 0 
  ? content.array().reduce((sum, p) => sum + p.file.inlinks.length + p.file.outlinks.length, 0) / content.length 
  : 0;
const connectivityScore = Math.min((avgConn / 10) * 100, 100);
const freshnessScore = content.length > 0 ? (recentContent / content.length) * 100 : 0;
const completenessScore = content.length > 0 ? ((content.length - stubNotes) / content.length) * 100 : 0;
const overallScore = ((contentScore + connectivityScore + freshnessScore + completenessScore) / 4).toFixed(0);

const getHealthEmoji = (score) => {
  if (score >= 80) return '💚';
  if (score >= 60) return '💛';
  if (score >= 40) return '🧡';
  return '❤️';
};

const getProgressBar = (score) => {
  const filled = Math.round(score / 10);
  return '█'.repeat(filled) + '░'.repeat(10 - filled);
};

dv.header(3, "💚 Category Health");
dv.paragraph(`**Overall Score**: ${getHealthEmoji(overallScore)} **${overallScore}/100**`);
dv.paragraph(`
- Content Volume: ${getProgressBar(contentScore)} ${contentScore.toFixed(0)}/100
- Connectivity: ${getProgressBar(connectivityScore)} ${connectivityScore.toFixed(0)}/100
- Freshness: ${getProgressBar(freshnessScore)} ${freshnessScore.toFixed(0)}/100
- Completeness: ${getProgressBar(completenessScore)} ${completenessScore.toFixed(0)}/100
`);

dv.header(3, "📊 Overview");
dv.paragraph(`
- **Secondary Categories**: ${secondaries.length}
- **Content Notes**: ${content.length}
- **Content Types**: ${Object.keys(types).length}
- **Avg Content per Secondary**: ${contentPerSecondary}
- **Most Populated Secondary**: ${mostPopulatedSecondary ? mostPopulatedSecondary.name : 'N/A'} (${mostPopulatedSecondary ? mostPopulatedSecondary.count : 0})
`);

dv.header(3, "🔗 Connectivity");
dv.paragraph(`
- **Incoming Links**: ${totalInlinks}
- **Outgoing Links**: ${totalOutlinks}
- **Avg Connections per Note**: ${avgConnections}
- **Most Incoming Links**: ${mostIncoming ? mostIncoming.file.link : 'N/A'} (${mostIncoming ? mostIncoming.file.inlinks.length : 0})
- **Most Outgoing Links**: ${mostOutgoing ? mostOutgoing.file.link : 'N/A'} (${mostOutgoing ? mostOutgoing.file.outlinks.length : 0})
- **Isolated Notes**: ${isolatedNotes} (${content.length > 0 ? ((isolatedNotes / content.length) * 100).toFixed(0) : 0}%)
`);

dv.header(3, "📒 Content");
dv.paragraph(`
- **Top Type**: ${topType ? topType[0] : 'N/A'} (${topType ? topType[1] : 0})
- **Well-Developed**: ${wellDeveloped} (${content.length > 0 ? ((wellDeveloped / content.length) * 100).toFixed(0) : 0}%)
- **Stubs**: ${stubNotes} (${content.length > 0 ? ((stubNotes / content.length) * 100).toFixed(0) : 0}%)
`);

dv.header(3, "⏰ Activity Metrics");
dv.paragraph(`
- **Updated in Last 30 Days**: ${recentContent} (${content.length > 0 ? ((recentContent / content.length) * 100).toFixed(0) : 0}%)
- **Not Updated in 6+ Months**: ${oldContent} (${content.length > 0 ? ((oldContent / content.length) * 100).toFixed(0) : 0}%)
- **Last Activity**: ${content.length > 0 ? content.array().sort((a, b) => b.file.mtime - a.file.mtime)[0].file.mtime.toFormat('MMM dd, yyyy') : 'N/A'}
`);
```

### Top 5 Most Related Primary Categories
```dataviewjs
const currentPrimary = dv.current().file.link;

const mySecondaries = dv.pages('"02 - Secondary Categories"')
  .where(p => p.file.outlinks.includes(currentPrimary))
  .map(p => p.file.link);

const relatedPrimaries = dv.pages('"01 - Primary Categories"')
  .where(p => p.file.name != dv.current().file.name)
  .map(p => {
    const sharedSecondaries = dv.pages('"02 - Secondary Categories"')
      .where(s => s.file.outlinks.includes(p.file.link) && 
                  mySecondaries.some(ms => ms.path === s.file.path))
      .length;
    return {link: p.file.link, shared: sharedSecondaries};
  })
  .filter(p => p.shared > 0)
  .sort(p => p.shared, 'desc')
  .limit(5);

if (relatedPrimaries.length > 0) {
  dv.table(
    ["Related Primary Category", "Shared Secondary Category Count"],
    relatedPrimaries.map(p => [p.link, p.shared])
  );
} else {
  dv.paragraph("*No related primary categories found.*");
}
```

---

## Secondary Categories

> [!seealso]- Click to Expand
> ```dataview
>  LIST
>  FROM "02 - Secondary Categories"
>  WHERE contains(primary-categories, this.file.link) OR contains(parents, this.file.link)
>  SORT file.name ASC 
> ```

### Recent Activity
```dataview
TABLE WITHOUT ID
  file.link AS "Secondary Category",
  file.mtime AS "Modified"
FROM "02 - Secondary Categories"
WHERE contains(file.outlinks, this.file.link)
SORT file.mtime DESC
LIMIT 10
```

### Most Connected
```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  length(file.outlinks) AS "→ Links Out",
  length(file.inlinks) AS "← Links In",
  (length(file.outlinks) + length(file.inlinks)) AS "Total"
FROM "02 - Secondary Categories"
WHERE contains(file.outlinks, this.file.link)
SORT file.name ASC
```

---

## Content Notes

> [!seealso]- Click to Expand
> ```dataview
>  LIST
>  FROM "03 - Content"
>  WHERE contains(primary-categories, this.file.link) OR contains(parents, this.file.link)
>  SORT file.name ASC 
> ```

### Recent Activity
```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  type AS "Content Type",
  file.mtime AS "Modified"
FROM "03 - Content"
WHERE contains(file.outlinks, this.file.link)
SORT file.mtime DESC
LIMIT 10
```

### Most Connected
```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  type AS "Type",
  length(file.outlinks) AS "→ Links Out",
  length(file.inlinks) AS "← Links In",
  (length(file.outlinks) + length(file.inlinks)) AS "Total"
FROM "03 - Content"
WHERE contains(file.outlinks, this.file.link)
SORT (length(file.outlinks) + length(file.inlinks)) DESC
LIMIT 10
```

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
