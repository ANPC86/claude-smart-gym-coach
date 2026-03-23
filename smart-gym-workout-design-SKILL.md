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
last_updated: 2026-03-23
---

# smart cable gym — Workout Design Skill

## Step 0: Fetch Carry-Forward Flags (run BEFORE intake)

**Always fetch the Current Workout Notes page before starting intake.**
Page ID: `YOUR_WORKOUT_NOTES_PAGE_ID`

Look for a `## 🚩 Carry-Forward Flags` section. If present:
- Read every flag and treat it as a **hard constraint** for this session design
- Surface each flag explicitly in the Session Overview section
- Examples: hold weight on X, watch bilateral balance on Y, add exercise type Z

If absent or empty: proceed normally — no carry-forward constraints.

**Do not skip this step.** This is the mechanism that connects session review to workout design.

---

## Step 1: Intake (run BEFORE generating any JSON)

Use `ask_user_input` to collect session parameters in a single call.

**Q1 — Time Available** (single_select): 30 min | 45 min | 60 min | 75+ min
**Q2 — Session Goal** (multi_select): Discovery | Strength Building | Gym Challenge | Recovery / Light Day
**Q3 — Cardio Preference** (single_select): Rowing | Skiing | Vita | Skip
**Q4 — Any notes?** (open text) — e.g. "right shoulder tender", "skip legs today"

---

## Step 2: Derive Muscle Readiness from Notion

Query the Training Journal to determine which muscle groups are fresh vs fatigued.

```
Tool: Notion:notion-search
query: "training session"
data_source_url: "YOUR_TRAINING_JOURNAL_DATA_SOURCE"
page_size: 5
```

Fetch the last 3–5 sessions (sort by most recent). Check `Focus` property for muscle groups trained.

| Status | Criteria |
|--------|---------|
| 🔴 FATIGUED | Trained in last 24h |
| 🟡 RECOVERING | Trained 24–48h ago |
| 🟢 READY | > 48h rest |

State derived readiness in walkthrough preamble. Honor any user overrides from intake notes.

---

## Step 3: Select Exercises

### A) Known exercises — look up weight and scores in Notion Exercise Library

For any exercise you plan to include with progressive load (strength goal), query its history:

```
Tool: Notion:notion-search
query: "[exercise title]"
data_source_url: "YOUR_EXERCISE_LIBRARY_DATA_SOURCE"
page_size: 3
```

Then `notion-fetch` the matched page to get: `Last Weight`, `Last Amp`, `Last Force`, `Last Bal`.

Use these for weight selection (rationale code A) and progression gate checks.

**AMP Score Special Cases — do not apply as progression gate:**

Some exercises have a structural ROM ceiling where AMP will never reach 4–5 regardless of technique. Identify these through repeated sessions where AMP stays capped. Use FCS-only gating for load progression on these.

| Exercise | AMP Rule |
|----------|----------|
| Seated Tricep Rope Cross-Body Crunch | AMP = mobility indicator only. Use FCS to gate load. AMP reflects forward flexion ROM ceiling, not technique. |
| [Add exercises here as you identify them] | AMP = mobility indicator only; FCS gates load |

### B) Discovery candidates — find undiscovered exercises

Discovery candidates come from `library_cache_slim.json` — exercises where the corresponding Library ID has `Discovered = false` (or is absent) in the Notion Exercise Library.

**Efficient approach:**
1. From `library_cache_slim.json`, generate candidate IDs matching today's target muscle groups and positions
2. For each candidate title, spot-check Notion with a quick search to confirm not yet discovered
3. Prioritize candidates that:
   - Match today's target muscle groups
   - Have positions compatible with the session's position flow (non-increasing 10→0)
   - Are not stretch/Pilates (dataStatType 4, 5, tabId 18, 20)

**Discovery filtering rules for `library_cache_slim.json`:**
- Exclude Pilates (tabId 18, 20)
- Exclude stretch-only (dataStatType 4, 5) from discovery pool
- Cardio exercises (category 6) use `id` field; strength exercises use `actionLibraryGroupId`
- Cast all IDs to `str()` for matching

