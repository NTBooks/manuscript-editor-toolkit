# Check Continuity

Audit the manuscript for continuity errors across chapters: character details, objects, settings, and facts that should remain consistent.

## What to Track

### Characters
- Physical descriptions (hair, height, scars, distinguishing features)
- Names and nicknames (consistent spelling, who uses which name)
- Relationships (family, friends, romantic, professional)
- Backstory facts (hometown, education, career, key life events)

### Objects and Props
- Items introduced should persist or be explicitly removed
- Vehicle details (make, model, color)
- Money amounts that should add up
- Clothing in continuous scenes

### Settings
- Room layouts and geography
- Distances between locations
- Building descriptions
- Weather consistency within the same timeframe

### Facts and Logic
- Rules of the story world applied consistently
- Information characters know vs. don't know (no acting on unlearned info)
- Cause-and-effect chains

## Process: Grep-First, Read-Second

### 1. Start with scene_index.md

Required. If it doesn't exist, tell the user to run `/scene-guide` first. Use it to identify which chapters share characters, locations, and objects.

### 2. Grep by category

Run targeted greps across `chapters/` before reading any files:

- **Character descriptions:** grep for character names near physical descriptors ("hair," "eyes," "tall," "short," "scar," etc.)
- **Names/nicknames:** grep for each character name and its variants to find inconsistent spelling
- **Vehicles:** grep for car makes, models, colors
- **Money:** grep for dollar amounts, "paid," "cost," "owed," "borrowed"
- **Locations:** grep for place names and addresses
- **Ages:** grep for character names near "old," "age," numbers

### 3. Targeted reads only for conflicts

When grep results show the same detail described differently in two chapters, read only those specific lines (with offset/limit) to verify the conflict in context.

### 4. Save the report

Write to `continuity_check.md`:

```markdown
# Continuity Audit

## Character Tracker
### [Character Name]
- [Detail]: [value] (ch.N, line X)
- [Detail]: [value] (ch.M, line Y) -- CONFLICT: [describe]

## Issues Found

### [Issue Title]
**Type:** character / object / setting / logic
**Chapters:** [N, M]
**Conflict:** [what doesn't match, with quotes from grep results]
**Severity:** minor / major
**Suggested fix:** [which version to keep and why — do not write replacement prose]

## Clean
[Categories checked with no issues found]
```

## Guidelines

- Grep results are your primary evidence. Only read chapter text to disambiguate.
- Minor inconsistencies deserve a note, not an alarm
- If an inconsistency might be intentional (unreliable narrator, character lying), note it but don't assume it's an error
- Suggested fixes describe the editorial change, not the new prose

$ARGUMENTS
