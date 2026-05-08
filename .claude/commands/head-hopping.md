# Detect Head-Hopping

Audit the manuscript for point-of-view violations where the narration slips into a character's thoughts or knowledge that the current POV character couldn't access.

## What is head-hopping

Head-hopping occurs when the narration reveals the internal state (thoughts, feelings, motivations, sensory experiences) of a character who is NOT the current point-of-view character. In close third or first person, the narrator only has access to one character's interiority per scene or chapter.

Examples of head-hopping:
- POV character is Jake. "Sarah felt a chill run down her spine." (Jake can't know Sarah's physical sensations.)
- POV character is Jake. "Sarah wondered if he was lying." (Jake can't know what Sarah is wondering.)
- POV character is Jake. "Behind him, Sarah rolled her eyes." (Jake can't see behind himself unless established via mirror, reflection, etc.)

What is NOT head-hopping:
- "Sarah looked uncomfortable." (Observable behavior; Jake can see this.)
- "Sarah was probably lying, Jake thought." (Jake's inference, clearly attributed.)
- "Sarah said she felt sick." (Reported speech.)
- Omniscient narration that is consistently omniscient (not switching between close-limited and omniscient within a scene).

## Process

### 1. Read style_guide.md

Check the POV convention. If it says omniscient, this check is mostly irrelevant (but flag inconsistent distance shifts). If first person or close third, proceed.

### 2. Determine POV structure from scene_index.md

Read `scene_index.md` to identify:
- Which character is the POV/narrator for each chapter or scene
- Whether POV shifts happen within chapters (scene breaks) or only between chapters

If scene_index.md doesn't note POV, tell the user to update it or run `/scene-guide` with POV tracking.

### 3. Grep for interiority markers in non-POV characters

For each chapter, identify the POV character, then grep for non-POV character names appearing near interiority language:

**Thought/feeling verbs near non-POV names:**
- grep for non-POV character names within 5 words of: "felt," "thought," "wondered," "knew," "realized," "hoped," "feared," "wished," "believed," "remembered," "forgot," "noticed," "sensed," "decided"

**Internal sensation near non-POV names:**
- grep for non-POV names near: "heart," "stomach," "spine," "pulse," "breath," "chill," "flush," "nausea"

**Behind-the-back actions:**
- grep for "behind him," "behind her," "without [POV name] noticing," "unbeknownst"

### 4. Targeted reads for flagged lines

For each grep match, read only the surrounding 3-5 lines (use offset/limit) to check:
- Is this actually head-hopping, or is it the POV character's inference? ("She seemed angry" = fine. "She was angry" = head-hop.)
- Is there a dialog tag or attribution that makes it reported speech?
- Is this within a scene break where POV legitimately shifts?

### 5. Generate the report

Save to `head_hopping_check.md`:

```markdown
# Head-Hopping Audit

## POV Structure
[Summary: which character has POV in which chapters/scenes]

## Issues Found

### Chapter N, line X
**POV character:** [name]
**Violation:** "[quoted text]"
**Problem:** Narration accesses [non-POV character]'s [thoughts/feelings/unseen action]
**Severity:** clear violation / borderline (could be inference)

### Chapter M, line Y
...

## Borderline Cases
[Matches that might be intentional or might be the POV character's guess. List for author review.]

## Clean
[Chapters checked with no issues found]
```

### 6. Guidelines

- Borderline cases are common and often intentional. Flag them gently, don't alarm.
- "She looked angry" is observation. "She was angry" is head-hopping. The line between these is the whole job of this audit.
- First-person narrators get more leeway for confident assertions about others ("I could tell she was furious") since that's characterization of the narrator, not a POV break.
- If the manuscript uses omniscient narration consistently, note that and skip the detailed audit. Only flag if the distance shifts inconsistently (close-limited in one paragraph, omniscient in the next).

$ARGUMENTS
