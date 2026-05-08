# Manuscript Editor Toolkit

A set of 14 Claude Code slash commands that turn Claude into a professional manuscript editor. Claude analyzes, advises, and organizes — it never writes prose for you. The author writes; Claude edits.

> **Every word of prose is written by the human author.** This toolkit uses AI strictly as an editorial and organizational tool — never as a ghostwriter. The system is designed at every level to prevent AI from generating narrative text. See the full [AI Use Disclosure & Ethics](#ai-use-disclosure--ethics) section below.

## What You Get

| Command | What it does |
|---------|-------------|
| `/split` | Split manuscript into individual chapter files |
| `/recombine` | Rebuild full manuscript + Word-ready export |
| `/wordcount` | Chapter-by-chapter word count with visual charts |
| `/scene-guide` | Generate a scene-by-scene navigation index |
| `/summarize` | Chapter-by-chapter plot summary (spoils ending) |
| `/find` | Locate scenes, quotes, or patterns in the manuscript |
| `/propose` | Create editorial proposals (describes changes, never writes prose) |
| `/query-letter` | Generate query letter + synopsis for agent submissions |
| `/style-check` | Audit manuscript against your style guide |
| `/chronology` | Audit timeline consistency |
| `/continuity` | Audit character/object/setting consistency |
| `/head-hopping` | Detect POV violations |
| `/pacing` | Analyze pacing, beat structure, tension arcs, scene types |
| `/bible` | Generate character/world bible with profiles, knowledge matrix |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI, desktop app, or IDE extension)
- [Node.js](https://nodejs.org/) (v18+ recommended, for the script-based commands)

## Installation

1. Copy these files into your manuscript project root:

```
your-novel-project/
├── .claude/
│   └── commands/       ← copy the entire commands/ directory
├── CLAUDE.md           ← copy this file
└── style_guide.md      ← copy this file
```

2. Place your manuscript in the project root as `novel.md` (preferred) or `novel.txt`. See [Preparing Your Manuscript](#preparing-your-manuscript) below.

3. Open Claude Code in the project directory. The slash commands are available immediately.

## Getting Started

The recommended workflow:

1. **Place your manuscript** as `novel.md` in the project root
2. **`/split`** — splits into individual chapter files in `chapters/`
3. **`/scene-guide`** — generates `scene_index.md` (used by most other commands)
4. **`/style-check`** or **`/wordcount`** — quick first passes
5. Use any other command as needed

Most commands depend on `scene_index.md` existing. Run `/scene-guide` early.

## Preparing Your Manuscript

The toolkit reads your manuscript as `novel.md` (preferred) or `novel.txt`. Most authors write in Word — here's how to convert.

### Converting from Word to Markdown

The easiest method is to use a markdown editor like [Obsidian](https://obsidian.md/) as a conversion tool:

1. **Open this project folder as a vault in Obsidian** (File > Open Vault > Open folder as vault). Obsidian will create a `.obsidian/` folder which is already gitignored.

2. **Create a new file** called `novel.md` inside the vault.

3. **Copy your entire manuscript from Word** (Ctrl+A, Ctrl+C in Word) and **paste it into `novel.md`** (Ctrl+V in Obsidian). Obsidian automatically converts Word formatting to markdown:
   - Word headings (Heading 1, Heading 2, etc.) become markdown headings (`#`, `##`, etc.)
   - Bold and italic carry over
   - Most other formatting translates cleanly

4. **Check your chapter headings.** The `/split` command samples your manuscript's headings and builds a splitter script matched to your format. Common formats that work out of the box:
   - `## Chapter One`, `## Chapter 1`, `# Chapter Twenty-Three — The Storm`
   - `## 1. The Storm`, `## 2. Into Fire` (numbered with titles)
   - `## The Storm`, `## Into Fire` (title-only headings)
   - `## I.`, `## II.` (Roman numerals)
   
   Any heading level (`#`, `##`, or `###`) works. `/split` will show you the pattern it detected and ask you to confirm before proceeding.

5. **Check your part headers** (if applicable). Part divisions should be headings containing "PART" followed by a number or word — for example:
   - `## PART ONE`
   - `# Part Two — Into the Fire`

### Alternative: Plain Text

If you prefer not to use markdown, save your manuscript from Word as a plain text file (File > Save As > Plain Text) named `novel.txt`. In plain text mode, the splitter looks for lines like `Chapter 1` or `CHAPTER ONE` without requiring heading markers.

### Other Markdown Editors

Any markdown editor works — [Typora](https://typora.io/), [iA Writer](https://ia.net/writer), VS Code with a markdown preview, or even a plain text editor. The key requirement is that chapter headings use markdown heading syntax (`#`, `##`, or `###`). The `/split` command detects your specific format automatically.

### Node.js Requirement

Several commands (`/split`, `/wordcount`, `/recombine`, `/pacing`, `/bible`) run Node.js scripts for mechanical work like splitting files and counting words. Install [Node.js](https://nodejs.org/) v18 or later if you don't already have it. You can check with:

```
node --version
```

## Customizing the Style Guide

Edit `style_guide.md` to match your preferences. The template has `[author to fill in]` placeholders for:

- Punctuation rules (em dashes, semicolons, ellipses)
- Dialog conventions (tags, action beats, internal thought)
- POV and tense
- Formatting (scene breaks, chapter headings, emphasis)
- Banned words and preferred spellings

The `/style-check` command reads this file and greps for violations.

## Making It Yours

Every command ships with sensible defaults, but defaults are just starting points. If a command's conventions don't fit your novel, tell Claude what you want instead. Some examples:

- **Genre conventions** — "My novel is a cozy mystery; when checking pacing, don't flag quiet domestic scenes as low-tension"
- **POV rules** — "I write omniscient third person, so `/head-hopping` should only flag mid-paragraph POV shifts, not scene-level ones"
- **Structure** — "My chapters don't have titles, just numbered parts like `PART 1: 1`, `PART 1: 2`"
- **Continuity focus** — "Track magic system rules as a continuity concern, not just character appearances"
- **Proposal style** — "When proposing edits, organize by character arc rather than chapter order"
- **Output format** — "I want the pacing report as a table, not a narrative summary"

You can customize at whatever level fits:

- **On the fly** — just tell Claude what you want during any command
- **Persistently in `CLAUDE.md`** — add a `## My Preferences` section so your rules apply every session
- **Edit the commands directly** — the slash commands are markdown files in `.claude/commands/`. Open any of them, read how it works, and change whatever you like — the output format, the criteria, the analysis steps, all of it

## How It Works

The toolkit follows strict **token economy** principles:

- **Scene index first** — `scene_index.md` answers most questions without reading chapters
- **Grep second** — search for specific terms before loading any file
- **Targeted reads last** — only load the specific chapter passage needed
- **Scripts for mechanical work** — word counting, chapter splitting, and metrics run via Node.js scripts, not through the conversation

This means Claude can work with novel-length manuscripts without running into context limits.

## AI Use Disclosure & Ethics

Every word of prose in a manuscript edited with this toolkit was written by a human. AI is used as an editorial and organizational tool, never as a ghostwriter. The system is specifically and repeatedly designed to prevent AI from generating any narrative text.

### What the AI Does

**Mechanical / scripted tasks (no AI judgment involved):**
These are performed by JavaScript scripts that the AI executes. The AI doesn't read or interpret the manuscript for these — it runs code.

- **Word counting** — A script reads chapter files and outputs word counts, averages, and outlier detection. Pure math.
- **File splitting** — A script splits the single manuscript file into chapter files using regex pattern matching. Pure text processing.
- **File recombination** — A script reassembles chapter files into a single manuscript and a Word-ready format. Pure concatenation.

**Search & navigation (read-only):**

- **Find** — Locates specific passages, quotes, or scenes using text search (grep). Reports chapter and line numbers. Does not modify anything.
- **Style check** — Searches for punctuation and word-level violations defined by the author's own style guide (e.g., banned words). Pattern matching, not rewriting.
- **Head-hopping detection** — Searches for point-of-view violations using keyword proximity matching. Reports locations for the author to review.

**Analytical tasks (AI reads and summarizes, does not generate prose):**

- **Scene index** — Reads chapters in batches and creates a factual, navigational index: location, characters present, what happens. Not narrative prose.
- **Book summary** — Builds a plot summary for use in query letters. Written in professional summary language, not the author's narrative voice.
- **Pacing analysis** — Combines script-generated metrics with scene classifications to produce structural analysis. Outputs tables, charts, and beat mapping.
- **Chronology audit** — Checks timeline consistency by cross-referencing temporal markers across chapters. Reports contradictions.
- **Continuity audit** — Checks for inconsistencies in character descriptions, object details, and setting facts. Reports conflicts.
- **Story bible** — Catalogs characters, locations, objects, and a knowledge matrix. A reference document, not narrative.

**Editorial proposals (AI advises, does not write):**

- **Propose** — The AI writes a proposal document that diagnoses an issue, explains why it matters, and describes what should change. The proposal explicitly does not contain replacement prose. The author writes all revisions.

**Business writing (AI generates non-manuscript text):**

- **Query letter & synopsis** — Generates pitch materials for literary agent submissions. This is the only command where the AI produces extended original text, and it is professional business correspondence *about* the book, not part of the book itself.

### What the AI Is Forbidden From Doing

These prohibitions are hard-coded into the system's foundational instructions (`CLAUDE.md`) and reinforced in every individual command:

1. **Never generate replacement prose, new dialog, or narrative text.**
2. **Never rewrite passages "in the author's voice."** The AI is not trained on or imitating the author's style for generative purposes.
3. **Never edit chapter files or the manuscript directly.** All editorial output goes to separate proposal files. The manuscript is untouched by AI.
4. **When proposing changes, describe WHAT should change and WHY — not what the new words should be.** The AI operates like a traditional editor writing margin notes, not a co-writer.

### In Practice

| Task | Who Does It | AI Role |
|------|-------------|---------|
| Writing prose | Author | None |
| Writing dialog | Author | None |
| Revising prose | Author | Describes what to change, never how |
| Deciding what to change | Author | Proposes options, author decides |
| Checking consistency | AI | Reports findings for author review |
| Counting words | Script | Automated, no AI judgment |
| Formatting for submission | Script | Automated, no AI judgment |
| Writing query letters | AI | Business correspondence about the book |
| Plot/character decisions | Author | AI may surface questions, author decides |

### The Analogy

This system is the equivalent of hiring a freelance editor who reads your manuscript and takes notes, flags inconsistencies and timeline errors, writes editorial memos about what could be stronger and why, helps you prepare submission materials — and never picks up a pen and writes a sentence of your book. The difference is that this editor runs on a GPU instead of coffee. The editorial relationship — author writes, editor advises — is identical to traditional publishing workflows.

### Technical Transparency

- **AI model:** Claude (Anthropic)
- **Interface:** Claude Code with custom command definitions
- **All command definitions** are plain-text markdown files stored in `.claude/commands/` and are available for inspection
- **The system instructions** (`CLAUDE.md`) that govern AI behavior are stored in the project root and are available for inspection
- **No fine-tuning** was performed on the author's writing. The AI has no special training on any manuscript or author's style
- **No AI-generated text exists in the manuscript.** The only AI-generated text files in the project are reports (word counts, pacing analysis, continuity checks), reference documents (scene index, story bible), and submission materials (query letter, synopsis)

## File Structure After Use

```
your-novel-project/
├── .claude/commands/        # Toolkit commands (committed)
├── chapters/                # Split chapter files (gitignored)
├── proposals/               # Editorial proposals (gitignored)
├── CLAUDE.md                # Editor instructions (committed)
├── style_guide.md           # Your style preferences (committed)
├── novel.md                 # Your manuscript (gitignored)
├── scene_index.md           # Generated scene index (gitignored)
├── story_bible.md           # Generated character bible (gitignored)
├── wordcount_report.md      # Generated word count (gitignored)
├── pacing_report.md         # Generated pacing analysis (gitignored)
├── chronology_check.md      # Generated timeline audit (gitignored)
├── continuity_check.md      # Generated continuity audit (gitignored)
├── style_check.md           # Generated style audit (gitignored)
└── book_summary.md          # Generated plot summary (gitignored)
```

The `.gitignore` included with the toolkit prevents your manuscript and generated reports from being committed to version control.

## License

MIT
