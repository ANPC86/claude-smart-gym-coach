---
name: smart-gym-session-review
description: >
  Review and analyse a completed smart cable gym training session. Trigger on:
  "review my session", "analyse this workout", "post-session review", "how did I do",
  or whenever the user uploads a training record JSON file. Parses session data,
  identifies PRs, flags technique limiters, updates discovery count, assesses muscle
  readiness, and recommends the next session focus. Always use this skill when a
  training record is present — even if the user doesn't explicitly ask for a review.
last_updated: 2026-03-23
---

# smart cable gym — Session Review Skill

## Trigger Conditions
- User uploads or pastes a training record JSON
- User says "review my session / workout / training"
- User asks "how did I do" or "what were my scores"
- Any post-workout analysis request

---

## Step 0: Retrieve Current Workout Notes from Notion

**Before parsing the training record**, fetch the Current Workout Notes page.

- Page ID: `YOUR_WORKOUT_NOTES_PAGE_ID`
- Use `notion-fetch` to retrieve the page
- Extract every populated `💬 **Your notes:**` field
- Store as **session context notes** — these capture mid-session adjustments not visible in the raw record

**How to use the notes:**
- Weight reduction noted mid-set → use reduced weight as actual working weight, not planned
- Fatigue / discomfort flag → include in Technique Limiters with `📓 User Note:` label
- Note conflicts with record → trust the note over the record, flag the discrepancy
- Acknowledge at top of review: `📓 Workout Notes retrieved: [count] note(s)`

**After the full review**, add at the bottom:
> 🗑️ **Clear workout notes?** Notes have been incorporated. Clear the Current Workout Notes page before next session.

---

## Step 1: Parse the Training Record

Training record JSON has nested structure. Exercise data is in `detail.detail` (array).
Use `maxWeight` field for peak load. Always cast `actionLibraryGroupId` to `str()` when matching.

Extract per-exercise:

| Field | Purpose |
|-------|---------|
| `date` / `time` / `duration` | Session metadata |
| `actionLibraryGroupId` | Library ID — use for Notion lookup |
| `name` | Exercise title — use for Notion search |
| `sets` → `reps`, `weight`, `mode`, `rest` | Volume data |
| `completionScore` | Overall quality 1–5 |
| `amplitudeStableScore` | ROM consistency 1–5 |
| `forceControlScore` | Force output control 1–5 |
| `bilateralBalanceScore` | Left/right balance 1–5 |
| `actionRating` | User's own rating if present |

**BBS=0 on unilateral exercises** is a data artifact — the device does not score bilateral balance on alternating side-by-side sets. Do not flag as a balance concern.

---

## Step 2: Look Up Each Exercise in Notion Exercise Library

For each exercise in the record, query the Exercise Library in Notion:

```
Tool: Notion:notion-search
query: "[exercise title from record]"
data_source_url: "YOUR_EXERCISE_LIBRARY_DATA_SOURCE"
page_size: 3
```

From the result:
- **Match by title** (exact or close match) — get the page URL
- If found: `notion-fetch` the page to read all properties (Last Weight, Last Amp, Last Force, Last Bal, Last Seen, First Seen, Discovered)
- If not found: exercise is a **new discovery** — create entry (see Step 8)

**Batch searches where possible** — group exercises by similar terms to reduce round-trips.

Use the fetched properties to:
- Compare current session scores/weights to prior history → identify PRs
- Check `Discovered` checkbox — if false/absent → new discovery
- Check `First Seen` — if null on a discovered exercise → backfill from `notes` field or session date

---

## Step 3: Session Summary

```
Date:          [date + time]
Duration:      [X min]
Exercises:     [count]
Total Sets:    [count]
Est. Volume:   [total reps × weight where applicable, lbs]
📓 Workout Notes retrieved: [count] note(s)
```

---

## Step 4: PRs — Flag All Improvements

Compare each exercise against Notion Exercise Library history. Report:
- **Weight PR** — heavier than `Last Weight` in Notion
- **Score PR** — any score improved vs Notion record
- **First Appearance** — `Discovered` was false (= discovery)

```
🏆 PRs THIS SESSION
- [Exercise Name]: weight 45→50 lb (+5 lb PR)
- [Exercise Name]: forceControlScore 3→4 (technique PR)
```

If no PRs: state clearly — "No weight or score PRs this session."

---

## Step 5: Technique Limiters

Flag every exercise where `amplitudeStableScore` OR `forceControlScore` < 3.

For each:
- Name the score and value
- State what it's blocking (gate requires ≥3; compounds ≥4)
- Give one specific, actionable cue tied to that score

**Progression gate reminder:** Load increases only when both AMP and FCS ≥3. Compounds (squat, deadlift, press, row) require ≥4.

### AMP Score Special Cases

Some exercises have a structural ROM ceiling where AMP will never reach 4–5 regardless of technique. For these exercises, AMP is a **mobility progress indicator only** — not a load progression gate. FCS gates load.

| Exercise | AMP Rule | Reason |
|----------|----------|--------|
| Seated Tricep Rope Cross-Body Crunch | AMP = mobility indicator only; FCS gates load | Forward flexion ROM ceiling — user cannot reach the scoring threshold due to flexibility limits, not technique failure |

> **Tip:** Identify these through repeated sessions where AMP stays capped even as technique improves. Add them to this table and note in Notion. When reviewing: report the score for tracking but do not apply as a progression gate.

