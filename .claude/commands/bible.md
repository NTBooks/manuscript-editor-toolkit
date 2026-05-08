# Character & World Bible

Generate a persistent reference document cataloging all characters, locations, key objects, world rules, and the knowledge matrix across narrators. Output to `story_bible.md`.

This is a **catalog**, not an audit. It records what the manuscript establishes. For conflict detection, use `/continuity` instead.

## Steps

### 1. Check prerequisites

- `scene_index.md` must exist. If not, tell the user to run `/scene-guide` first.
- `chapters/` directory must exist. If not, tell the user to run `/split` first.

### 2. Write and run the extraction script

Write `bible_extract.mjs` to the project root:

```javascript
import { readFileSync, readdirSync, writeFileSync } from 'fs';
import { join } from 'path';

const indexPath = join(process.cwd(), 'scene_index.md');
const chaptersDir = join(process.cwd(), 'chapters');

let indexText = '';
try {
  indexText = readFileSync(indexPath, 'utf8');
} catch {
  console.error('scene_index.md not found. Run /scene-guide first.');
  process.exit(1);
}

// Detect parts dynamically from chapter files
const partBoundaries = [];
try {
  const chapterFiles = readdirSync(chaptersDir)
    .filter(f => /^chapter\d+\.(txt|md)$/.test(f))
    .sort((a, b) => parseInt(a.match(/\d+/)[0]) - parseInt(b.match(/\d+/)[0]));

  let currentPart = null;
  for (const file of chapterFiles) {
    const num = parseInt(file.match(/\d+/)[0]);
    const text = readFileSync(join(chaptersDir, file), 'utf8');
    const partMatch = text.match(/^(?:#{1,3}\s+)?PART\s+(ONE|TWO|THREE|FOUR|FIVE|SIX|SEVEN|EIGHT|NINE|TEN|\d+)\b.*$/im);
    if (partMatch) {
      if (currentPart) currentPart.last = num - 1;
      currentPart = { name: partMatch[0].replace(/^#{1,3}\s+/, '').trim(), first: num, last: num };
      partBoundaries.push(currentPart);
    }
    if (currentPart) currentPart.last = num;
  }
} catch {
  // No chapters dir or unreadable — parts will be empty
}

function getPart(num) {
  for (const p of partBoundaries) {
    if (num >= p.first && num <= p.last) return p.name;
  }
  return partBoundaries.length > 0 ? 'Unknown' : '';
}

// Parse chapters from scene_index
const chapterBlocks = indexText.split(/###\s+Chapter\s+(\d+)/);
const chapters = [];

for (let i = 1; i < chapterBlocks.length; i += 2) {
  const num = parseInt(chapterBlocks[i]);
  const block = chapterBlocks[i + 1] || '';

  const locMatch = block.match(/\*\*Location:\*\*\s*(.+?)(?=\s*\n)/);
  const charMatch = block.match(/\*\*Characters:\*\*\s*(.+?)(?=\s*\n)/);
  const summaryMatch = block.match(/\*\*Summary:\*\*\s*(.+?)(?=\n\*\*|\n---|\n###|$)/s);
  const revealsMatch = block.match(/\*\*Key reveals?:\*\*\s*(.+?)(?=\n\*\*|\n---|\n###|$)/s);
  const blindMatch = block.match(/\*\*Blind spot:\*\*\s*(.+?)(?=\n\*\*|\n---|\n###|$)/s);

  chapters.push({
    num,
    location: locMatch ? locMatch[1].trim() : '',
    characters: charMatch ? charMatch[1].trim() : '',
    summary: summaryMatch ? summaryMatch[1].trim() : '',
    reveals: revealsMatch ? revealsMatch[1].trim() : '',
    blindSpot: blindMatch ? blindMatch[1].trim() : ''
  });
}

// Extract unique character names from Characters fields
const charCounts = {};
const charFirstAppearance = {};
const charLastAppearance = {};
const charLocations = {};

for (const ch of chapters) {
  const names = ch.characters
    .split(/[;,]/)
    .map(s => s.replace(/\(.*?\)/g, '').trim())
    .filter(s => s.length > 0 && !s.match(/^(and|plus|others|various|crowd|background)$/i));

  for (let name of names) {
    name = name.replace(/^\s*(the|a|an)\s+/i, '').trim();
    const cleaned = name.split(/\s+/).filter(w => /^[A-Z]/.test(w)).join(' ');
    if (!cleaned || cleaned.length < 2) continue;

    if (!charCounts[cleaned]) {
      charCounts[cleaned] = 0;
      charFirstAppearance[cleaned] = ch.num;
      charLocations[cleaned] = new Set();
    }
    charCounts[cleaned]++;
    charLastAppearance[cleaned] = ch.num;
    if (ch.location) charLocations[cleaned].add(ch.location.split(/[—\-→;]/)[0].trim());
  }
}

// Extract unique locations
const locationChapters = {};
for (const ch of chapters) {
  const locs = ch.location.split(/[→;]/).map(s => s.trim()).filter(s => s.length > 0);
  for (const loc of locs) {
    const key = loc.replace(/\s*\(.*?\)\s*/g, '').trim();
    if (!locationChapters[key]) locationChapters[key] = [];
    locationChapters[key].push(ch.num);
  }
}

// Sort characters by appearance count (most frequent first)
const sortedChars = Object.entries(charCounts)
  .sort((a, b) => b[1] - a[1]);

// Build character appearance matrix
let out = '## Character Appearance Matrix\n\n';
out += '| Character | Appearances | First | Last | Parts |\n';
out += '|-----------|-------------|-------|------|-------|\n';

for (const [name, count] of sortedChars) {
  const first = charFirstAppearance[name];
  const last = charLastAppearance[name];
  const parts = new Set();
  for (const ch of chapters) {
    if (ch.characters.includes(name.split(' ')[0])) {
      const part = getPart(ch.num);
      if (part) parts.add(part);
    }
  }
  const partsStr = parts.size > 0 ? [...parts].join(', ') : '—';
  out += `| ${name} | ${count} | Ch ${first} | Ch ${last} | ${partsStr} |\n`;
}

// Build location catalog
out += '\n## Location Catalog\n\n';
out += '| Location | Chapters |\n';
out += '|----------|----------|\n';

const sortedLocs = Object.entries(locationChapters)
  .sort((a, b) => a[1][0] - b[1][0]);

for (const [loc, chs] of sortedLocs) {
  const ranges = [];
  let start = chs[0], prev = chs[0];
  for (let i = 1; i < chs.length; i++) {
    if (chs[i] === prev + 1) {
      prev = chs[i];
    } else {
      ranges.push(start === prev ? `${start}` : `${start}–${prev}`);
      start = chs[i];
      prev = chs[i];
    }
  }
  ranges.push(start === prev ? `${start}` : `${start}–${prev}`);
  out += `| ${loc} | ${ranges.join(', ')} |\n`;
}

console.log(out);

// Write JSON for Claude analysis
const jsonData = {
  partBoundaries,
  characters: sortedChars.map(([name, count]) => ({
    name,
    appearances: count,
    firstChapter: charFirstAppearance[name],
    lastChapter: charLastAppearance[name],
    locations: [...(charLocations[name] || [])]
  })),
  locations: sortedLocs.map(([loc, chs]) => ({ name: loc, chapters: chs })),
  chapters: chapters.map(c => ({
    num: c.num,
    location: c.location,
    characters: c.characters,
    summary: c.summary,
    reveals: c.reveals,
    blindSpot: c.blindSpot
  }))
};
writeFileSync(join(process.cwd(), 'bible_data.json'), JSON.stringify(jsonData, null, 2));
```

