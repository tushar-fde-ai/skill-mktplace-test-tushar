---
name: custom-analytics-agent-setup
description: |
  Analytics-specific Phase 2 (first push) and Phase 4 (re-push after customer reviews) file-edit checklist. Push mechanics live in ../../shared/push_pattern_fresh_project.md.
---

# Custom Analytics Agent — Agent Setup

Analytics-specific overlay on the shared fresh-project push pattern. The push mechanics (`tdx llm project create`, `tdx agent push -y`) live in `../../shared/push_pattern_fresh_project.md`. This file specifies *which files to edit* for an analytics-agent engagement.

## Routing

| Task | Action |
|------|--------|
| The actual push command sequence | Read `../../shared/push_pattern_fresh_project.md` |
| Render the Phase 1d Confluence requirements doc body | Read `references/requirements_doc.md` |
| Distill the Phase 3 KBs from the inference bundle + Phase 4 re-distill from customer-edited doc | See custom-audience-agent's `business_context_template.md` for the distillation rules — same approach (strip markers, hedge phrases, convert to terse imperative). Apply to all 3 customer-specific KBs. |
| Generate test cases, run `tdx agent test`, manage two-round eval | Read `references/eval.md` (and `../../shared/test_cases_pattern.md`) |

## Phase 2 — First Push

Follow `../../shared/push_pattern_fresh_project.md` end-to-end. **Apply the analytics-specific file edits below between Step 3 (set tdx.json) and Step 6 (push) of the shared push pattern.**

### Files to edit (between shared Step 3 and Step 6)

| File | Change |
|------|--------|
| `tdx.json` | `llm_project` → `<Customer> Analytics Agent` (replaces the `<Customer> Analytics Agent` placeholder shipped in the template) |
| `knowledge_bases/master_database.yml` | `database: <database>` → set to the customer's TD database name (replaces the `<database>` placeholder shipped in the template) |
| `Analytics Agent/agent.yml` | Verify `model: claude-4.5-sonnet` (template ships with this; flip if it ever drifts) |
| `data_source_finder/agent.yml` | Verify `model: claude-4.5-sonnet` |
| `Analytics Agent/prompt.md` | Optional: vertical-specific tone tweaks. The shipped prompt is vertical-agnostic; only edit if the customer's domain calls for unusual constraints. |
| `knowledge_bases/data_dictionary.md` | **Phase 2: don't edit.** Leave as the shipped stub. Phase 3 distills from the inference bundle. |
| `knowledge_bases/business_context.md` | **Phase 2: don't edit.** Leave as the shipped stub. Phase 3 distills from the inference bundle. |
| `knowledge_bases/sql_templates.md` | **Phase 2: don't edit.** Leave as the shipped stub. Phase 3 distills from the inference bundle if the customer provided templates. |
| `knowledge_bases/plotly_instructions.md` | **Don't edit.** Generic chart rules — same for every customer. |
| `knowledge_bases/react_dashboard_instructions.md` | **Don't edit.** Generic dashboard rules — same for every customer. |

### No directories to delete before push

Fresh LLM project — no read-only platform agents exist. Skip the `rm -rf TD-Managed:*` step that audience-agent has.

### Pre-push integration check

The shipped template doesn't include a chat integration config (`integrations/`). If the customer wants one (e.g., a Slack integration or a custom chat widget), that's an add-on outside the standard flow — discuss with the FDE lead before pushing.

## Phase 4 — Re-push After Customer Reviews

Follow `../../shared/push_pattern_fresh_project.md` "Re-Pushing Later" section end-to-end (no `rm -rf` step needed — fresh project). Analytics-specific updates between sessions:

1. Read the customer's reviewed requirements doc via `getConfluencePage` (URL in Current Project State).
2. **Re-distill the 3 customer-specific KBs from the customer's reviewed doc** — same distillation rules as Phase 3 (strip markers, hedge phrases, customer-collaborative voice; convert to terse direct agent instructions). The customer's edits supersede the Phase 1 inferences for any section they touched. KB destinations:
   - `knowledge_bases/data_dictionary.md` — schema details from the customer's edited Data Sources / Priority Tables section
   - `knowledge_bases/business_context.md` — business model, KPIs, key terms, segment naming, exclusions from the customer's edited Business Context sections
   - `knowledge_bases/sql_templates.md` — customer-validated SQL patterns from §9 (if the customer kept or edited the inference; drop unfilled `[NEEDS YOUR INPUT]` items)
3. `tdx agent push -y`.
4. Re-run test cases (Round 2) per `references/eval.md`.