---

## Step 6: Discovery Update

List every exercise where `Discovered` was false (or absent) in Notion:
```
🔍 DISCOVERY THIS SESSION
- [Exercise Name] (ID: xxx) — [main muscle group]

Running total: [prior count + new] exercises seen | [library total - total] remaining
```

For running total: use the prior session's known count + count of new discoveries this session.
If no new discoveries: note it and flag whether minimum quota (1–2/session) was met.

---

## Step 7: Muscle Readiness Assessment

| Status | Criteria |
|--------|---------|
| 🔴 FATIGUED | Trained today or < 24h ago |
| 🟡 RECOVERING | Trained 24–48h ago, or scores degraded |
| 🟢 READY | > 48h rest, scores stable or improving |

List all major groups (Chest, Back, Shoulders, Arms, Core, Legs/Glutes) with status.
Flag any `bilateralBalanceScore` < 3 — note the weaker side. Ignore BBS=0 on unilateral exercises (data artifact).

---

## Step 8: Update Notion Exercise Library

**After writing the full review**, update Notion. Process every strength exercise in the record.
Skip: stretch (categoryId 5), cardio (categoryId 6), warm-up only (categoryId 3 with no scores).

### For a known exercise (Discovered = true, page exists):
```
Tool: Notion:notion-update-page
command: update_properties
page_id: [page ID from Step 2 lookup]
properties:
  Last Weight: [maxWeight from record, integer, lbs]
  Last Amp: [amplitudeStableScore]
  Last Force: [forceControlScore]
  Last Bal: [bilateralBalanceScore, omit if 0 on unilateral]
  Times Seen: [fetched Times Seen + 1; if null, set to 1]
  date:Last Seen:start: [session date YYYY-MM-DD]
  date:Last Seen:is_datetime: 0
```

Set `First Seen` only if it was null in the fetched record:
```
  date:First Seen:start: [session date YYYY-MM-DD]
  date:First Seen:is_datetime: 0
```

### For a new discovery (Discovered = false or page absent):

First check if the page exists but has `Discovered = false`:
- **If page exists**: update_properties, set `Discovered: "__YES__"` plus all score fields and `First Seen`
- **If page is absent**: create a new entry (look up muscle group and accessories from `library_cache_slim.json`):

```
Tool: Notion:notion-create-pages
parent: {data_source_id: "YOUR_EXERCISE_LIBRARY_DATA_SOURCE_ID"}
pages: [{
  properties:
    Exercise: "[title]"
    Library ID: "[actionLibraryGroupId as string]"
    Discovered: "__YES__"
    Category: "[from library_cache_slim]"
    Muscle Group: "[mainMuscleGroupName from library_cache_slim]"
    Accessories: "[from library_cache_slim]"
    outPosition: [from library_cache_slim]
    Last Weight: [maxWeight, integer]
    Last Amp: [score]
    Last Force: [score]
    Last Bal: [score, omit if unilateral]
    Times Seen: 1
    date:Last Seen:start: "[YYYY-MM-DD]"
    date:Last Seen:is_datetime: 0
    date:First Seen:start: "[YYYY-MM-DD]"
    date:First Seen:is_datetime: 0
}]
```

### Excluded exercises:
Maintain a list of exercises to skip silently (never update or create entries for these). Common reasons:
- No usable score data (e.g. exercise doesn't generate meaningful scores)
- User-requested exclusion

Add excluded exercises to this list as you encounter them. Flag in Notion Notes field with `[EXCLUDE]` prefix if an entry exists.

### Batching:
- `notion-create-pages` accepts up to 100 pages per call — batch all new discoveries in one call
- `notion-update-page` is single-page — prioritize exercises with PRs or score changes first

---

## Step 9: Update Carry-Forward Flags + Training Journal

**1. Update Current Workout Notes page** (`YOUR_WORKOUT_NOTES_PAGE_ID`):
- Replace `🚩 Carry-Forward Flags` section with updated flags from this review
- Include: technique holds, weight ceilings, deferred exercises, preference notes, physical flags
- Clear previous session walkthrough content

**2. Create Training Journal entry** (`YOUR_TRAINING_JOURNAL_DATA_SOURCE`):
- Session name, date, duration, exercise count, total sets
- Focus (muscle groups), PRs, Coach Notes (3–5 bullets), Recovery Rating

---

## Physical Context (customize for your user)

These are examples — replace with your own user's constraints:
- Forearm / wrist sensitivity — flag any exercise where wrist position may have contributed to poor scores
- Limited knee flexion — note if any lower-body exercise showed ROM limitations; kneeling positions may have a structural AMP ceiling
- Dominant-hand preference — check `bilateralBalanceScore` on unilateral exercises; note if non-dominant side is lagging
- Standing handle-bar exercises — expect lower baseline scores; weight expectations calibrated lower

---

## Tone & Format
- Lead with positives before limiters
- Be specific: name the exercise, name the score, give the cue
- Keep review scannable — use emoji section headers
- End with Next Session Focus block always

---

## Output Order (strict)
1. Session Summary block
2. PRs
3. Technique Limiters
4. Discovery Update
5. Muscle Readiness Assessment
6. Next Session Focus
7. Notion update confirmation (exercises updated, new entries created)
8. 🗑️ Clear Workout Notes reminder
