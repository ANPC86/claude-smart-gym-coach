# smart cable gym — Claude Fitness Coach Setup

A Claude Project configuration for using Claude as an AI fitness coach with the **smart cable gym** smart cable machine. Designed for systematic library discovery, technique-gated progression, and Notion-based session journaling.

---

## Plain-English Overview

This system is organized around one top-level dashboard with four main operating databases, a live carry-forward notes page, and a coaching reference layer.

At the top is **Fitness Coach**, which acts as the hub. It shows the current training context, links to the core databases, and serves as the main entry point into the system.

The system is structured like this:

- **strategy:** goals
- **state:** body metrics
- **execution:** current workout notes
- **history:** training journal
- **knowledge:** exercise playbook

---

## System Breakdown

### 1. Command Center / Dashboard
The **Fitness Coach** page is the command center.

It gives a high-level snapshot of:
- current phase
- training context
- body snapshot
- strength references
- physical considerations
- links to the rest of the system

This is the main "where am I right now?" page.

### 2. Live Session Handoff
**Current Workout Notes** is the live operational page.

It holds:
- carry-forward flags
- next-session instructions
- exercise caps and holds
- exclusions
- mandatory movements
- session-planning notes

This is the short-horizon coaching layer: the page that tells the next session what matters most.

### 3. Training History
**Training Journal** is the session archive.

It tracks workout-level history such as:
- session date
- duration
- focus
- recovery rating
- PRs
- notes
- coach review summary

This is the historical record of what was done and how it went.

### 4. Exercise Knowledge
**Exercise Playbook** is the exercise reference system.

It stores:
- coaching cues
- common mistakes
- preferred setup notes
- load notes
- personal observations
- status and tags
- **undiscovered exercises** — a dynamic list to guide systematic library discovery

This is the knowledge base for how each exercise works for the user, as well as your discovery frontier.

### 5. Goals
**Fitness Goals** is the strategy layer.

It captures:
- what the goal is
- priority
- status
- current state
- target
- target date
- approach

This is the long-range direction for the system.

### 6. Body Metrics
**Body Metrics** is the state-tracking layer.

It tracks:
- body weight
- body fat
- resting heart rate
- energy level
- sleep quality
- phase
- notes

This is the physical status and trend view.

### 7. Coaching / Reference Layer
A separate reference layer supports the system itself.

This includes things like:
- coaching rules
- workflow notes
- system references
- changelog or operating notes

This is the meta-layer that explains how the system should be interpreted and used.

---

## Benefits of This Setup

- Clear separation between strategy, state, execution, history, and knowledge
- Easy to understand what is current versus what is archival
- Session planning can use live carry-forward notes without cluttering long-term records
- Exercise-specific lessons are preserved in one place instead of being lost inside session logs
- Goals stay visible without being mixed into daily workout details
- The system can grow over time without losing structure
- Undiscovered exercises are tracked in one place, making systematic discovery actionable instead of random

---

## What This Does

- **Designs workouts** tailored to your muscle readiness, physical constraints, and training goals — then writes them to a Notion page you can read on your phone or tablet during the session
- **Reviews completed sessions** by parsing the smart gym export JSON — identifies PRs, technique limiters, new exercise discoveries, and muscle readiness for the next session
- **Maintains coaching continuity** via carry-forward flags between sessions, including technique holds, deferred exercises, and preference notes
- **Uses Notion as the source of truth** for session history, exercise notes, goals, and live workout guidance

---

## Files in This Repo

| File | What It Is |
|------|------------|
| `PROJECT_INSTRUCTIONS_template.md` | The system prompt / project instructions for your Claude Project. Fill in your biometrics and constraints. |
| `smart-gym-workout-design-SKILL.md` | Skill file for workout design. Paste into your project as a file upload. |
| `smart-gym-session-review-SKILL.md` | Skill file for post-session review. Paste into your project as a file upload. |
| `notion-reference.md` | Notion database IDs and tool call patterns. Fill in your own IDs. |
| `fitness-notion-workflow.md` | Rules for when and what Claude writes to Notion. |

---

## Setup Overview

### 1. Create a Claude Project
In Claude.ai, create a new Project and name it something like **Fitness Coach**.

### 2. Set the System Prompt
Copy `PROJECT_INSTRUCTIONS_template.md` into the Project Instructions field. Fill in:
- your biometrics (weight, height, body fat %, resting HR)
- your working weights (Squat / Row / Bench / OHP)
- your physical constraints (injuries, mobility limits, preferences)
- your training frequency

### 3. Upload Project Files
Upload the following to your Claude Project's file storage:
- `smart-gym-workout-design-SKILL.md`
- `smart-gym-session-review-SKILL.md`
- `notion-reference.md` (after filling in your Notion IDs)
- `fitness-notion-workflow.md`

### 4. Set Up Notion
Create the following in Notion:
- a **Fitness Coach landing page** (dashboard)
- a **Current Workout Notes page** (session walkthrough — opened on device during workout)
- a **Training Journal database** (session log)
- an **Exercise Playbook database** (per-exercise notes, coaching cues, and undiscovered exercises list)
- a **Fitness Goals database** (strategy and direction)
- a **Body Metrics database** (physical state and trends)

Connect the **Notion MCP** in Claude, then add your page and database IDs to `notion-reference.md`.

### 5. Start Training
Ask Claude: `design a workout for today`

Claude can generate:
- importable workout JSON
- a full coaching walkthrough
- a Notion-ready session page or notes structure

