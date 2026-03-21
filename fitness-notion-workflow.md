# Fitness Coach — Notion Workflow
> Defines when and what Claude writes to Notion. Behavioural rules for the AI coach.

## Overview

Notion is the fitness journal and reference system. Claude writes to it; the user reads from it. The user rarely writes directly in Notion — they tell Claude what to record, and Claude writes it.

See `notion-reference.md` for database IDs, schemas, and tool call examples.

---

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

**These notes are required context for an accurate review.** Do not write PRs, technique flags, or tracker updates without reading them first — user mid-session feedback can change the interpretation of scores and weights.

### Before Every Workout Design
**Always fetch the Current Workout Notes page before starting intake.**
The `🚩 Carry-Forward Flags` section at the top contains technique holds, deferred exercises, and preference notes from the last reviewed session. Apply all flags before generating the workout.

---

## When to Write to Notion

### After a Session Review

**1. Update the Current Workout Notes page**:
- Replace the `🚩 Carry-Forward Flags` section with updated flags derived from the review
- Include: technique holds, weight ceilings, deferred exercises, preference notes, physical flags
- Clear the previous session's exercise walkthrough content (it has been reviewed)
- This page is the handoff document to the next workout design — it must be current

**2. Create a Training Journal entry** with:
- Session name (e.g., "Upper Body — Jan 15")
- Session Date, Duration, Exercise count, Total Sets
- Focus (muscle groups trained)
- PRs (any new personal records)
- Coach Notes (review summary — 3–5 bullet points max)
- Recovery Rating (based on muscle readiness assessment)
- Leave "My Notes" empty — that's for the user to fill in

### After a Workout Design
**Write the full session walkthrough to the Current Workout Notes page**:
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

---

## What NOT to Put in Notion
- Raw set/rep/weight data (→ belongs in the tracker JSON or a separate pipeline)
- Full session review output (→ Training Journal gets a summary, not the full review)
- Exercise library master data (→ stays in JSON project files)
- Coaching rules and programming logic (→ stays in project system prompt / skills)

---

## Workflow Pattern
```
User tells Claude → Claude responds in conversation → Claude writes to Notion
                                                       (user never needs to ask "update Notion")
```

After session reviews and exercise-specific discussions, proactively offer to update Notion. Example: "I've noted that exercise as a Mainstay in your Playbook with coaching cues. Want me to adjust anything?"
