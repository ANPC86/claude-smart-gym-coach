---
name: smart-gym-workout-design
description: >
  Design structured workout sessions for import into the smart cable gym.
  Trigger on: "design a workout", "plan my session", "build a workout", "make a workout for today",
  or any request to generate a smart gym session. ALWAYS run the intake flow first, then produce
  a validated JSON block followed by a full coaching walkthrough. Do NOT output workout JSON
  for general fitness Q&A — only on explicit workout design requests. Governs all aspects
  of smart gym workout generation: intake, JSON schema, exercise ordering, mode selection,
  discovery tracking, safety rules, and weight rationale.
---

# smart cable gym — Workout Design Skill

## Step 0: Fetch Carry-Forward Flags (run BEFORE intake)

**Always fetch the Current Workout Notes page before starting intake.**
Page ID: `YOUR_CURRENT_WORKOUT_NOTES_PAGE_ID`

Look for a `## 🚩 Carry-Forward Flags` section at the top of the page. If present:
- Read every flag and treat it as a **hard constraint** for this session design
- Surface each flag explicitly in the Session Overview section of the walkthrough
- Examples of flags: hold weight on exercise X, watch bilateral balance on Y, unlock Z to higher weight, add exercise type W to session

If the section is absent or empty: proceed normally — no carry-forward constraints.

**Do not skip this step.** This is the mechanism that connects the session review's "Next Session Focus" to the actual workout design. Without it, coaching continuity is lost.

---

## Step 1: Intake (run BEFORE generating any JSON)

Use `ask_user_input` to collect session parameters. Present all questions in a single call.

### Intake Questions

**Q1 — Time Available** (single_select)
- 30 minutes
- 45 minutes
- 60 minutes
- 75+ minutes

**Q2 — Session Goal** (multi_select)
- Discovery (new exercises priority)
- Strength Building (load progression)
- Gym Challenge (volume/kcal)
- Recovery / Light Day

**Q3 — Cardio Preference** (single_select)
- Rowing (default)
- Skiing
- Vita (calorie target)
- Skip cardio today

**Q4 — Any notes?** (open text — ask as prose, not a widget)
e.g. "right shoulder tender", "skip legs today", "focus upper body"

### Muscle Readiness (auto-derive, no intake question needed)
- Read `Gym_Tracker_Master_CURRENT.json` and query Training Journal in Notion for recent sessions
- Flag any muscle group trained in the last 24h as FATIGUED, last 48h as RECOVERING
- State derived readiness in the walkthrough preamble
- If user notes an override ("legs feel fine", "shoulder still sore") → honor it

---

## Step 2: Validate Inputs Against Library

Before building the workout:
1. Search `library_cache_slim.json` for all exercises to be included
2. Confirm every `id` matches `actionLibraryGroupId` exactly — never invent IDs
3. Select discovery exercises (zero `times_seen_in_records` in tracker) matching target muscle groups and positions for today's session
4. If an ID cannot be validated → find a valid replacement or omit

**Discovery filtering rules for `library_cache_slim.json`:**
- Exclude Pilates (tabId 18, 20)
- Exclude stretch-only (dataStatType 4, 5) from discovery pool
- Cardio exercises (category 6) use `id` field; strength exercises use `actionLibraryGroupId`
- Always cast IDs to `str()` for matching — tracker stores them as strings

---

## Step 3: Build the Workout

### Session Sizing by Time
| Time | Total Exercises |
|------|----------------|
| 30 min | 12–14 |
| 45 min | 14–16 |
| 60 min | 16–18 |
| 75+ min | 18–20 |

### Workout Structure

| Phase | Content |
|-------|---------|
| **1 — Setup** | Neck Stretch — rotate variants each session. Do not hardcode a single ID. |
| **2 — Cardio** | Per intake Q3. Row/Ski: reps=480, mode=1. Vita: kcal_target=85, level=3–5, mode=1. Never mix types. |
| **3 — Mobility** | 4–5 dynamic moves (scale down to 3 for 30-min sessions) |
| **4 — Strength** | Per session goal. Sorted strictly position → accessory. |
| **5 — Decompression** | Supine Spinal Stretch + Butterfly Stretch (use IDs from your library) |

