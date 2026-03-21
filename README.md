# smart cable gym — Claude Fitness Coach Setup

A Claude Project configuration for using Claude as an AI fitness coach with the **smart cable gym** smart cable machine. Designed for systematic library discovery, technique-gated progression, and Notion-based session journaling.

---

## What This Does

- **Designs workouts** tailored to your muscle readiness, physical constraints, and training goals — then writes them to a Notion page you can read on your phone/tablet during the session
- **Reviews completed sessions** by parsing the smart gym export JSON — identifies PRs, technique limiters, new exercise discoveries, and muscle readiness for next session
- **Tracks all 1,041 exercises** in the smart gym library across sessions via a JSON tracker file — enabling systematic discovery of the full library
- **Maintains coaching continuity** via carry-forward flags between sessions (technique holds, deferred exercises, preference notes)

---

## Files in This Repo

| File | What It Is |
|------|------------|
| `PROJECT_INSTRUCTIONS_template.md` | The system prompt / project instructions for your Claude Project. Fill in your biometrics and constraints. |
| `smart-gym-workout-design-SKILL.md` | Skill file for workout design. Paste into your project as a file upload. |
| `smart-gym-session-review-SKILL.md` | Skill file for post-session review. Paste into your project as a file upload. |
| `notion-reference.md` | Notion database IDs and tool call patterns. Fill in your own IDs. |
| `fitness-notion-workflow.md` | Rules for when/what Claude writes to Notion. |
| `Gym_Tracker_skeleton.json` | Starter tracker with 10 sample exercises. Expand as you train. |

> **Not included:** The full 1,041-exercise library cache (`library_cache_slim.json`). This is derived from the your gym app's own data and is not redistributable here. See the section below on how to build or obtain it.

---

## Setup Overview

### 1. Create a Claude Project
In Claude.ai → create a new Project → name it something like "Fitness Coach".

### 2. Set the System Prompt
Copy `PROJECT_INSTRUCTIONS_template.md` into the Project Instructions field. Fill in:
- Your biometrics (weight, height, body fat %, resting HR)
- Your working weights (Squat / Row / Bench / OHP)
- Your physical constraints (injuries, mobility limits, preferences)
- Your training frequency

### 3. Upload Project Files
Upload the following to your Claude Project's file storage:
- `smart-gym-workout-design-SKILL.md`
- `smart-gym-session-review-SKILL.md`
- `notion-reference.md` (after filling in your Notion IDs)
- `fitness-notion-workflow.md`
- `Gym_Tracker_skeleton.json` (or your own tracker if you have one)
- Your `library_cache_slim.json` (see below)

### 4. Set Up Notion (optional but recommended)
Create the following in Notion:
- A **Fitness Coach landing page** (dashboard)
- A **Current Workout Notes page** (session walkthrough — opened on device during workout)
- A **Training Journal database** (session log)
- An **Exercise Playbook database** (per-exercise notes and coaching cues)

Connect the **Notion MCP** in Claude → Settings → Integrations. Then add your page/database IDs to `notion-reference.md`.

### 5. Start Training
Ask Claude: `"design a workout for today"` → answer the intake questions → Claude outputs importable JSON + a full coaching walkthrough.

After your session: export the training record from the your gym app → upload to Claude → Claude reviews scores, flags PRs, updates the tracker.

---

## The Library Cache

The smart gym has 1,041 exercises. Claude needs a local cache of this library to:
- Validate exercise IDs before generating workouts
- Look up muscle groups, accessories, difficulty, and positions
- Identify discovery candidates (exercises not yet in your tracker)

**This file is not included** in this repo as it's derived from the manufacturer's proprietary data.

Options:
- Use the your gym app API or export tools to pull the exercise library
- Check if your gym's manufacturer provides an API or export tools — some community projects exist for specific machines
- Build your own by querying the app's local data

The expected schema for each entry in `library_cache_slim.json`:
```json
{
  "id": 12345,
  "actionLibraryGroupId": 12345,
  "title": "Exercise Name",
  "mainMuscleGroupName": "Chest",
  "auxiliaryMuscleGroupList": ["Triceps", "Shoulders"],
  "accessories": "1,5",
  "outPosition": 9,
  "category_id": 1,
  "tabId": 1,
  "recommendedWeight": 20,
  "dataStatType": 1,
  "completionMethod": 1,
  "isUseDevice": 1
}
```

---

## The Tracker File

`Gym_Tracker_Master_CURRENT.json` is a flat JSON array where each entry represents one exercise and accumulates session data over time. It lives in your Claude Project and is updated after each session review.

Key fields:
- `actionLibraryGroupId` — string ID matching the library (always stored as string for consistent matching)
- `times_seen_in_records` — how many times you've done this exercise (0 = undiscovered)
- `last_maxWeight` — peak weight achieved in most recent session
- `last_amplitudeStableScore` / `last_forceControlScore` — technique scores 1–5, gates load progression
- `last_bilateralBalanceScore` — left/right balance 1–5, useful for catching asymmetries
- `notes` — running log of session-by-session data in plain text

The skeleton file in this repo has 10 sample entries. As you train, Claude appends entries and you re-upload the updated file.

---

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

Claude generates workouts as JSON that can be imported into the your gym app (via the Gym Workout Manager or directly):

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

Mode 1 = concentric only | Mode 3 = eccentric (resistance on both concentric and eccentric phases)

---

## Tips

- **Physical constraints matter** — fill them in accurately. The skill applies them every session (wrist cues, knee modifications, load adjustments for standing movements).
- **The carry-forward system is the coaching memory** — after each session review, Claude writes flags to Notion that the next workout design reads before doing anything else.
- **Discovery is a long game** — 1,041 exercises, 1–2 new ones per session. At 5 sessions/week that's roughly a year to see everything.
- **Re-upload the tracker after reviews** — Claude provides an updated tracker file after each session review. Drop it back into your Claude Project to keep the history current.

---

## Related Projects

- Check GitHub for community workout manager projects specific to your machine

---

## License

MIT — use freely, adapt for your own setup.
