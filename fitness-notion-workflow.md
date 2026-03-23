# Fitness Coach — Notion Workflow
> **Last updated**: 2026-03-23

## Overview

Notion is the fitness journal and reference system. Claude writes to it; the user reads/consumes from it. The user will rarely write directly in Notion — they'll tell you what to record, and you write it.

See `notion-reference.md` for database IDs, schemas, and tool call examples.

## When to Read from Notion

### Before Every Session Review
**Always fetch the Current Workout Notes page before writing the review.**
Page ID: `YOUR_WORKOUT_NOTES_PAGE_ID`

This page contains user notes written mid-session in the `💬 **Your notes:**` fields under each exercise. These notes capture:
- How an exercise felt (too heavy, too light, uncomfortable)
- Physical flags (pain, strap discomfort, wrist issues)
- User ratings and preferences ("this is a good one", "want to see again")
- Exercises that were skipped and why
- Any deviations from the planned weights

**These notes are required context for an accurate review.** Do not write PRs, technique flags, or Exercise Library updates without reading them first — user mid-session feedback can change the interpretation of scores and weights.

### Before Every Workout Design
**Always fetch the Current Workout Notes page before starting intake.**
The `🚩 Carry-Forward Flags` section at the top contains technique holds, deferred exercises, and preference notes from the last reviewed session. Apply all flags before generating the workout.

---

## When to Write to Notion

### After a Session Review

**1. Update the Notion Exercise Library** (`YOUR_EXERCISE_LIBRARY_DATA_SOURCE`):
- For every strength exercise in the session record, update its Exercise Library entry
- Write: Last Weight, Last Amp, Last Force, Last Bal, Last Seen
- Set Discovered = `__YES__` and First Seen for new discoveries
- See `smart-gym-session-review-SKILL.md` Step 8 for full write-back pattern

**2. Update the Current Workout Notes page** (`YOUR_WORKOUT_NOTES_PAGE_ID`):
- Replace the `🚩 Carry-Forward Flags` section with updated flags derived from the review
- Include: technique holds, weight ceilings, deferred exercises, preference notes, physical flags
- Clear or archive the previous session's exercise walkthrough content (it has been reviewed)
- This page is the handoff document to the next workout design — it must be current

**3. Create a Training Journal entry** with:
- Session name (e.g., "Upper Body — March 22")
- Session Date, Duration, Exercise count, Total Sets
- Focus (muscle groups trained)
- PRs (any new personal records)
- Coach Notes (your review summary — keep concise, 3–5 bullet points max)
- Recovery Rating (based on your muscle readiness assessment)
- Leave "My Notes" empty — that's for the user to fill in

### After a Workout Design
**Write the full session walkthrough to the Current Workout Notes page** (`YOUR_WORKOUT_NOTES_PAGE_ID`):
- Replace the previous session content (below the `🚩 Carry-Forward Flags` section) with the new session
- Include: session header, all 5 phases with per-exercise coaching cues, `💬 **Your notes:**` field after each exercise (left blank for user to fill in mid-session)
- The Carry-Forward Flags section is read-only during design — do not modify it here; it gets updated during session review
- Format should match the established page structure so the user can follow along on their device during the workout

### When the User Gives Exercise Feedback
Update or create an **Exercise Playbook** entry:
- "I like this exercise" → Status: Mainstay or Regular
- "This felt bad / hurts my wrist" → Status: Avoid + My Notes
- "ROM was limited" → My Notes with specifics
- "Load was too light/heavy" → Load Notes
- "I want this for golf" → add Golf Relevant tag (or your own sport/goal tag)
- Always preserve existing content — append, don't overwrite My Notes or Load Notes

### When You Give Coaching Cues
Update the **Exercise Playbook** entry for that exercise:
- Posture, breathing, tempo → Coaching Cues
- Common errors to watch for → Common Mistakes
- smart gym-specific setup tips → smart gym Settings

### When Goals Are Discussed
Create or update **Fitness Goals** entries:
- New goal mentioned → create with Status: Active or Planned
- Progress update → update Current field
- Goal achieved → Status: Achieved
- Approach changes → update Approach field

### When Body Metrics Are Reported
Add a **Body Metrics** entry:
- Monthly weigh-in, body fat %, resting HR
- Phase, sleep quality, energy level if mentioned
- Notes for anything contextual

## What NOT to Put in Notion
- Raw set/rep/weight data per individual rep (→ belongs in a database pipeline, e.g. Postgres via automated sync)
- Full session review output (→ Training Journal gets a summary, not the full review)
- Coaching rules and programming logic (→ stays in project system prompt / skills)

## What Lives in Notion (not JSON files)
- **Exercise history** (last weight, scores, discovered status, first/last seen) → **Exercise Library DB**
- **Session history** (dates, focus, PRs, coach summary) → **Training Journal DB**
- **Per-exercise coaching notes and preferences** → **Exercise Playbook DB**
- **Current session walkthrough** → **Current Workout Notes page**
- **Active carry-forward flags** → **Current Workout Notes page** (top section)

## Workflow Pattern
```
User tells Claude → Claude responds in conversation → Claude writes to Notion
                                                       (user never needs to ask "update Notion")
```

After session reviews and exercise-specific discussions, proactively offer to update Notion. Example: "I've noted that exercise as a Mainstay in your Playbook with the coaching cues. Want me to adjust anything?"