### Mandatory Every Session (regardless of goal)
Define your own mandatory exercises based on your training goals. Common examples:
1. **A core rotation movement** (e.g. Woodchop variant) — Phase 3 or 4, 2–3 sets. Rotate through unseen variants to maximize discovery.
2. **A core stability movement** (e.g. Anti-rotation press, Crunch variation) — Phase 4, final strength exercise.
3. **A vertical pull variant** (e.g. Lat Pulldown, Straight-Arm Pulldown) — Phase 4, 2–3 sets. Rotate unseen variants; fall back to a known anchor.
4. **A push/twist mobility exercise** — Phase 3, every session.

> **Tip:** Set your mandatories based on movement patterns you want to consistently build, not just individual exercises. This ensures you rotate through the library while still hitting foundational patterns each session.

### Goal Modifiers
| Goal | Adjustments |
|------|------------|
| **Discovery** | Minimum 3–4 new exercises. Spread across muscle groups. Mode 1. |
| **Strength Building** | 8–10 compound/accessory exercises. 3 sets. Mode 3 sets 2+3. Progress weight only if scores ≥3 (≥4 compound). |
| **Gym Challenge** | Bias toward high-rep, higher-kcal exercises. Include Vita if cardio slot. Track kcal in walkthrough. |
| **Recovery / Light Day** | Mode 1 only. Reduce sets to 1–2. Drop weight 20–30% vs last session. Prioritize mobility and bodyweight. |

---

## Exercise Types & JSON Rules

### Standard Strength (dataStatType 1, 7)
```json
{ "reps": 15, "weight": 20, "mode": 3, "rest": 60 }
```

### Rowing / Skiing (dataStatType 2, 3)
```json
{ "reps": 480, "weight": 20, "mode": 1, "rest": 0 }
```
`reps` = seconds. Mode always 1.

### Bodyweight / Timed (dataStatType 4, 5)
```json
{ "reps": 30, "rest": 30 }
```
No weight. No mode.

### VITA (dataStatType 6)
Titles: Vita High Pull, Vita Row-Tation, Vita Row, Vita Slam, Vita Lunge, Vita Pull, Vita Twist
```json
{ "kcal_target": 85, "level": 5, "mode": 1, "rest": 30 }
```
No reps. No weight. Mode always 1. Level 3–5 for beginner. Max 1 set. ⚠️ level not applied on machine — include anyway.

### Pilates / Pilates-Mat (tabId 18, 20) — NOT SUPPORTED. Omit entirely.

---

## Modes
| Mode | Use |
|------|-----|
| **1** | Warm-ups, cooldowns, ALL discovery exercises, Row, Ski, Vita, Recovery day |
| **2** | Chains — only if explicitly requested |
| **3** | MANDATORY for strength sets 2 & 3 |

No Mode 6 exists. Never use it.

---

## Ordering (strict)

**Primary:** position 10 → 9 → 8 → … → 0. Never backtrack.

**Secondary within same position — minimize accessory swaps:**
- Pos 10: Barbell(4) → Belt(10) → no accessory
- Pos 9: Bench(1,9) → Handles(5) → Ankle Straps(3)
- Pos 0: Tricep Rope(2) → Handles(5)

**Rest buffers:** 30–60s position change | 60s accessory change | 2–3 min large changes

**FINAL CHECK:** scan entire exercise list for position backtracking before outputting.

---

## Accessory Map
```
1=Flat Bench | 2=Tricep Rope | 3=Ankle Straps | 4=Barbell Bar
5=Handles | 7=Skiing Handles | 8=Rowing Bench | 9=Adjustable Bench | 10=Weightlifting Belt
```
`library["accessories"]` is comma-separated — parse to sorted list for grouping.

