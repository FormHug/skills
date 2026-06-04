---
name: audit-formhug-form
description: |
  Review an existing FormHug form and propose (then apply) improvements.
  Use when the user asks to check/review/audit/improve a form ("what's
  wrong with my form", "why is nobody completing it", improve completion
  or conversion rate), or wants a pre-launch / pre-publish check before
  sharing a form.
---

# Audit a FormHug Form

A structured pre-flight review: gather the form's full state, walk a fixed checklist, report prioritized findings, and only change things the user approves. **Audit ≠ rewrite** — the user asked for findings, not a surprise redesign.

If FormHug MCP tools are not available, run the `setup-formhug-mcp` skill first.

## Step 1 — Gather state

Call all of: `get_form` (fields, scene, layout), `get_form_settings`, `get_form_theme`. Also `list_entries(limit=1)` — **whether entries exist changes what's safe to fix** (see Step 4). If the form's intent isn't clear from context (who fills this in? on what device? deadline?), ask before judging.

## Step 2 — Walk the checklist

### Fields
- **Type fit**: `short_text` where `email`/`phone`/`date`/`number`/`url`/`address` would validate; free-text where the answers are enumerable (→ `radio`/`dropdown`); ≳8 radio choices (→ `dropdown`); a run of same-scale rating questions (→ one `matrix_rating`/`likert`); separate province/city (or country/state) dropdowns (→ `cascade`).
- **Required discipline**: every `required: true` should have a reason the data is unusable without it. Required "comments/suggestions" fields are a classic abandonment driver. Count required fields; flag when most of the form is mandatory.
- **Question clarity**: double-barreled questions ("Are you satisfied with the product and the support?"), leading wording, missing `notes` help text where format/intent is ambiguous, unlabeled scale endpoints. For survey-type forms apply the `formhug-survey-methodology` skill for the deeper wording/scale review.
- **Choice lists**: overlapping or non-exhaustive options, missing `is_other: true` escape, inconsistent granularity.

### Structure
- Length: >~10 questions on one unbroken `classic` page → suggest `page_break`s plus the progress bar (`update_form_settings` → `submission_options.show_progress`).
- Order: engaging/easy questions first, identity & sensitive questions last, related questions grouped under `description` section headers.
- An intro `description` telling respondents what this is and how long it takes.

### Settings (`get_form_settings`)

(For the exact `update_form_settings` argument shapes — `after_submission`, `availability`, `submission_options` — follow `formhug://guide`; don't guess key names.)
- `after_submission` unset → respondents get a generic end; suggest a thank-you message, redirect, or (quiz/assessment) score report.
- Time-bound collection without `availability.schedule` / `entries_limit`; or a closed form the user thinks is open (`manually_closed`).
- `locale` mismatched with the audience language; `timezone` wrong for date interpretation.
- Sensitive forms without `access_control.access_password` when access should be restricted.

### Theme & privacy
- Header image irrelevant to the topic; palette/typography issues → check against `formhug://themes` guidance.
- PII collected without need (asking phone + email + address when one channel suffices); sensitive fields not marked `private` (private hides them from public reports).

## Step 3 — Report findings

Present as a prioritized list, each item: severity, the specific field/setting, why it hurts, and the concrete fix.

```
🔴 Blocker — field_7 "Contact info" is short_text: no validation, you'll get unusable
   mixed phone/email strings → replace with a `phone` (required) field
🟡 Improve — 12 of 14 fields are required → keep 5, make opinions optional
🟢 Fine — after_submission thank-you message is set
```

End with: "Want me to apply some or all of these?" — then apply **only what's approved**.

## Step 4 — Apply fixes safely

The editing mechanics — `update_fields` shallow-merge plus wholesale nested-array (`choices`) replacement, `page_break` placement (0 or ≥2, first field must be one), and a field's `type` being fixed at creation — are all in `formhug://guide`; re-read it rather than guessing key names. What the **audit** adds is the entry-safety lens: Step 1's `list_entries` told you whether responses already exist, and that decides what's safe to touch.

- **No entries yet** — edit freely; there's nothing to orphan.
- **Has entries** — be conservative:
  - **Additive changes stay safe**: `add_fields`, `notes` edits, theme, settings.
  - **Editing a choice list** orphans the submitted values of any choice you drop — and because `update_fields` replaces the whole `choices` array, omitting an existing choice silently deletes it. Resend every choice with its `api_code`.
  - **Changing a field's type** is `remove_field` + `add_fields`, and removing a field **discards its collected answers**. Warn explicitly, get a second confirmation, and usually prefer to add the new field and retire the old one after the campaign.
- Re-run `preview_form` after changes and surface the public URL `https://formhug.ai/f/<token>` + edit URL `https://formhug.ai/forms/<token>/edit`.

## Common mistakes

| Mistake | Fix |
|---|---|
| Auditing fields only | Settings, theme, and privacy are where launches break |
| Rewriting the form unprompted | Findings first; change only what's approved |
| `update_fields` with just the changed choice | Resend the full `choices` array with api_codes |
| Removing/retyping fields on a live form casually | Warn that entry data is lost; prefer additive |
| "Too long" verdict with no evidence | Count fields/required/pages; cite the numbers |
| Generic advice ("improve UX") | Every finding names a field/setting and a tool call |
