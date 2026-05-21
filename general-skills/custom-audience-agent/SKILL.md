---
name: custom-audience-agent
description: |
  Customize and deploy a custom AI Foundry audience agent on top of an existing TD-Managed parent segment project. Trigger on: audience agent, custom audience agent, CDP audience agent, segment agent, parent segment agent, audience copilot, marketing copilot customization. Push-first workflow → customer requirements gathering on Confluence → two-round test cases → customer-specific documentation.
---

# Custom Audience Agent

Customize and deploy a custom AI Foundry agent that analyzes CDP parent segment data — queries customer attributes, behaviors, and existing segments to answer business questions via natural language and draft new segment rules.

## Mental Model

When a parent segment is set up in TD, the platform auto-provisions an AI project named `TD-Managed: <Parent Segment Name>` containing 3 **read-only** TD-Managed agents. Those cannot be edited.

To customize behavior, we **push a parallel set of agents into that same project**:
- `Custom Audience Agent` (main, claude-4.5-sonnet)
- `Clone Data Source Finder` (claude-4.5-sonnet mirror)
- `Clone Questions Suggester` (claude-4.5-sonnet mirror)
- Custom knowledge bases (`business_context.md`, optionally `sql_templates.md`)
- Chat integration that points at the Custom Audience Agent prompt

The `TD-Managed: *` agent directories in the template repo are reference copies only — they must be deleted locally before push.

**Note on the model default.** The template repo's `agent.yml` files currently ship with `model: gpt-4.1`. As of this skill, all customer engagements use `model: claude-4.5-sonnet` (matches the platform default and the TD-Managed agents). The Phase 2 file-edit checklist includes flipping the model field in the three Custom + Clone agent.yml files before push.

The legacy `get_segment_draft_rules` knowledge base + tool was a gpt-only workaround. With Claude this is unnecessary — the schema description in the prompt is followed reliably. **Drop the KB and its tool reference from the template before push.** The Phase 2 instructions cover this.

## Template Repo

```
https://github.com/tushar-fde-ai/custom-audience-agent
```

## Solution-specific configuration

These values are passed into the shared patterns:

| Field | Value |
|------|-------|
| Solution folder name | `Custom Audience Agent` |
| Solution folder title variants | `Audience Agent`, `Custom Agent`, `Custom Audience Agent` |
| Push pattern | `../shared/push_pattern_td_managed.md` |
| Read-only platform agents to delete locally | `TD-Managed: Marketing Copilot`, `TD-Managed: Data Source Finder`, `TD-Managed: Questions Suggester` |
| Integration check substring | `name: "Custom Audience Agent"` (in `integrations/chat_parent_segment.yml` `actions:` block) |

## Entry Point — Where to Start

Before starting any phase, ask the user: **"Is this a new audience agent engagement, or are you resuming an existing one?"**

- **New engagement:** start at Phase 1.
- **Resume:** ask for the customer name, search Confluence for `Current Project State - <Customer>` (see `../shared/current_project_state.md`). Read it — its "Current phase" + "Next Action" fields say where to pick up. If no State page exists, treat as new but skip already-done steps.

Common phrases mapped to phases (when a customer name is given):

| User says | Likely phase |
|---|---|
| "the customer filled out their requirements" / "their answers came back" | Phase 5 |
| "let's run tests on the agent" | Phase 4 (Round 1) if `business_context.md` is empty, Phase 5 (Round 2) if it has customer content — check Current Project State |
| "the agent's responses need tweaking" / "fix this failing test case" | Phase 5 iteration loop |
| "write the docs / Confluence pages for the agent" | Phase 6 |
| "push a fresh audience agent for [parent segment]" | Phase 1 → Phase 2 |
| "create the requirements doc for the customer" | Phase 3 (assumes Phase 2 push already done) |

When in doubt, **read Current Project State first** — it's authoritative.

## Workflow Sequence

The flow is **push first, gather requirements second, test third, document last.** Phase 2 pushes with the shipped placeholder. Phase 4 generates a *schema-derived draft* of `business_context.md`, runs Round 1 tests against that, and surfaces business-specific gaps the customer needs to close. Phase 5 merges the customer's answers into the draft (customer wins on conflict; schema defaults fill gaps).

This flow spans multiple sessions. Phase 3 ends with the customer being asked to fill out a Confluence page; the FDE engineer typically resumes in a later session at Phase 4 or 5.

### Phase 1: Lightweight Parent Segment Discovery

Just enough to know which project to push into. Read `../shared/push_pattern_td_managed.md` Step 1 for the project discovery flow (`tdx llm project list ... | grep "TD-Managed:"` with fallback).

Audience-specific:
1. Ask for the **parent segment name**.
2. Project discovery as per shared push pattern.
3. Quick schema sanity check using `tdx-skills:parent-segment-analysis`: `tdx ps desc <parent-segment-name> -o`.

### Phase 2: Push Minimal Custom Audience Agent

Read `../shared/push_pattern_td_managed.md` for the full push flow (clone, set tdx.json, integration check, `rm -rf TD-Managed:*`, `tdx agent push -y`). Read `agent-setup/SKILL.md` for the audience-specific files-to-edit table.

Audience-specific:
- Template repo: `https://github.com/tushar-fde-ai/custom-audience-agent`
- Don't edit `business_context.md` yet — the shipped placeholder is fine for first push. Phase 5 fills it in.
- Integration check substring: `name: "Custom Audience Agent"`
- Read-only dirs to delete: `TD-Managed: Marketing Copilot`, `TD-Managed: Data Source Finder`, `TD-Managed: Questions Suggester`

After push, update **Current Project State**: Phase 2 complete, project name, push date.

### Phase 3: Create Customer Requirements Doc on Confluence