---

## Weight Selection
- **A) History exists** → last successful weight ± adjustment for technique scores + readiness
- **B) Related pattern** → conservative carry-over from similar movement
- **C) No history** → library `recommendedWeight`, adjusted down for standing/unilateral

State rationale (A/B/C) for every exercise in walkthrough.

**Progression gate:** scores ≥3 required to increase load. Prefer ≥4 for compounds (squat, deadlift, press, row).

**Important:** smart gym accepts integer weights only — always round down, never use decimals.

---

## Discovery Exercise Rules
- Zero `times_seen_in_records` in tracker = eligible
- Spread across muscle groups, categories, positions
- Mode 1 always
- Start at library `recommendedWeight`, adjust down for standing/unilateral
- Flag in walkthrough: **⭐ [DISCOVERY]** + muscle target + setup notes

---

## User Physical Constraints (customize for your user)
Replace these with your own constraints — these are examples:
- Forearm tightness / wrist sensitivity → relaxed grip, neutral wrist cues on all pulling exercises
- Limited knee flexion → box squat modification; avoid deep-knee-flexion exercises unless explicitly cleared
- Lighter load on standing handle-bar pulls and standing movements
- Prefer floor exercises; supine cooldown with neck wedge

**Strength References:** Fill in from user profile (Squat / Row / Bench / OHP working weights in lbs)

---

## Safety Rules
- Pain target 0/10. Cap 2/10. If pain rises or form breaks → reduce load/ROM immediately.
- Never invent IDs. Unvalidated ID → omit or replace.

---

## Workout Naming
Format: `<Level> <Tag>-<Focus> DAY x/y v#`
- Tags: STR=Strength | DISC=Discovery | CHAL=Challenge | REC=Recovery | FB=Full Body | UB=Upper | LB=Lower

---

## Output Format

**FIRST:** one fenced `json` block. Nothing before it.
**AFTER:** walkthrough in JSON order.

### JSON Schema
```json
{
  "name": "B DISC-FB DAY 1/1 v1",
  "exercises": [
    {
      "id": "<library id>",
      "sets": [ { "reps": 15, "weight": 20, "mode": 3, "rest": 60 } ]
    }
  ]
}
```

### Walkthrough sections
1. **Session Overview** — date, goal, time budget, muscle readiness summary (derived + any overrides)
2. **Phase-by-phase breakdown** — for each phase: position grouping, accessory swap logic, setup notes
3. **Per-exercise notes** — weight rationale (A/B/C), ⭐ [DISCOVERY] flags, technique cues, scaling rules
4. **Discovery count update** — X new exercises today | Y total seen | Z remaining of 1,041

### Walkthrough formatting conventions
- ⭐ Discovery exercise
- ★ Mandatory exercise
- 📈 Progression (weight increase)
- ⏸ Hold (maintain weight, refine technique)
- ⚠️ Technique flag
- `💬 **Your notes:**` prompt on every strength and cardio exercise (leave blank for user to fill in mid-session)
- Weight rationale codes (A/B/C) on each exercise

---

## Pre-Response Checklist
- [ ] **Step 0 complete** — Current Workout Notes fetched; carry-forward flags read and applied (or confirmed absent)
- [ ] Carry-forward flags listed in Session Overview (if any)
- [ ] Intake complete (time, goal, cardio, notes)
- [ ] Muscle readiness derived and stated
- [ ] Every `id` validated against library
- [ ] No position backtracking
- [ ] Phase 1–5 structure present
- [ ] Vita: kcal_target, mode=1, no weight, level set
- [ ] Row/Ski: mode=1, not mode=6
- [ ] Goal modifiers applied
- [ ] Mandatory exercises present
- [ ] Physical constraints applied
- [ ] Weight rationale (A/B/C) for every exercise
- [ ] All weights are integers (rounded down)
- [ ] Discovery count updated in walkthrough
