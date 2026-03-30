# Prompt: Bulk-Populate Recommended Stretches in Exercise Library

Use this prompt to ask an LLM (e.g. Claude) to suggest `Recommended Stretches` content for exercises in your Notion Exercise Library.

Run this after you have a populated Exercise Library — it works best when exercises already have Muscle Group and category data filled in.

---

## When to Use

- Initial library setup: pre-fill stretches for all discovered exercises in bulk
- After adding a batch of new discoveries from a session review
- When you want to audit or improve mobility prep coverage across a muscle group

---

## Prompt Template

Paste the following into a Claude conversation (not the Fitness Coach project — use a general Claude chat or a separate project):

---

```
I'm building a fitness coaching system that tracks exercises in a Notion database.
I need you to suggest pre-exercise mobility/stretch recommendations for each exercise below.

**Field format:**
Stretch Name · brief technique note · duration cue
(If multiple stretches apply, separate with a semicolon)

**Guidelines:**
- Recommend 1–2 stretches that directly address the primary ROM or mobility demand of the movement
- Keep technique notes short (5–10 words max) — just enough to identify setup
- Duration cue: typically 20–30s/side for static holds, or 8–10 reps for dynamic
- Use generic stretch names (no brand names)
- If the exercise is a stretch or warm-up itself, write: "None — movement is its own mobility prep"
- If the exercise is a machine-constrained isolation with no meaningful ROM limiter, write: "None — ROM not a limiting factor"

**Exercises to populate:**

| Exercise Name | Muscle Group | Category | Notes |
|---------------|-------------|----------|-------|
| [Exercise Name] | [e.g. Biceps] | [e.g. Training] | [optional notes] |
| [Exercise Name] | [e.g. Chest] | [e.g. Training] | |
...

For each exercise, return:
Exercise Name | Recommended Stretches value
```

---

## After Running

For each exercise returned, update the Notion Exercise Library:

```
Tool: Notion:notion-update-page
command: update_properties
page_id: [exercise page ID]
properties:
  Recommended Stretches: "[LLM output for that exercise]"
```

You can batch multiple updates in a single conversation turn by chaining `notion-update-page` calls.

---

## Notes

- Library IDs are optional in bulk-populated entries — add them when you can cross-reference to a specific stretch entry in your library, but they are not required
- Review LLM suggestions before writing — generic suggestions may not account for your specific physical constraints; override as needed
- The Fitness Coach will also populate this field opportunistically during session reviews when a specific stretch connection is identified from training experience