Read `../shared/confluence_folder_setup.md` for folder discovery + creation (the `Custom Audience Agent` folder under `<Customer>/FDE Solutions/`).

Read `../shared/current_project_state.md` for the State page setup — create it now, before the requirements doc.

Read `../shared/requirements_doc_pattern.md` for the customer-shareable page workflow.

Audience-specific:
- Page title: `Audience Agent Requirements - <Customer>` (every Confluence page must be suffixed with the customer name — Confluence enforces unique titles per space)
- Body template (the 9-section customer-fillable form): see `agent-setup/references/requirements_doc.md`

### Phase 4: Generate Test Cases (Round 1 — Schema-Derived Draft Context)

Read `../shared/test_cases_pattern.md` for the full test-case lifecycle (TC-IDs, Confluence page format, `tdx agent test`, `updateConfluencePage` mechanics).

If resuming in a new session, **first read Current Project State**.

**Round 1 runs against a schema-derived draft of `business_context.md`, not the empty placeholder.** This makes Round 1 a meaningful baseline — failures map to *business-specific* gaps (terms like "VIP", segment naming patterns) rather than "agent doesn't know any columns."

Audience-specific Phase 4 sequence (overrides the generic shared flow):

1. **Schema exploration** using `tdx-skills:parent-segment-analysis` — `tdx ps desc <parent_segment> -o`, sample data, list distinct values for low-cardinality columns.

2. **Draft `business_context.md` from schema discovery.** Write a working draft covering only the schema-derivable fields:
   - **Priority Attributes** — top ~10 columns from the `customers` table by likely usefulness for marketing/segmentation (monetary fields, dates, channel/source, status enums, demographic non-PII). Each gets an inferred 1-line description from the column name. If inference is uncertain, write `<TODO: confirm with customer>`.
   - **Exclusion Rules → PII columns** — auto-flag columns matching common PII patterns (`email`, `phone`, `mobile`, `ssn`, `dob`, `birth`, `name`, `address`, `zip`, `postal`).
   - All other sections (Business Model, Key Terms, Segment Naming Conventions, Customer filters, hidden behavior tables) — leave as empty placeholders. Phase 5 fills these from the customer's answers.
   See `agent-setup/references/business_context_template.md` for the schema-derivable vs customer-required field split.

3. **Confirmation gate.** Present the draft to the FDE engineer:
   > Here's the schema-derived draft of `business_context.md`. Review the Priority Attributes (any wrong descriptions?) and Exclusion Rules → PII columns (any false positives or missed PII?). Confirm or edit before I push and run Round 1 tests.
   
   Wait for confirmation. Apply edits if any.

4. **Push the draft KB:**
   ```bash
   tdx agent push -y
   ```
   The deployed agent now reads the schema-derived context for Round 1.

5. **Generate, store, and run test cases** per `../shared/test_cases_pattern.md`:
   - 5 test categories: schema discovery, attribute queries, behavior aggregations, segment draft creation, ambiguous/guardrail
   - Optional 6th category if customer provides SQL templates in §9 of the requirements doc (Phase 5 only)
   - Full category details + example prompts: `agent-setup/references/eval.md`
   - Use the schema-derived attributes from Step 1-2 to write concrete test prompts (real column names, not placeholders)

6. **Update Current Project State** at end of Phase 4: test cases page URL, Round 1 pass rate, failing TC-IDs, and a note that `business_context.md` was schema-derived and will merge with customer answers in Phase 5.

### Phase 5: Update Agent with Customer Requirements (Round 2)

Almost always a new session. **First action: read Current Project State.**

Read `../shared/test_cases_pattern.md` for Round 2 + iteration loop. Read `agent-setup/SKILL.md` Phase 5 for the audience-specific re-push checklist.

Audience-specific updates:
1. Read filled requirements doc via `getConfluencePage`.
2. **Merge customer answers into the existing `business_context.md` draft** (see `agent-setup/references/business_context_template.md` "Phase 5 merge rule"). Section-by-section:
   - **Empty customer field** → keep the schema-derived draft as-is.
   - **Customer wrote something** → customer wins, replaces the draft section.
   - **Customer added items the draft didn't have** (e.g., custom PII column) → append.
3. If customer provided SQL templates in §9, create `knowledge_bases/sql_templates.md` (see `agent-setup/references/sql_templates_template.md`).
4. Re-confirm pre-push state: `TD-Managed: *` directories absent locally, integration reference intact.
5. `tdx agent push -y`.
6. Re-run test cases (Round 2).

Failure-to-fix mapping: see `agent-setup/references/eval.md` Round 2 section.

### Phase 6: Customer-Specific Documentation

Read `../shared/customer_docs_pattern.md` for the 5-page set + create order + keep-current rules.

If resuming in a new session, **first read Current Project State**.

Before authoring the Behavior page, `Read` the local `knowledge_bases/business_context.md` and (if present) `knowledge_bases/sql_templates.md`.

Audience-specific page content: see `prod-docs/SKILL.md`.

## Sub-Folder Reference

| Folder / File | Contents |
|---------------|----------|
| `agent-setup/SKILL.md` | Audience-specific Phase 2 + Phase 5 file edits (which files to touch, which to leave alone) |
| `agent-setup/references/requirements_doc.md` | The 9-section customer-fillable body template + Phase 5 mapping reference tables |
| `agent-setup/references/business_context_template.md` | Minimal 5-section template for `business_context.md` |
| `agent-setup/references/sql_templates_template.md` | Format for the optional `sql_templates.md` KB |
| `agent-setup/references/eval.md` | Audience-specific test categories + example prompts + failure-to-fix mapping |
| `prod-docs/SKILL.md` | Audience-specific content for the Phase 6 documentation page set |
| `../shared/*.md` | Generic patterns reused by every solution under `general-skills/` |