### C) Validate every ID
Confirm every `id` in the workout JSON matches `actionLibraryGroupId` in `library_cache_slim.json` exactly. Never invent IDs.

---

## Step 4: Build the Workout

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
| **1 — Setup** | Neck Stretch — rotate variants each session |
| **2 — Cardio** | Row/Ski: reps=480, mode=1. Vita: kcal_target=85, level=3–5, mode=1 |
| **3 — Mobility** | 4–5 dynamic moves (3 for 30-min) |
| **4 — Strength** | Position-ordered, non-increasing (10→0) |
| **5 — Decompression** | Supine Spinal Stretch + Butterfly Stretch |

### Mandatory Every Session
Define your own mandatory exercises based on your training goals. Structure mandatories around **movement patterns** to ensure consistent coverage while rotating through the library. Common patterns:

1. **A core rotation movement** (e.g. Woodchop variant) — Phase 3 or 4, 2 sets. Rotate unseen variants to maximize discovery.
2. **A core stability movement** (e.g. Crunch variation, Anti-rotation press) — Phase 4, final strength exercise.
3. **A vertical pull variant** (e.g. Lat Pulldown) — Phase 4, 2–3 sets. Rotate unseen variants.
4. **A push/twist mobility exercise** — Phase 3 or 4, every session.
5. **A hip flexor mobility exercise** — Phase 3, every session.
6. **Kneeling exercises caution** — not a mandatory, but a constraint: if the user has quad flexibility limits, kneeling positions structurally suppress AMP scores. Do not over-load; use FCS-only gating.

> **Tip:** For each mandatory pattern, maintain a priority list of unseen variants. Once all variants are discovered, fall back to your preferred anchor exercise.

### Goal Modifiers
| Goal | Adjustments |
|------|------------|
| **Discovery** | Min 3–4 new exercises. Mode 1. Spread across muscle groups. |
| **Strength Building** | 8–10 exercises. 3 sets. Mode 3 for sets 2+3. Progress only if scores ≥3 (≥4 compounds). |
| **Gym Challenge** | High-rep, higher-kcal bias. Vita if cardio slot. Confirm whether an active challenge is running before applying. |
| **Recovery / Light Day** | Mode 1 only. 1–2 sets. Weight –20–30%. Mobility/bodyweight priority. |

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
```json
{ "kcal_target": 85, "level": 5, "mode": 1, "rest": 30 }
```
No reps. No weight. Mode always 1. Level 3–5. Max 1 set.

### Pilates (tabId 18, 20) — NOT SUPPORTED. Omit.

---

## Modes
| Mode | Use |
|------|-----|
| **1** | Warm-ups, cooldowns, ALL discovery exercises, Row, Ski, Vita, Recovery |
| **2** | Chains — only if explicitly requested |
| **3** | MANDATORY for strength sets 2 & 3 |

---

## Ordering (strict)

**Primary:** position 10 → 9 → 8 → … → 0. Never backtrack.

**Secondary (minimize accessory swaps):**
- Pos 10: Barbell(4) → Belt(10) → no accessory
- Pos 9: Bench(1,9) → Handles(5) → Ankle Straps(3)
- Pos 0: Tricep Rope(2) → Handles(5)

**Rest buffers:** 30–60s position change | 60s accessory swap | 2–3 min large changes

**FINAL CHECK:** scan entire list for position backtracking before output.

---

## Accessory Map
```
1=Flat Bench | 2=Tricep Rope | 3=Ankle Straps | 4=Barbell Bar
5=Handles | 7=Skiing Handles | 8=Rowing Bench | 9=Adjustable Bench | 10=Weightlifting Belt
```

---

## Weight Selection

- **A) Notion Exercise Library history exists** → fetch `Last Weight` ± adjustment for scores + readiness
- **B) Related movement** → conservative carry-over from similar pattern
- **C) No history** → `library_cache_slim` `recommendedWeight`, adjusted down for standing/unilateral

