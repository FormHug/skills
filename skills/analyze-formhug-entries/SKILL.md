---
name: analyze-formhug-entries
description: |
  Analyze the responses collected by a FormHug form. Use when the user
  asks to summarize/report on submissions, compute statistics ("what's
  the average satisfaction score", "how many chose X", NPS score,
  response trends), find themes in open-text answers, export entries,
  or "show me what people answered". Also use before answering any
  question that requires reading entry data.
---

# Analyze FormHug Entries

Produce a faithful analysis of a form's submissions. The two failure modes to avoid: **analyzing raw choice api_codes as if they were labels**, and **computing statistics over a partial page of entries without saying so**.

If FormHug MCP tools (`list_entries`, …) are not available, run the `setup-formhug-mcp` skill first.

## Step 1 — Locate the form and build the field map

If you only have a form name, `list_forms` and match by name — when several forms match, ask the user which one (don't pick silently).

Then call `get_form` and build a lookup before touching entries:

- `api_code → label` and `type` for every field
- for choice fields (`radio`/`checkbox`/`dropdown`/`likert`/`cascade`/`ranking`): nested `choice api_code → choice label`
- for `matrix`/`matrix_rating`/`likert`/`table`: `statement/dimension api_code → label`

Every value you report to the user must be translated through this map — entry values store **api_codes, never display text**.

## Step 2 — Fetch all entries (paginate!)

```
list_entries(form_token, limit=100)            → records + next_cursor
list_entries(form_token, limit=100, cursor=…)  → repeat until next_cursor is null
```

- Default `limit` is 20 — always raise it to 100 for analysis.
- Loop until `next_cursor` is absent. **Never compute statistics from page 1 only.**
- Very large datasets (thousands of entries): tell the user the count, then either analyze everything (it just takes more calls) or agree on a sample/time window — but state explicitly which one you did.
- For big jobs in a coding environment, consider writing a small script that pages through and aggregates, instead of holding every entry in context.

## Step 3 — Aggregate per field type

`formhug://entry-values` has the exact JSON shape for every type — read it instead of guessing. Decode each value through the Step 1 map, then aggregate by what the type *means*, which is where analysis goes wrong:

| Type | How to aggregate |
|---|---|
| `radio`/`dropdown` | frequency over the choice; bucket `is_other` free-text answers separately |
| `checkbox` | multi-select — percentages sum to >100%; report "% of respondents", not "% of answers" |
| `rating` | mean **and** distribution (a 3.0 from polarized 1s and 5s ≠ a uniform 3) |
| `nps` | NPS = %Promoters(9–10) − %Detractors(0–6) over those who answered, reported as an integer −100..100 — never an average of scores, never a % |
| `likert` | aggregate **per statement**, never one number for the whole grid; map the scale to 1..N for means |
| `matrix_rating` | per-statement mean and distribution |
| `phone`/`address`/`location` | identity data — list it, don't aggregate |
| skipped (key absent, `null`, or checkbox `[]`) | exclude from that question's base; report a per-question answer count |

## Step 4 — Aggregate honestly

- **Per-question response base**: "n=42 of 50 answered" — optional questions have different bases.
- **Choice questions**: frequency table sorted by count, with percentages of that question's base.
- **Numeric/rating**: mean, median, min–max, and the distribution (a 3.0 mean from polarized 1s and 5s ≠ a uniform 3).
- **Open text**: group into themes, count mentions per theme, and quote 1–3 verbatim answers per theme (mark them as quotes). Don't fabricate sentiment percentages from a handful of texts.
- **Time trend**: entries carry creation timestamps — daily/weekly volume reveals campaign spikes.
- **Small samples**: below ~30 responses, report counts, not percentages-as-conclusions ("3 of 7" — not "43% of users").
- **Cross-tabs** (e.g. satisfaction by user type) only when the user asks or an obvious segmentation exists — state the segment sizes.

## Step 5 — Deliver the report

Structure: ① overview (total entries, date range, per-question answer counts) → ② per-question results in form order → ③ cross-cutting findings (the 3–5 things worth acting on) → ④ caveats (sample size, skipped questions, sampling). Use tables for distributions. Offer the entries page — `https://formhug.ai/forms/<token>/entries` — for FormHug's built-in live charts.

If the user wants raw data: write a CSV/Excel with one row per entry, columns labeled with field labels (not api_codes), choice values translated.

## Common mistakes

| Mistake | Fix |
|---|---|
| Reporting choice api_codes (`"choice_3"`) as answers | Translate through the Step 1 field map |
| Stats from the first page of `list_entries` | Paginate with `next_cursor` until exhausted |
| Checkbox percentages forced to sum to 100% | They're multi-select — use "% of respondents" |
| NPS as average of scores or as a percentage | %Promoters − %Detractors, an integer from −100 to 100 |
| One aggregate for a whole likert/matrix field | Aggregate per statement |
| Treating skipped answers as negative/zero | Exclude; report per-question n |
| "43% of users" from 7 responses | Report counts when n is small |