After your session:
- export the training record from the smart gym app
- upload it to Claude
- Claude reviews scores, flags PRs, and updates the Notion-based coaching memory

## Score Interpretation

The smart gym generates four technique scores per exercise set (1–5 scale):

| Score | Meaning |
|-------|---------|
| `completionScore` | Overall set quality |
| `amplitudeStableScore` | ROM consistency — are your reps hitting the same range each time? |
| `forceControlScore` | Force output control — smooth and controlled, not jerky? |
| `bilateralBalanceScore` | Left/right symmetry on bilateral movements |

**Progression gate used in this setup:**
- Isolation exercises: both AMP and FCS ≥ 3 before increasing weight
- Compound movements (squat, deadlift, press, row): both ≥ 4

---

## Workout Import Format

Claude generates workouts as JSON that can be imported into the smart gym app:

```json
{
  "name": "B DISC-FB DAY 1/1 v1",
  "exercises": [
    {
      "id": "434",
      "sets": [
        { "reps": 15, "weight": 40, "mode": 1, "rest": 60 },
        { "reps": 15, "weight": 40, "mode": 3, "rest": 60 },
        { "reps": 12, "weight": 40, "mode": 3, "rest": 60 }
      ]
    }
  ]
}
```

Mode 1 = concentric only  
Mode 3 = eccentric (resistance on both concentric and eccentric phases)

---

## Tips

- **Physical constraints matter** — fill them in accurately. The skill applies them every session.
- **The carry-forward system is the coaching memory** — after each session review, Claude writes flags that the next workout design reads before doing anything else.
- **Discovery is a long game** — the library is large, and systematic discovery takes time. Use the undiscovered exercises list in your Exercise Playbook to guide what to try next.
- **Notion is the operating layer** — keep the dashboard, current notes page, and databases clean so the coaching system stays usable.

---

## Related Projects

- [UnofficialSpeedianceWorkoutManager](https://github.com/hbui3/UnofficialSpeedianceWorkoutManager) — community-built workout manager for the same machine, useful for workout imports and library exploration

---

## Changelog

### 2026-03-30
**Added `Recommended Stretches` field to Exercise Library schema**

Pre-exercise mobility work was previously buried as unstructured text in the Notes field — invisible to the coaching workflow and not queryable by muscle group or movement pattern.

- New field: `Recommended Stretches` (Rich text) in the Notion Exercise Library database
- Format: `stretch name · technique note · Library ID(s) · duration cue`; multiple stretches separated by semicolons
- Populated opportunistically during session reviews when a specific stretch connection is identified from training experience
- For bulk pre-population across the library, see the new prompt file: [`/prompts/recommended-stretches-bulk-populate.md`](prompts/recommended-stretches-bulk-populate.md)
- Field is RICH_TEXT for now; a future migration to a Relation column (pointing to a dedicated Stretches table) is noted as a future improvement once a backend pipeline is in place

**`notion-reference.md`**
- `Recommended Stretches` added to Exercise Library schema table
- New tool call example: "Adding or updating Recommended Stretches"

---

### 2026-03-23
**Notion Exercise Library replaces tracker JSON as source of truth**

The biggest architectural change in this update: exercise history (last weight, scores, discovered status) now lives in a Notion database instead of the flat `Gym_Tracker_Master_CURRENT.json` file. The tracker JSON is retired.

**Session Review skill (`smart-gym-session-review-SKILL.md`)**
- New Step 2: look up each exercise in the Notion Exercise Library before writing the review (replaces tracker JSON cross-reference)
- Step 8 rewritten: full Notion write-back pattern — update known exercises, create new discoveries, batch where possible
- Step 9 added: update Carry-Forward Flags and create Training Journal entry after review
- Added `Times Seen` field tracking — increment on each known exercise update, set to 1 on new discovery
- Added AMP Score Special Cases table — documents exercises where AMP is a structural mobility indicator only, not a load gate (first documented example: Seated Tricep Rope Cross-Body Crunch)
- Clarified BBS=0 on unilateral exercises is a device data artifact, not a bilateral balance flag
- Added kneeling exercise note to Physical Context: quad flexibility limits can structurally suppress AMP scores

**Workout Design skill (`smart-gym-workout-design-SKILL.md`)**
- New Step 2: derive muscle readiness from Notion Training Journal (replaces tracker JSON lookup)
- New Step 3: select exercises with Notion Exercise Library lookup for known exercises + AMP special cases
- Added matching AMP Special Cases table for design-time reference
- Hip flexor mobility exercise added as mandatory every session
- Added kneeling exercise caution to Physical Constraints
- Gym Challenge modifier now includes a note to confirm whether a challenge is currently active
- Notion Writes section added at end — walkthrough to Current Workout Notes, landing page update, Training Journal entry

**Notion reference (`notion-reference.md`)**
- Exercise Library database section added (was missing from template)
- `Times Seen` field added to Exercise Library schema — with increment rule and null-handling note
- Exercise Playbook schema table added
- Explicit callout that the flat tracker JSON is retired
- `[EXCLUDE]` convention documented for excluded exercises

**Notion workflow (`fitness-notion-workflow.md`)**
- Exercise Library write-back added as Step 1 of the post-review workflow (was missing)
- "What Lives in Notion" section added — makes the data home for each type explicit
- Removed stale reference to exercise master data living in JSON files

---

## License

MIT — use freely, adapt for your own setup.