State rationale code (A/B/C) for every exercise. Smart gym accepts integer weights only — round down.

**Progression gate:** AMP ≥3 AND FCS ≥3 to increase load. Compounds require ≥4.
See AMP special cases in Step 3A — some exercises use FCS-only gating.

---

## Discovery Exercise Rules
- `Discovered = false` (or absent from Notion Exercise Library) = eligible
- Spread across muscle groups, categories, positions
- Mode 1 always
- Weight: library `recommendedWeight`, adjusted down for standing/unilateral
- Flag in walkthrough: **⭐ [DISCOVERY]** + muscle target + setup notes

---

## User Physical Constraints (customize for your user)
Replace these with your own constraints — these are examples:
- Forearm tightness / wrist sensitivity → relaxed grip, neutral wrist on all pulling exercises
- Limited knee flexion → box squat modification; avoid deep knee flexion
- Kneeling exercises — if quad flexibility is a known limiter, AMP scores may be structurally suppressed; don't over-load kneeling positions based on AMP alone
- Lighter load on standing handle-bar pulls and standing movements
- Prefer floor exercises; supine cooldown with neck wedge

**Strength References (lbs):** Squat [YOUR_SQUAT] | Row [YOUR_ROW] | Bench [YOUR_BENCH] | OHP [YOUR_OHP]

---

## Safety Rules
- Pain 0/10 target, cap 2/10. Rising pain or form breakdown → reduce load/ROM immediately.
- Never invent IDs. Unvalidated ID → omit or replace.

---

## Workout Naming
Format: `<Level> <Tag>-<Focus> DAY x/y v#`
Tags: STR | DISC | CHAL | REC | FB | UB | LB

---

## Output Format

**FIRST:** one fenced `json` block.
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
1. **Session Overview** — date, goal, time budget, carry-forward flags, muscle readiness
2. **Phase-by-phase** — position grouping, accessory logic, setup notes
3. **Per-exercise** — weight rationale (A/B/C), ⭐ discovery flags, technique cues, scaling
4. **Discovery count** — X new exercises today | Y total | Z remaining of [library total]
5. `💬 **Your notes:**` prompt on every strength and cardio exercise

### Walkthrough icons
- ⭐ Discovery | ★ Mandatory | 📈 Progression | ⏸ Hold | ⚠️ Technique flag

---

## Notion Writes After Workout Design

**1. Write full walkthrough to Current Workout Notes** (`YOUR_WORKOUT_NOTES_PAGE_ID`):
- `replace_content` — full page rewrite
- Include all 5 phases, per-exercise cues, `💬 **Your notes:**` prompts (blank for user)
- Do NOT modify the `🚩 Carry-Forward Flags` section — read-only during design

**2. Update Fitness Coach landing page** (`YOUR_LANDING_PAGE_ID`):
- `update_content` — replace Current Session block with new date/workout name

**3. Create Training Journal entry** (`YOUR_TRAINING_JOURNAL_DATA_SOURCE`):
- Status: PLANNED, session name, date, focus, estimated duration

---

## Pre-Response Checklist
- [ ] Step 0 complete — Carry-forward flags read and applied (or confirmed absent)
- [ ] Carry-forward flags listed in Session Overview
- [ ] Intake complete (time, goal, cardio, notes)
- [ ] Muscle readiness derived from Notion Training Journal
- [ ] Known exercise weights fetched from Notion Exercise Library
- [ ] Every `id` validated against `library_cache_slim.json`
- [ ] No position backtracking
- [ ] Phase 1–5 structure present
- [ ] Vita: kcal_target, mode=1, no weight, level set
- [ ] Row/Ski: mode=1, not mode=6
- [ ] Goal modifiers applied
- [ ] Mandatory exercises present (core rotation, core stability, vertical pull, push/twist, hip flexor)
- [ ] Physical constraints applied
- [ ] Weight rationale (A/B/C) for every exercise
- [ ] All weights are integers (rounded down)
- [ ] Discovery count updated in walkthrough
- [ ] Notion writes queued (Current Workout Notes, landing page, Training Journal)
