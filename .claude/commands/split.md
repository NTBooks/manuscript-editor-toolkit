# Split Manuscript into Chapters

Split the manuscript into individual chapter files in the `chapters/` directory. The script is generated dynamically based on the manuscript's actual heading format.

## Steps

### 1. Check for manuscript source

Look for `novel.md` first (preferred), then fall back to `novel.txt`. If neither exists, tell the user and stop.

### 2. Detect chapter heading format

Read the first 150 lines of the manuscript file. Identify lines that look like chapter boundaries — headings or standalone lines that mark chapter divisions.

Common patterns to look for:

**Markdown headings with "Chapter":**
- `## Chapter One`, `# Chapter 1`, `### Chapter Twenty-Three — The Storm`

**Markdown headings with numbers only:**
- `## 1`, `## 2. The Storm`, `# 23`

**Markdown headings that are titles only (no "Chapter" or number prefix):**
- `## The Storm`, `## Into the Fire` (every h2 is a chapter)

**Plain text (novel.txt):**
- `CHAPTER ONE`, `Chapter 1`, `CHAPTER TWENTY-THREE`

**Other formats:**
- `## I.`, `## II.` (Roman numerals)
- `## Part One: Chapter 1`
- Anything else — look for the repeating pattern

Show the user 3-5 example headings you found and the regex you plan to use. Ask them to confirm before proceeding. If fewer than 2 chapter headings are detected, tell the user and ask them to describe their format.

### 3. Detect part headers (if any)

In the same sample, look for part-level divisions: lines like `## PART ONE`, `# PART TWO — Title`, or `PART 1`. If found, note the pattern. If not found, the script skips part handling.

### 4. Write the split script

Write `split_chapters.mjs` to the project root using the detected patterns. Use this template, replacing `CHAPTER_REGEX` and `PART_REGEX` with the detected patterns:

```javascript
import { createReadStream, existsSync } from 'fs';
import { createWriteStream, mkdirSync } from 'fs';
import { join } from 'path';

const chaptersDir = join(process.cwd(), 'chapters');
mkdirSync(chaptersDir, { recursive: true });

const mdFile = join(process.cwd(), 'novel.md');
const txtFile = join(process.cwd(), 'novel.txt');

let sourceFile;
let useMarkdown;

if (existsSync(mdFile)) {
    sourceFile = mdFile;
    useMarkdown = true;
} else if (existsSync(txtFile)) {
    sourceFile = txtFile;
    useMarkdown = false;
} else {
    console.error('Error: No manuscript found. Place novel.md or novel.txt in the project root.');
    process.exit(1);
}

console.log(`Using source: ${sourceFile}`);

const ext = useMarkdown ? '.md' : '.txt';
const stream = createReadStream(sourceFile, { encoding: 'utf8', highWaterMark: 64 * 1024 });

let chapterNum = 0;
let currentChapterPath = null;
let writeStream = null;
let buffer = '';
let pendingLines = [];

const CHAPTER_REGEX = /* REPLACE: regex matching chapter headings */;
const PART_REGEX = /* REPLACE: regex matching part headers, or null if no parts */;

function isPartHeader(trimmedLine) {
    return PART_REGEX ? PART_REGEX.test(trimmedLine) : false;
}

function isChapterHeader(trimmedLine) {
    return CHAPTER_REGEX.test(trimmedLine);
}

stream.on('data', (chunk) => {
    buffer += chunk;
    const lines = buffer.split('\n');

    if (!buffer.endsWith('\n')) {
        buffer = lines.pop();
    } else {
        buffer = '';
    }

    for (const line of lines) {
        const trimmedLine = line.trimEnd().replace(/\r$/, '').trim();

        if (isChapterHeader(trimmedLine)) {
            chapterNum += 1;
            const filename = `chapter${chapterNum}${ext}`;
            currentChapterPath = join(chaptersDir, filename);

            if (writeStream) writeStream.end();
            writeStream = createWriteStream(currentChapterPath);

            for (const pending of pendingLines) {
                writeStream.write(pending.replace(/\r$/, '') + '\n');
            }
            pendingLines = [];

            writeStream.write(line.replace(/\r$/, '') + '\n');
            continue;
        }

        if (isPartHeader(trimmedLine) || pendingLines.length > 0) {
            pendingLines.push(line);
            continue;
        }

        if (writeStream) {
            writeStream.write(line.replace(/\r$/, '') + '\n');
        }
    }
});

stream.on('end', () => {
    if (buffer && writeStream) writeStream.write(buffer.replace(/\r$/, '') + '\n');
    if (writeStream) writeStream.end();
    console.log(`Split complete. ${chapterNum} chapters written to ${chaptersDir}`);
});

stream.on('error', (err) => {
    console.error('Error reading file:', err);
    process.exit(1);
});
```

Replace `CHAPTER_REGEX` and `PART_REGEX` with the actual regex literals you detected. Examples:

- If headings are `## Chapter One`, `## Chapter Two`: use `/^#{1,3}\s+chapter\s+.+$/i`
- If headings are `## 1. The Storm`, `## 2. Into Fire`: use `/^#{1,3}\s+\d+[\.\s]/`
- If every `##` is a chapter: use `/^##\s+.+$/`
- If plain text `CHAPTER ONE`: use `/^chapter\s+.+$/i`
- If Roman numerals `## I.`, `## II.`: use `/^#{1,3}\s+[IVXLC]+\.?\s/`

The regex must NOT match part headers. If both patterns use markdown headings, make sure the chapter regex excludes part headers (e.g., use a negative lookahead or order the checks so part headers are caught first in the script logic, which the template already handles).

### 5. Run the script

```
node split_chapters.mjs
```

### 6. Verify and report

List the files created in `chapters/` and report:
- How many chapters were extracted
- Which source file was used
- The heading pattern that was detected
- Confirm the files are ready

### 7. Clean up

Delete `split_chapters.mjs` after a successful run.

## Notes

- The script prefers `novel.md` over `novel.txt`. When sourcing from markdown, chapter files are written as `.md`; from plain text, as `.txt`.
- Part headers are prepended to the first chapter of that part, not appended to the previous one.
- The script strips Windows-style carriage returns for cross-platform compatibility.
- The script uses streaming reads (64KB chunks) so it handles large manuscripts efficiently.
- If the detected pattern produces wrong results (too many or too few chapters), ask the user to show you a few chapter headings and adjust the regex.

$ARGUMENTS