Run: `node bible_extract.mjs`

Save the script's stdout output — it becomes the appearance matrix and location catalog sections of the bible.

### 3. Classify characters

From the script output and scene_index.md, classify every character. Detect narrators/POV characters dynamically from scene_index.md (look for narrator/POV fields, or identify the character whose interiority is shown in each chapter). Then classify:

- **Narrators/POV characters** — characters who serve as the narrative lens (if multi-POV) or **Protagonist** (if single-POV)
- **Major** — appear in 5+ chapters or drive significant plot events
- **Supporting** — appear in 2-4 chapters with speaking roles or direct impact
- **Minor/mentioned** — appear once, or are referenced without appearing

### 4. Grep for physical and biographical details

Run targeted greps across `chapters/` for each **narrator/POV and major** character. Search for:

- **Physical appearance:** `[Name]` near "hair," "eyes," "tall," "short," "wore," "wearing," "looked like," "built," "skin," "face," "scar," "tattoo," "beard," "glasses"
- **Age/background:** `[Name]` near "old," "age," "born," "grew up," "school," "college," "graduated," "from"
- **Relationships:** `[Name]` near "wife," "husband," "married," "daughter," "son," "brother," "sister," "mother," "father," "friend," "partner," "ex"
- **Occupation:** `[Name]` near "work," "job," "firm," "company," common occupation words

