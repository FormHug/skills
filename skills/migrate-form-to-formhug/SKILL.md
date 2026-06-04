---
name: migrate-form-to-formhug
description: |
  Recreate an existing form, survey, or questionnaire in FormHug.
  Use when the user wants to migrate/convert/import a form from another
  platform (Google Forms, Typeform, Jotform, Microsoft Forms, SurveyMonkey,
  Tally, Jinshuju, Wenjuanxing, Tencent Questionnaire) or rebuild one from
  a document — Word, PDF, Excel, Markdown, a screenshot/photo of a paper
  form, or a web page. Triggers include "move this survey to FormHug",
  "make this an online form", "rebuild this questionnaire".
---

# Migrate a Form to FormHug

Turn an existing form (from another platform or a document) into a faithful FormHug form. The core discipline: **inventory first, create second, verify third**. Never compose `create_form` directly from a skim of the source — extraction mistakes silently drop questions.

If FormHug MCP tools (`create_form`, `list_forms`, …) are not available, run the `setup-formhug-mcp` skill first.

## Step 1 — Acquire the source

| Source | How to read it |
|---|---|
| File (Word/PDF/Excel/Markdown) | Read it directly; for PDF read every page |
| Screenshot / photo | Read the image; ask for more screenshots if the form is cut off |
| Live form URL | Fetch the page; if it's behind login or renders client-side, ask the user for screenshots or an export instead |
| Another platform's account | Prefer the platform's export (e.g. Google Forms → linked Sheet header row shows all questions) over scraping |

If parts are illegible or ambiguous, ask — don't guess silently.

## Step 2 — Build a question inventory

Before any tool call, extract every item into a flat list:

```
#  | label             | source type        | required | choices / scale              | notes
1  | Your name         | short answer       | yes      | —                            |
2  | Satisfaction      | linear scale 1–5   | yes      | 1=Very poor … 5=Excellent    | endpoint labels
3  | Anything else?    | paragraph          | no       | —                            |
```

Also capture: section headers, page boundaries, intro/description text, any quiz scoring, and any conditional logic ("show Q5 only if Q4 = Yes"). When the source doesn't mark a question required/optional, default to **optional** (most platforms' default) and flag the assumption in the inventory.

**Show the inventory to the user and confirm** before creating — this is the cheap moment to catch a missed question or a wrong required flag.

## Step 3 — Map source types to FormHug types

Read `formhug://field-types` first, and the per-type resource for any advanced field you use — **never guess attribute names** (e.g. `likert` takes `statements[]` + `choices[]`, not rows/columns). Common mappings:

| Source concept | FormHug field |
|---|---|
| Short answer / single line | `short_text` (use `email` / `phone` / `url` / `number` / `date` when the question is actually one of those — they validate) |
| Paragraph / long answer | `long_text` |
| Multiple choice (single) | `radio`; switch to `dropdown` when there are many choices (≳8) |
| Checkboxes (multi) | `checkbox` |
| Dropdown | `dropdown` |
| Linear scale | `rating` (3–10 levels); 0–10 "would you recommend" → `nps` |
| Multiple-choice grid (one scale, many rows) | `likert` (`likert_choice_style: "single"`) |
| Rating grid (stars per row) | `matrix_rating` |
| Grid with mixed column types | `matrix` |
| Ranking | `ranking` |
| File upload | `attachment` |
| Date / time | `date` (set `precision`) / `time` |
| Name / address | `name` / `address` (structured — better than free text) |
| Section title + description | `description` field (`label` = heading, `notes` = body) |
| Page boundary (source actually paginates) | `page_break` — a section header that does NOT start a new page stays a `description`; ask the user if it's unclear whether they want pages or inline headers |
| "Other: ___" option | choice with `is_other: true` |
| Graded quiz / scored test | `scene: "quiz"` or `"assessment"` — follow the guide's "Creating a Quiz or Assessment" section; reconcile the score scale (max possible = sum of each field's highest `score`) against any pass/fail threshold the source states |

**`page_break` placement rules** (server-enforced): a form has either 0 page_breaks or ≥2, and when present the FIRST field must be a page_break. So a 3-page source form becomes: `page_break, …page1 fields, page_break, …page2 fields, page_break, …page3 fields`.

## Step 4 — Declare what won't migrate

Check the source for features FormHug MCP can't reproduce, and tell the user **before** creating:

- **Conditional logic / branching** — not configurable via MCP; list the rules so the user can recreate them (or accept a linear form)
- **Signature values** — a `signature` field can exist, but old signature data can't be imported
- **Platform-specific widgets** (payment, calculated fields, embedded video) — flag them and propose the closest substitute

An honest "these 2 things need manual follow-up" beats a migration that silently lost behavior.

## Step 5 — Create and verify

1. One `create_form` call with the full field list (`header_image`: 1–2 English keywords matching the form's topic; keep the source's title and intro text — intro becomes `description` or the form's `description`).
2. `preview_form` and **diff against the inventory**: same question count, same order, required flags match, every choice list complete (count the choices!).
3. Fix gaps with `add_fields` / `update_fields` (remember: `update_fields` replaces nested `choices` arrays wholesale — include every existing choice with its `api_code`).
4. Carry over behavior with `update_form_settings`: locale (Chinese source → `locale`), timezone, deadline/entry caps (`availability`), thank-you message (`after_submission`).
5. Surface the public URL (`https://formhug.ai/f/<token>`) and edit URL (`https://formhug.ai/forms/<token>/edit`), plus the manual follow-up list from Step 4.

## Step 6 (optional) — Migrate old responses

If the user has exported responses (CSV/Excel) and wants them in FormHug: read `formhug://entry-values` for per-type value shapes, map columns to field `api_code`s, then `submit_entry` per row. Watch for: choice values must be choice `api_code`s (not display text), `phone` is an object, `date` format follows the field's `precision`. Dry-run one row, `get_entry` to verify, then proceed. Tell the user submitted entries get fresh timestamps — original submission times can't be preserved.

## Common mistakes

| Mistake | Fix |
|---|---|
| Composing `create_form` straight from the source without an inventory | Inventory + user confirmation first; diff after creation |
| Mapping a 1–5 scale with labeled endpoints to `radio` | Use `rating`; put endpoint labels in `notes` if needed |
| Flattening a grid question into N separate radios | Use `likert` / `matrix_rating` — one field, same rows |
| Adding exactly one `page_break`, or none first | 0 or ≥2, and the first field must be a `page_break` |
| Silently dropping conditional logic | Declare it in Step 4 |
| Guessing field attribute keys (`rows`, `columns`, `levels`, …) | Read `formhug://field-types/<type>` before composing |
| Importing old responses with display text as choice values | Use choice `api_code`s; read `formhug://entry-values` |
