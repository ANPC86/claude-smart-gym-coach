# Fitness Coach — Notion Workspace Reference
> **Template version** — replace all page/database IDs with your own before use.
> **Last updated**: 2026-03-30
> If Claude's memory conflicts with this file, prefer whichever has the more recent date.

Use this reference when creating, updating, or querying Notion. The connector is available in all conversations — just call the Notion tools directly once connected.

---

## How to Find Your Notion IDs

- **Page ID**: Open a page in Notion → share → copy link. The UUID is the string after the last `/` (format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
- **Database ID**: Open a database as a full page → copy link. Same UUID format.
- **Data Source ID**: Use `notion-fetch` on the database page → look for `collection://` reference in the response

---

## Hub Page

- **Hub**: `YOUR_HUB_PAGE_ID`

---

## Projects Database (optional)

If you use a central Projects database to track fitness initiatives alongside other work:

- **Database ID**: `YOUR_PROJECTS_DB_ID`
- **Data Source**: `YOUR_PROJECTS_DATA_SOURCE`

Suggested schema:

| Property | Type | Options |
|----------|------|---------|
| Project | Title | — |
| Persona | Multi-select | e.g. `Fitness Coach`, `Other Personas` |
| Status | Select | `Active`, `Planned`, `On Hold`, `Done` |
| Priority | Select | `P0 - Critical`, `P1 - High`, `P2 - Medium`, `P3 - Low` |
| Area | Multi-select | e.g. `Fitness`, `Analytics`, `Finance` |
| Target Date | Date | — |
| Notes | Rich text | — |

---

## Fitness Coach — Key Pages

| Page | Placeholder ID | Purpose |
|------|----------------|---------|
| 💪 Fitness Coach (landing) | `YOUR_LANDING_PAGE_ID` | Main dashboard; current session content goes here |
| 📝 Current Workout Notes | `YOUR_WORKOUT_NOTES_PAGE_ID` | Mid-session notes — weight adjustments, form feels, fatigue flags |
| 📚 Reference | `YOUR_REFERENCE_PAGE_ID` | Static reference docs |
| 🗂️ AI System Changelog | `YOUR_CHANGELOG_PAGE_ID` | Skill file change log |

---

### 📝 Current Workout Notes — Usage

This page is the **primary session reference** — opened on tablet/phone during the workout.

**Structure:**
- `🚩 Carry-Forward Flags` section at top — constraints from last reviewed session
- `## Phase 1–5` — phase headers with `### Exercise Name` blocks
- Each exercise: ID in backticks, weight/sets/mode, cues, `💬 **Your notes:**` prompt

**Writing during workout design:**
```
Tool: Notion:notion-update-page
Command: replace_content
page_id: "YOUR_WORKOUT_NOTES_PAGE_ID"
```

**Reading during session review (Step 0):**
```
Tool: Notion:notion-fetch
id: "YOUR_WORKOUT_NOTES_PAGE_ID"
```

---

### 💪 Fitness Coach Landing Page — Usage Pattern

Keep lean — dashboard only, not a history log.

**Current session block:**
```markdown
### 💪 Current Session — [Date]
**[Workout Name]** · ~[X] min · [Cardio] · [Focus]
**Goals:** [Goals line]
📋 Full walkthrough + coaching cues → [Current Workout Notes](YOUR_WORKOUT_NOTES_URL)

> 💡 [Key coaching note]

---
```

**Update pattern:**
```
Tool: Notion:notion-update-page
Command: update_content
page_id: "YOUR_LANDING_PAGE_ID"
content_updates: [{ "old_str": "### 💪 Current Session...", "new_str": "### 💪 Current Session — [new date]..." }]
```

> Always fetch the page first to get exact `old_str` content.

---

## Fitness Coach — Databases

### Training Journal

- **Database ID**: `YOUR_TRAINING_JOURNAL_DB_ID`
- **Data Source**: `YOUR_TRAINING_JOURNAL_DATA_SOURCE`

| Property | Type | Options |
|----------|------|---------|
| Session | Title | — |
| Session Date | Date | — |
| Focus | Multi-select | `Chest`, `Back`, `Shoulders`, `Arms`, `Core`, `Legs`, `Glutes`, `Full Body`, `Cardio`, `Mobility` |
| Duration | Number | minutes |
| Exercises | Number | count |
| Total Sets | Number | — |
| Coach Notes | Rich text | AI coach review summary |
| My Notes | Rich text | Personal observations |
| PRs | Rich text | New personal records |
| Recovery Rating | Select | `Fresh`, `Moderate`, `Fatigued`, `Overtrained` |

#### Creating a Training Journal entry

```
Tool: Notion:notion-create-pages
Parent: { "data_source_id": "YOUR_TRAINING_JOURNAL_DATA_SOURCE" }
Properties:
  Session: "B STR-BACK DAY 1/1 v1"
  "date:Session Date:start": "2026-01-01"
  "date:Session Date:is_datetime": 0
  Focus: "[\"Back\", \"Core\"]"
  Duration: 75
  Exercises: 18
  Total Sets: 42
  Coach Notes: "PLANNED — not yet completed"
Content:
  Full walkthrough in iPad-friendly markdown
```

---

### Exercise Library ← Source of truth for exercise history

- **Database URL**: `YOUR_EXERCISE_LIBRARY_PAGE_URL`
- **Data Source**: `YOUR_EXERCISE_LIBRARY_DATA_SOURCE`

> **Important**: This database is the authoritative source for last weight, scores, and discovered status. It replaces the flat JSON tracker file used in older versions of this setup.

| Property | Type | Notes |
|----------|------|-------|
| Exercise | Title | Exercise name |
| Library ID | Text | `actionLibraryGroupId` as string — exact match required |
| Category | Select | `Training`, `Warm up`, `Body-weight`, `Stretch`, `Row&Ski`, `Rehab`, `HIIT`, `Pilates` |
| Muscle Group | Select | Primary muscle |
| Accessories | Text | Comma-separated IDs (1=Bench, 2=Rope, 3=Straps, 4=Bar, 5=Handles, 7=Ski, 8=Row, 9=AdjBench, 10=Belt) |
| outPosition | Number | 0–10; workout ordering position |
| Discovered | Checkbox | `__YES__` = appeared in at least one session record |
| First Seen | Date | Date of first session appearance |
| Last Seen | Date | Date of most recent session appearance |
| Last Weight | Number | Last working weight in lbs |
| Last Amp | Number | Last amplitudeStableScore (1–5) |
| Last Force | Number | Last forceControlScore (1–5) |
| Last Bal | Number | Last bilateralBalanceScore (1–5); null for unilateral alternating sets |
| Times Seen | Number | Count of session records containing this exercise; increment by 1 on each session update, set to 1 on first discovery. If null when fetched, treat as 0 before incrementing. |
| Recommended Stretches | Rich text | Pre-exercise mobility work for this movement; format: stretch name · technique note · Library ID(s) · duration cue. Populated from session experience; see `/prompts/recommended-stretches-bulk-populate.md` for bulk pre-population. |
| Notes | Text | Coach notes, technique flags, weight anchors, exclusion flags |

#### Finding an exercise by title

```
Tool: Notion:notion-search
query: "Exercise Title Here"
data_source_url: "YOUR_EXERCISE_LIBRARY_DATA_SOURCE"
page_size: 3
→ match first result by title
→ notion-fetch the page URL to get full properties
```

#### Checking if an exercise is discovered

```
notion-fetch the page → check Discovered property
"__YES__" = discovered | "__NO__" or null = undiscovered
```

#### Updating an existing exercise after a session

```
Tool: Notion:notion-update-page
command: update_properties
page_id: [from search result]
properties:
  Discovered: "__YES__"
  Last Weight: [integer, lbs]
  Last Amp: [1–5]
  Last Force: [1–5]
  Last Bal: [1–5, omit if unilateral alternating]
  Times Seen: [fetched Times Seen + 1; if null, set to 1]
  "date:Last Seen:start": "YYYY-MM-DD"
  "date:Last Seen:is_datetime": 0
  # Set First Seen only if currently null:
  "date:First Seen:start": "YYYY-MM-DD"
  "date:First Seen:is_datetime": 0
```

#### Creating a new exercise entry (first-time discovery)

```
Tool: Notion:notion-create-pages
Parent: { "data_source_id": "YOUR_EXERCISE_LIBRARY_DATA_SOURCE" }
Properties:
  Exercise: "Exercise Title"
  Library ID: "123456789"          ← string, from actionLibraryGroupId
  Category: "Training"
  Muscle Group: "Biceps"
  Accessories: "5,1"
  outPosition: 9
  Discovered: "__YES__"
  Last Weight: 20
  Last Amp: 3
  Last Force: 4
  Last Bal: 3
  Times Seen: 1
  "date:Last Seen:start": "YYYY-MM-DD"
  "date:Last Seen:is_datetime": 0
  "date:First Seen:start": "YYYY-MM-DD"
  "date:First Seen:is_datetime": 0
  Notes: "First seen YYYY-MM-DD."
```

> **Note**: Batch via `notion-create-pages` (up to 100 per call) when creating multiple new discoveries from the same session.

#### Adding or updating Recommended Stretches

```
Tool: Notion:notion-update-page
command: update_properties
page_id: [exercise page ID]
properties:
  Recommended Stretches: "Stretch Name · technique note (e.g. hold cable/frame for support) · Library ID: 123456789 · 30s/side"
```

> Use semicolons to separate multiple stretches in one field.
> Populate opportunistically from session experience; for bulk pre-population see `/prompts/recommended-stretches-bulk-populate.md`.

---

#### Excluded exercises (do not update, do not include in workout design)
Maintain your own list of excluded exercises with reasons:
- Add excluded exercises here as you identify them
- Flag in Notion Notes field with `[EXCLUDE]` prefix if an entry exists

---

### Exercise Playbook

- **Database URL**: `YOUR_EXERCISE_PLAYBOOK_PAGE_URL`
- **Data Source**: `YOUR_EXERCISE_PLAYBOOK_DATA_SOURCE`

| Property | Type | Options |
|----------|------|---------|
| Exercise | Title | — |
| Status | Select | `Mainstay`, `Regular`, `Exploring`, `Avoid`, `Needs Work` |
| Muscle Group | Multi-select | `Chest`, `Back`, `Shoulders`, `Arms`, `Core`, `Legs`, `Glutes` |
| Tags | Multi-select | `Compound`, `Isolation`, `Unilateral`, `Cable`, `Handle Bar`, `Mobility`, `Core Stability` |
| Coaching Cues | Rich text | Posture, breathing, tempo, key form points |
| Common Mistakes | Rich text | What to watch for |
| smart gym Settings | Rich text | Mode, handle, cable position, seat height |
| Load Notes | Rich text | Current working weight, rep range observations |
| My Notes | Rich text | How it feels, ROM observations |

> Create Exercise Playbook entries one at a time (not batched).

---

## Rules

- Always tag the correct **Persona** — this scopes future AI agent access.
- Multi-select values must be **JSON array strings**: `"[\"Value1\", \"Value2\"]"`
- Date properties use expanded format: `"date:Due Date:start"`, `"date:Due Date:is_datetime"`
- Always **fetch before update** — use exact content strings from fetch results for `old_str` matching.