Record chapter and line number for every detail found. Do NOT read full chapters — grep results plus scene_index summaries are sufficient.

For **supporting and minor** characters, grep only their name and record what the scene_index says about them. Skip the detailed physical grep.

### 5. Build character profiles

For each **narrator/POV** character, compile:

```markdown
### [Character Name]
**Role:** [narrator/POV character/major/supporting/minor]
**First appearance:** Chapter N
**Full name:** [if known]
**Occupation:** [from grep]
**Physical description:** [compiled from grep results with chapter citations]
**Key relationships:**
- [Relationship]: [details] (ch.N)
**Backstory:** [key facts from grep + scene_index]
**Arc:** [where they start → where they end, from scene_index summaries]
**Narrator blind spots:** [what this character consistently misses, from scene_index blind spots — omit if not applicable]
```

For **major** characters, use the same template but shorter. For **supporting/minor**, use a single-line entry.

### 6. Build the knowledge matrix (multi-POV only)

If the manuscript has multiple narrators/POV characters, compile a matrix from scene_index.md blind spots and reveals:

```markdown
## Knowledge Matrix

| Fact | [Narrator 1] Knows | [Narrator 2] Knows | ... | Reader Learns |
|------|---------------------|---------------------|-----|---------------|
| [key fact] | Yes/No (ch.N) | Yes/No (ch.N) | ... | Ch N |
```

Use dynamically detected narrator names as column headers. Focus on facts that create dramatic irony — things the reader knows from one narrator that another narrator is blind to.

If the manuscript is single-POV, skip this section entirely.

### 7. Catalog key objects and props

From scene_index reveals and summaries, catalog objects that recur or carry plot significance. Organize by category:

- **Vehicles**
- **Money/financial assets**
- **Documents/letters**
- **Technology/devices**
- **Other significant props**

For each: what it is, when introduced, who knows about it, its status at the end of each part (if the novel has parts).

### 8. Catalog world rules

From scene_index summaries and reveals, document the internal logic of the story world:

- **Systems/mechanics** — any invented or specialized systems central to the plot
- **Financial/economic rules** — amounts, debts, transactions that matter
- **Legal/institutional structures** — contracts, organizations, regulations
- **Real-world timeline anchors** — historical events, dates, technology that fixes the era
- **Geography** — key routes, distances, location relationships

Compile from scene_index only. Grep for specific terms only if the index is ambiguous.

### 9. Assemble the bible

Combine everything into `story_bible.md`:

```markdown
# Story Bible

## Overview
[2-3 sentences: genre, structure, core conflict, narrators/protagonist]

## Characters

### [Narrator/POV Characters or Protagonist]
[Full profiles]

### Major Characters
[Profiles for characters appearing 5+ chapters]

### Supporting Characters
[Shorter entries]

### Minor / Mentioned
[Single-line entries]

## Character Appearance Matrix
[Script-generated table]

## Knowledge Matrix
[Who knows what, with dramatic irony highlighted — multi-POV only]

## Key Objects & Props
[Cataloged items with status tracking]

## Locations
### [Location Name]
**Chapters:** [N, M, ...]
**Description:** [compiled from scene_index]
**Key events here:** [brief list]

## Location Catalog
[Script-generated table]

## World Rules
[Categories as applicable to this manuscript]

## Notes
- Generated from scene_index.md and targeted greps. No full chapter reads.
- Physical descriptions are cited with chapter numbers for verification.
- Knowledge matrix reflects what's explicitly stated, not implied.
```

### 10. Clean up

Delete `bible_data.json` and `bible_extract.mjs` after the report is assembled.

## Handling $ARGUMENTS

If the user provides arguments:
- A character name (e.g., `/bible Mary`) — generate only that character's full profile
- `characters` — only the characters section
- `locations` — only the locations section
- `knowledge` — only the knowledge matrix
- `objects` — only the objects/props catalog
- No arguments — full bible

$ARGUMENTS
