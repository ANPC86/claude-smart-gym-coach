---
name: smart-gym-session-review
description: >
  Review and analyse a completed smart cable gym training session. Trigger on:
  "review my session", "analyse this workout", "post-session review", "how did I do",
  or whenever the user uploads a training record JSON file. Parses session data,
  identifies PRs, flags technique limiters, updates discovery count, assesses muscle
  readiness, and recommends the next session focus. Always use this skill when a
  training record is present — even if the user doesn't explicitly ask for a review.
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

- Page ID: `YOUR_CURRENT_WORKOUT_NOTES_PAGE_ID`
- Use `notion-fetch` to retrieve the page
- Extract every bullet under `## 📝 Current Workout Notes` or any `💬 **Your notes:**` fields populated mid-session
- Store these as **session context notes** — they represent in-session adjustments the user flagged that may not appear in the raw record data

**How to use the notes in the review:**
- If a note references a weight reduction mid-set → use the reduced weight as the actual working weight for that exercise, not the planned weight
- If a note flags fatigue, discomfort, or technique feel → include it in the Technique Limiters section with an `📓 User Note:` label
- If a note conflicts with what the record shows → trust the note over the record for weight, and flag the discrepancy
- Acknowledge the notes at the top of the review output under the Session Summary block (one line: `📓 Workout Notes retrieved: [count] note(s)`)

**After the full review is written**, add a reminder at the bottom:
> 🗑️ **Clear workout notes?** The notes above have been incorporated into this review. You may want to clear the Current Workout Notes section before your next session.

---

## Step 1: Parse the Training Record

The your gym app exports training records as JSON. Key fields per exercise:

| Field | Purpose |
|-------|---------|
| `date` / `time` / `duration` | Session metadata |
| `actionLibraryGroupId` | Links to library for exercise name lookup |
| `name` | Display name |
| `sets` → `reps`, `weight`, `mode`, `rest` | Volume data |
| `completionScore` | Overall quality 1–5 |
| `amplitudeStableScore` | ROM consistency 1–5 |
| `forceControlScore` | Force output control 1–5 |
| `bilateralBalanceScore` | Left/right balance 1–5 |
| `actionRating` | User's own rating if present |

> **Implementation note:** Training record JSON has a nested structure. Exercise data is in `detail.detail` (array). Use `maxWeight` field for peak load per exercise. Always cast `actionLibraryGroupId` to `str()` when matching against the tracker — the tracker stores IDs as strings.

Cross-reference `actionLibraryGroupId` against `Gym_Tracker_Master_CURRENT.json` to:
- Confirm exercise names
- Compare scores and weights to prior sessions
- Identify first-time exercises (discovery)

**Use Python exact-match lookup** for ID validation — do not rely on semantic/fuzzy search. False positives break the discovery count.

---

## Step 2: Session Summary

Lead with a concise header block:
```
Date:          [date + time]
Duration:      [X min]
Exercises:     [count]
Total Sets:    [count]
Est. Volume:   [total reps × weight where applicable, lbs]
```

---

## Step 3: PRs — Flag All Improvements

Check each exercise against tracker history. Report any of:
- **Weight PR** — heavier than all prior sessions
- **Reps PR** — more reps at same or greater weight
- **Score PR** — any technique score improved vs prior best
- **First Appearance** — never in records before (= discovery)

Format:
```
🏆 PRs THIS SESSION
- [Exercise Name]: weight 45→50 lb (+5 lb PR)
- [Exercise Name]: forceControlScore 3→4 (technique PR)
```

If no PRs: state clearly — "No weight or score PRs this session."

---

## Step 4: Technique Limiters

Flag every exercise where `amplitudeStableScore` OR `forceControlScore` < 3.

For each:
- Name the score and value
- State what it's blocking (progression gate requires ≥3, compounds ≥4)
- Give one specific, actionable cue tied to that score

**Progression gate reminder:** Load increases only when both scores ≥3. Compounds (squat, deadlift, press, row) require ≥4.

---

## Step 5: Discovery Update

List every exercise that was its first appearance in records:
```
🔍 DISCOVERY THIS SESSION
- [Exercise Name] (ID: xxx) — [main muscle group]
- [Exercise Name] (ID: xxx) — [main muscle group]

Running total: X exercises seen | Y remaining of 1,041
```

If no new discoveries: note it and flag whether discovery quota (min 1–2/session) was met.

---

## Step 6: Muscle Readiness Assessment

Based on today's volume, muscle groups trained, and scores:

| Status | Criteria |
|--------|---------|
| 🔴 FATIGUED | Trained today or < 24h ago |
| 🟡 RECOVERING | Trained 24–48h ago, or scores degraded vs baseline |
| 🟢 READY | > 48h rest, scores stable or improving |

List all major groups (Chest, Back, Shoulders, Arms, Core, Legs/Glutes) with status.

Note any asymmetry from `bilateralBalanceScore` < 3 — flag the weaker side.

---

## Step 7: Next Session Recommendations

Tie directly to what the data shows:
- Which muscle groups are ready for load progression (scores ≥3/4)
- Which techniques need a maintenance set at current weight before progressing
- Which muscle groups should rest or go light
- Suggested cardio modality (based on current patterns)
- Discovery priority: muscle groups or categories underrepresented in the tracker

Format:
```
NEXT SESSION FOCUS
✅ Ready to progress: [muscle groups]
⚙️  Hold weight, refine technique: [exercises]
🛌 Rest or go light: [muscle groups]
🔍 Discovery targets: [categories / muscle groups]
```

---

## Step 8: Update Tracker & Notion

After writing the review:

1. **Update `Gym_Tracker_Master_CURRENT.json`** — for every exercise in the record:
   - New exercise → create entry with all scores and metadata
   - Known exercise → update `last_maxWeight`, all score fields, `last_seen_iso`, increment `times_seen_in_records`, append to `notes`
   - Provide the updated file for the user to re-upload to the project

2. **Update Current Workout Notes page** (Notion):
   - Replace the `🚩 Carry-Forward Flags` section with flags derived from this review
   - Include: technique holds, weight ceilings, deferred exercises, preference notes, physical flags
   - Clear the previous session walkthrough content

3. **Create Training Journal entry** (Notion):
   - Session name, date, duration, exercise count, total sets
   - Focus (muscle groups), PRs, Coach Notes summary (3–5 bullets), Recovery Rating

---

## Physical Context (customize for your user)
These are examples — replace with your own user's constraints:
- Forearm / wrist sensitivity — flag any exercise where wrist position may have contributed to poor scores
- Limited knee flexion — note if any lower-body exercise showed ROM limitations
- Dominant-hand asymmetry — check `bilateralBalanceScore` on unilateral exercises, note if non-dominant side is lagging
- Standing handle-bar exercises — expect lower baseline scores; weight expectations are calibrated lower

---

## Tone & Format
- Lead with positives before limiters
- Be specific: name the exercise, name the score, give the cue
- Keep the full review scannable — use the emoji section headers above
- End with the Next Session Focus block always
- Coaching tone: instructional and insightful, not energetic or goofy

---

## Output Order (strict)
1. Session Summary block (with 📓 Workout Notes line if notes retrieved)
2. PRs
3. Technique Limiters (include 📓 User Note items here if applicable)
4. Discovery Update
5. Muscle Readiness Assessment
6. Next Session Recommendations
7. Tracker update instructions + Notion write summary
8. 🗑️ Clear Workout Notes reminder (always, at the very end)
