---
name: formhug-survey-methodology
description: |
  Survey design methodology for FormHug questionnaires. Use when composing
  or reviewing opinion/research surveys — satisfaction (CSAT), NPS,
  employee engagement, market/user research, academic questionnaires —
  or when the user asks for a "professional", "scientific", or "unbiased"
  survey, asks how to word questions, choose scales, or how long a survey
  should be. Not for data-entry forms (registration, orders, applications).
---

# Survey Methodology on FormHug

Opinion data is only as good as the instrument. These are the wording, scale, and structure rules to apply whenever a FormHug form measures attitudes rather than collects facts — plus the FormHug field that implements each pattern.

## Question wording

- **One concept per question.** "Are you satisfied with the product and the support?" is two questions — split it. Test: if someone could reasonably answer "satisfied with one, not the other", it's double-barreled.
- **Neutral phrasing.** "How much do you love the new feature?" presumes loving. Ask "How would you rate the new feature?" with a balanced scale.
- **Concrete reference periods.** "recently" → "in the past 30 days". Vague time = incomparable answers.
- **Respondent's vocabulary.** No internal jargon, feature codenames, or abbreviations; spell out on first use in the field's `notes`.
- **Avoid negations** ("Do you disagree that…"), and never double negatives.

## Scale design

- **5 or 7 points** for agreement/satisfaction scales. Fewer loses resolution; more adds noise.
- **Label every point** when possible (a `likert` field's `choices` are fully labeled); for `rating`-style scales label at least the endpoints (`matrix_rating` dimension supports `minimum_ratings_display_text` / `maximum_ratings_display_text`).
- **Balanced and symmetric**: as many positive as negative options around a midpoint ("Very dissatisfied / Dissatisfied / Neutral / Satisfied / Very satisfied").
- **Odd vs even points**: odd (with a neutral midpoint) is the default; even (forced choice) only when a deliberate stance is required — and acknowledge it skews data.
- **One direction throughout the survey.** If 5 = best on one question, 5 = best everywhere. Reversed items confuse more than they catch.
- **NPS is a fixed instrument**: 0–10, the standard wording "How likely are you to recommend … to a friend or colleague?" — use FormHug's `nps` field, don't rebuild it with `rating`. Employee version (eNPS): "…recommend this company as a place to work?".

## Research need → FormHug field

| Need | Field |
|---|---|
| Agreement battery (many statements, one scale) | `likert` — `statements[]` are the rows, `choices[]` the fully labeled scale, `likert_choice_style: "single"` |
| Rate several items on the same numeric scale | `matrix_rating` with endpoint labels |
| Overall satisfaction (CSAT) | `rating` (5-point) or 5-point `likert` |
| Recommendation intent | `nps` |
| Forced prioritization | `ranking` — keep to ≤7 items, ranking long lists is unreliable |
| Single categorical answer | `radio` (visible options read more reliably than `dropdown`) |
| "Select all that apply" | `checkbox` — order options logically, not by your hypothesis |
| Open exploration ("Anything else you'd like to tell us?") | `long_text`, optional, near the end |
| Sensitive demographics | `radio`/`dropdown` with a "Prefer not to say" choice, `private: true`, never required |

## Structure & length

1. **Intro `description` field** (`label` = heading, body text in `notes`): purpose, time estimate, anonymity statement. Honesty here improves completion and answer quality.
2. **Funnel order**: engaging/general questions first → specific/effortful middle → demographics and sensitive items last.
3. **Group by topic** with `description` section headers; one matrix/likert per topic beats scattered repeats.
4. **Length budget**: target ≤5 minutes ≈ 15–20 simple questions (a likert statement counts as a question). Past 10 minutes, expect heavy drop-off and straightlining. Cut any question without a planned analysis use — "nice to know" is not a use.
5. **Pagination**: on `classic` layout, page per topic — a form has 0 or ≥2 `page_break` fields, and when present the form's FIRST field must be one. Enable the progress bar (`submission_options.show_progress` via `update_form_settings`).
6. **Required restraint**: required only for questions the analysis cannot survive without. A required opinion invites a fabricated one.

## Response quality safeguards

- `is_other: true` escape options on choice lists that might not be exhaustive.
- Don't mark long likert/matrix fields `required` (it applies to the whole grid) — forced grids breed straightlining (same answer down the column). Since reversed items are also banned, the straightlining defenses are: short grids, optional answering, and checking for flat-line response patterns during analysis.
- **Anonymity must survive cross-tabulation**: in a small organization, department × tenure × seniority can re-identify individuals. Keep demographics coarse (broad buckets), collect only the ones with a planned analysis use, and never promise anonymity the data can break.
- Mutually exclusive, collectively exhaustive choice lists; ranges that don't overlap ("18–25 / 26–35", not "18–25 / 25–35").
- **Pilot before launch**: `preview_form`, fill it once yourself via the public URL, time it, then have 2–3 real people try it. Update the intro's time estimate with the measured time.

## When NOT to apply

Registration, ordering, application, sign-up, and data-entry forms optimize for speed and validation (right field types, minimal friction) — not for measurement neutrality. For those, the `audit-formhug-form` checklist is the right lens. For scored forms (quiz/assessment), follow the guide's "Creating a Quiz or Assessment" section.

## Common mistakes

| Mistake | Fix |
|---|---|
| Double-barreled questions | One concept per question; split |
| 1–10 unlabeled scales | 5/7 points, labeled, symmetric |
| NPS rebuilt as a 1–5 rating | Use the `nps` field with standard wording |
| Demographics first | Last — they're the most-abandoned-at questions |
| Everything required | Required only what analysis can't survive without |
| 30-question "nice to know" survey | Budget to ≤5 min; every question needs an analysis use |
| Launching without a pilot | Fill it yourself, time it, fix, then share |
