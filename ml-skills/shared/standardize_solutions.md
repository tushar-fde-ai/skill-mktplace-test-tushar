# Standardization Checklist for ML Solution Skills

This file records every structural change made to bring an ml-skills solution into alignment with the NBA Engagement Scores reference implementation. When standardizing a new solution (e.g. RFM, NBP), apply each item below to that solution's files.

---

## Change 1: Add Phase Summary and Phase-by-Phase context to `<solution>/SKILL.md`

**Problem:** A lookup table ("Pick the right reference for the task") caused the skill to jump directly into a reference file mid-stream, bypassing Phase 1 data exploration entirely.

**Fix:** Replace the lookup table with two sections that mirror NBA's `SKILL.md` structure:

1. `## Phase Summary` — a 6-row table listing all phases and what happens in each.
2. `## <Solution>-Specific Context by Phase` — one `### Phase N` subsection per phase, each containing:
   - What to do in that phase (explicit steps, not just "read file X")
   - Which reference files to read and when
   - Solution-specific values (repo URL, project name, config file path, Confluence folder name, title variants)

The reference lookup table content is not deleted — its file paths are embedded inside the correct phase subsection where they are actually needed.

**Reference:** Compare `nba-scores/SKILL.md` sections `## Phase Summary` and `## NBA-Specific Context by Phase` as the canonical template.

**Files to edit:** `<solution>/SKILL.md`

---

## Change 2: Phase 1 must be data exploration, not requirements questions

**Problem:** Some solutions started Phase 1 by asking the user business questions (ID Unification status, conversion definition, etc.) before any data had been explored.

**Fix:** Phase 1 in `<solution>/SKILL.md` must always be data exploration first, following these steps in order:

1. List every table in the customer database via `tdx tables`
2. Classify each table (activity/touchpoint vs profile/dimension vs system/output)
3. Apply solution-specific classification (e.g. for MTA: pageviews / email / sales / orders)
4. Run any required discovery queries (e.g. UTM coverage check for web tables)
5. Present findings to user for confirmation via `AskUserQuestion` (see Change 3)
6. Note down confirmed table details for `input_params.yml`

Business questions (sink database, conversion definition, model scope, etc.) belong in Phase 2 requirements gathering — not Phase 1.

**Files to edit:** `<solution>/SKILL.md` Phase 1 section

---

## Change 3: All questions to user must use `AskUserQuestion` tool, never plain text lists

**Problem:** Questions during data exploration and requirements gathering were presented as plain text lists in chat, requiring the user to type free-form answers.

**Fix:** Add the following instruction block wherever questions are asked of the user — both in `<solution>/SKILL.md` (Phase 1 table confirmation step) and in `<solution>/workflow-setup/references/requirements_doc.md` (top of the initial questions step):

> **How to ask:** Never present these as a plain text list. Always use the `AskUserQuestion` tool so each question renders as an interactive selector. Group into batches of up to 4 questions per call (tool limit). For each question provide 2–4 pre-populated answer options — mark the recommended default with `(Recommended)` — plus the implicit "Other" option that lets the user type a custom answer.

For the Phase 1 table confirmation step specifically, provide pre-populated options such as "Confirm as listed" and "Exclude one or more tables" with "Other" for custom input.

**Files to edit:**
- `<solution>/SKILL.md` — Phase 1 Step that presents table list to user
- `<solution>/workflow-setup/references/requirements_doc.md` — top of the initial questions section

---

## Change 4: Phase 6 — remove Parent Segment step for solutions that do not add PS attributes

**Problem:** The Phase 6 template from `shared/SKILL.md` includes a Parent Segment attributes step. Not all solutions add attributes to a Parent Segment.

**Fix:** For solutions where no Parent Segment update is required, replace the two-step Phase 6 structure with a single step going straight to Confluence documentation pages:

```markdown
### Phase 6: Customer Documentation

**Step 1 — Confluence documentation pages.**
Read `../shared/customer_docs_pattern.md` for the 5-page set.
...
```

**Solutions where this applies:**
- **MTA** — attribution scores are output tables only; no PS attribute addition needed.

**Solutions where the full two-step Phase 6 applies (keep Parent Segment step):**
- NBA Engagement Scores, RFM, NBP — all add score columns to a Parent Segment.

**Files to edit:** `<solution>/SKILL.md` Phase 6 section

---

## Solutions Status

| Solution | Change 1 | Change 2 | Change 3 | Change 4 |
|---|---|---|---|---|
| NBA Engagement Scores | ✅ Reference implementation | ✅ Reference implementation | ✅ Reference implementation | N/A (keeps PS step) |
| MTA Journey Analytics | ✅ Done | ✅ Done | ✅ Done | ✅ Done (PS step removed) |
| RFM | ⬜ Pending | ⬜ Pending | ⬜ Pending | N/A (keeps PS step) |
| NBP | ⬜ Pending | ⬜ Pending | ⬜ Pending | N/A (keeps PS step) |
