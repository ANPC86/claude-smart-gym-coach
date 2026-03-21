# Fitness Coach — Notion Workspace Reference
> **Template version** — replace all page/database IDs with your own before use.
> See README for how to find your Notion IDs.

Use this reference when creating, updating, or querying Notion. The Notion MCP connector handles authentication — just call the tools directly once connected.

---

## How to Find Your Notion IDs

- **Page ID**: Open a page in Notion → share → copy link. The UUID is the string after the last `/` (format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)
- **Database ID**: Open a database as a full page → copy link. Same UUID format.
- **Data Source ID**: Use `notion-fetch` on the database page → look for `collection://` reference in the response

---

## Fitness Coach — Key Pages

| Page | Placeholder ID | Purpose |
|------|----------------|---------|
| 💪 Fitness Coach (landing) | `YOUR_LANDING_PAGE_ID` | Main dashboard; current session pointer |
| 📝 Current Workout Notes | `YOUR_WORKOUT_NOTES_PAGE_ID` | Live session reference — opened on device during workout |
| 🗂️ AI System Changelog | `YOUR_CHANGELOG_PAGE_ID` | Skill/config change log |

---

## Fitness Coach — Databases

### Training Journal

- **Database ID**: `YOUR_TRAINING_JOURNAL_DB_ID`
- **Data Source**: `YOUR_TRAINING_JOURNAL_DATA_SOURCE_ID`

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
Parent: { "data_source_id": "YOUR_TRAINING_JOURNAL_DATA_SOURCE_ID" }
Properties:
  Session: "B STR-BACK DAY 1/1 v1"
  "date:Session Date:start": "2026-01-01"
  "date:Session Date:is_datetime": 0
  Focus: "[\"Back\", \"Core\"]"           ← JSON array string
  Duration: 60
  Exercises: 16
  Total Sets: 38
  Coach Notes: "PLANNED — not yet completed"
Content:
  Full walkthrough in markdown (phase headers, exercises, coaching cues)
```

---

### Exercise Playbook (optional but recommended)

- **Database ID**: `YOUR_EXERCISE_PLAYBOOK_DB_ID`
- **Data Source**: `YOUR_EXERCISE_PLAYBOOK_DATA_SOURCE_ID`

Suggested schema:

| Property | Type | Options |
|----------|------|---------|
| Exercise | Title | — |
| Status | Select | `Mainstay`, `Regular`, `Occasional`, `Testing`, `Avoid` |
| Muscle Group | Multi-select | major muscle groups |
| Coaching Cues | Rich text | — |
| Common Mistakes | Rich text | — |
| smart gym Settings | Rich text | position, accessory, mode notes |
| Load Notes | Rich text | weight progression history |
| My Notes | Rich text | personal observations |

> **Note:** Create entries one at a time — do not batch-create Exercise Playbook entries in a single call.

---

## 📝 Current Workout Notes — Usage

This page is the **primary session reference** — the user opens this on their tablet/phone during the workout.

**Structure:**
- `🚩 Carry-Forward Flags` section at the top — constraints and notes from last reviewed session
- `## Phase 1–5` — phase headers each containing `### Exercise Name` blocks
- Each exercise: ID in backticks, weight/sets/mode, weight rationale, cues, `💬 **Your notes:**` prompt (left blank for user)
- `## Discoveries Today` — projected count

**Writing during workout design (automatic):**
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
Scan for populated `💬 **Your notes:**` fields — these override tracker defaults.

---

## 💪 Landing Page — Usage Pattern

Keep the landing page lean — it's a dashboard, not a history log. Session history belongs in Training Journal.

**Current session block (updated each workout design):**
```markdown
### 💪 Current Session — [Date]
**[Workout Name]** · ~[X] min · [Cardio] · [Focus]
**Goals:** [Goals line]
📋 Full walkthrough → [Current Workout Notes](YOUR_WORKOUT_NOTES_URL)

> 💡 [Key coaching note — technique priority, fatigue flag, etc.]

---
```

**Update pattern:**
```
Tool: Notion:notion-update-page
Command: update_content
page_id: "YOUR_LANDING_PAGE_ID"
content_updates: [{ "old_str": "### 💪 Current Session...", "new_str": "### 💪 Current Session — [new date]..." }]
```

> **Always fetch the page first** to get exact content strings for `old_str` matching.

---

## Notion Operation Rules

- **Always `notion-fetch` before writing** — use exact strings from fetch for `old_str` matching
- `update_content` with `old_str`/`new_str` pairs for targeted edits
- `replace_content` for full page rewrites (Current Workout Notes)
- **Verify writes** by fetching the page after update — don't trust success response alone
- Multi-select values must be **JSON array strings**: `"[\"Value1\", \"Value2\"]"`
- Date properties use expanded format: `"date:Session Date:start"`, `"date:Session Date:is_datetime"`
