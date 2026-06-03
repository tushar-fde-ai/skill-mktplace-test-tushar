---
name: custom-analytics-agent
description: |
  Customize and deploy a custom AI Foundry analytics agent in a fresh LLM project. Trigger on: analytics agent, custom analytics agent, data analytics agent, reporting agent, dashboard agent, BI agent. Explore → propose draft → push → test → merge → document.
---

# Custom Analytics Agent

Customize and deploy a custom AI Foundry analytics agent — queries TD databases, writes Trino SQL, and renders results as Plotly charts or React dashboards. The agent's invariant logic (workflow, tone, guardrails, performance rules) ships in the template repo. Customer-specific knowledge — schema, business rules, SQL patterns — lives in 3 KBs that get populated via Phase 3/4 distillation.

## Mental Model

Unlike the audience agent (which customizes an existing `TD-Managed: <Parent Segment>` project), the analytics agent **runs in a fresh LLM project** created via `tdx llm project create`. There are no read-only platform agents to delete locally — the full template gets pushed as-is.

**Default model:** `claude-4.5-sonnet` (matches the platform default). All `agent.yml` files in the template ship with this.

## Template Repo

```
https://github.com/treasure-data/fde-custom-analytics-agent
```

The template ships with:
- `Analytics Agent/` — main agent (vertical-agnostic SQL analyst, 7 tools)
- `data_source_finder/` — sub-agent for ad-hoc schema discovery
- `knowledge_bases/` — 5 KBs:
  - `master_database.yml` — points at the customer's TD database (Phase 2 edit)
  - `data_dictionary.md` — Phase 3 distillation populates from schema discovery
  - `business_context.md` — Phase 3 distillation populates from customer requirements doc
  - `sql_templates.md` — Phase 3 distillation populates if customer provides templates (optional)
  - `plotly_instructions.md` — generic chart rules (no edits)
  - `react_dashboard_instructions.md` — generic dashboard rules (no edits)

## Solution-specific configuration

These values are passed into the shared patterns:

| Field | Value |
|------|-------|
| Solution folder name | `Custom Analytics Agent` |
| Solution folder title variants | `Analytics Agent`, `Custom Analytics Agent`, `Reporting Agent`, `Dashboard Agent` |
| Push pattern | `../shared/push_pattern_fresh_project.md` |
| Read-only platform agents to delete locally | None (fresh project) |
| Integration check substring | n/a (the template doesn't ship with a chat integration; if the customer wants one, that's a Phase 2 add-on outside the standard flow) |

## Entry Point — Where to Start

Before starting any phase, ask the user: **"Is this a new analytics agent engagement, or are you resuming an existing one?"**

- **New engagement:** start at Phase 1.
- **Resume:** ask for the customer name, search Confluence for `Current Project State - <Customer>` (see `../shared/current_project_state.md`). Read it — its "Current phase" + "Next Action" fields say where to pick up. If no State page exists, treat as new but skip already-done steps.

Common phrases mapped to phases (when a customer name is given):

| User says | Likely phase |
|---|---|
| "the customer reviewed their requirements" / "their answers came back" | Phase 4 |
| "let's run tests on the analytics agent" | Phase 3 (Round 1) if the KBs haven't been customer-validated yet, Phase 4 (Round 2) if they have — check Current Project State |
| "the agent's responses need tweaking" / "fix this failing test case" | Phase 4 iteration loop |
| "write the docs / Confluence pages for the agent" | Phase 5 |
| "push a fresh analytics agent for [customer]" | Phase 1 → Phase 2 |
| "create the requirements doc for the customer" | Phase 1 (it's part of Phase 1d) |

When in doubt, **read Current Project State first** — it's authoritative.

## Workflow Sequence

The flow is **explore → propose draft → push → test → merge → document.** Phase 1 does deep schema exploration and **publishes a first-draft Confluence requirements doc** (pre-filled from inferences). The customer reviews/edits the draft on Confluence in parallel while Phases 2 and 3 run. The customer's edits flow back into the agent in Phase 4. **Test runs are gated** — the LLM never auto-runs `tdx agent test`; it presents the cases and waits for the FDE engineer to say go.

This flow spans multiple sessions. Phase 1 ends with the customer being asked to review/edit the Confluence draft; the FDE engineer typically resumes in a later session at Phase 4.

| Phase | What happens | Customer-visible? |
|---|---|---|
| 1 | Project setup + schema exploration → engineer-confirmed inference bundle → publish first-draft Confluence requirements doc | **Yes** at Phase 1d |
| 2 | Push minimal agent (placeholder KBs). Pipeline sanity check. | No |
| 3 | Distill the 3 customer-specific KBs from the inference bundle, push, generate test cases, gated Round 1 run | No |
| 4 | Customer returns reviewed Confluence doc → re-distill the 3 KBs → push → gated Round 2 run | Customer's edits flow back |
| 5 | Customer-specific Confluence documentation set | Yes |

### Phase 1: Project Setup + Schema Exploration + Publish Draft Requirements Doc

Phase 1 has four sub-steps: customer + project setup, schema exploration, engineer confirmation gate, and Confluence publication.

#### 1a. Project setup

1. Ask for the **customer name** and the **TD database** the analytics agent should query (single database — the analytics agent's `master_database.yml` points at one database; cross-database queries are out of scope for the standard template).
2. Choose a project name following `<Customer> Analytics Agent` convention.
3. Record the project name + the database name.

#### 1b. Deep schema exploration

Skills to load before this step:
- `tdx-skills:tdx-basic` — for `tdx databases`, `tdx tables`, `tdx describe`, and general `tdx` syntax
- `sql-skills:trino` — for TD-specific SQL functions used in analytical queries and SQL templates
- `semantic-layer:data-dictionary` *(optional)* — if the customer has a documented data dictionary, prefer it over schema-only inference

Run a thorough scan against the customer's database to produce material for Phase 1d (Confluence page) and Phase 3 (the 3 customer-specific KBs):

1. **Table enumeration** — `tdx tables <database>` to list every table. Identify likely fact tables (transactions, events, orders), dimension tables (customers, products, dates), and pre-aggregated marts.
2. **Schema** — `tdx describe <database>.<table>` for the most-likely-relevant tables. Skip system / staging / temp tables.
3. **Sample rows** from each table (5-10 per table) to understand value formats, granularity, and join keys.
4. **Distribution queries** on key fact-table columns (date ranges, top categorical values, numeric ranges, null ratios on key columns) to ground inferences with real numbers.
5. **Type gotchas** — flag varchar columns that should be timestamps (need `TD_TIME_PARSE`), columns where empty string is meaningful (vs NULL), columns with high null ratios that should be excluded by default.

Use the queried results to **draft an inference bundle** with one entry per requirements-doc section. The section list is in `agent-setup/references/requirements_doc.md`.

Mark fields you can't confidently infer with the canonical customer-facing markers — `[INFERRED — please confirm]` for confident guesses you want the customer to verify, `[NEEDS YOUR INPUT]` for fields the schema didn't reveal at all. Same convention as the audience agent.

#### 1c. Engineer confirmation gate

Present the inference bundle to the FDE engineer **before** publishing to Confluence. **Render the full Confluence body in chat as if previewing the page** — show the engineer exactly what the customer will receive (with all `[INFERRED — please confirm]` and `[NEEDS YOUR INPUT]` markers in place, all authoring instructions resolved). The engineer scrolls through the rendered preview, calls out any inference that's wrong, and confirms once the bundle is ready.

> Here's the inference bundle I've drafted from the customer's database, rendered as the Confluence requirements doc body that will be published. The customer will review/edit/extend on Confluence — their edits flow back into the agent's KBs in Phase 4.
>
> [...full rendered body of the requirements doc, all sections...]
>
> Review section-by-section. Reply 'publish' to send to Confluence as the customer-shareable first draft, or specify edits.

Apply any edits the engineer requests, then proceed to 1d on the next user turn (with explicit "publish" approval).

#### 1d. Publish first-draft Confluence requirements doc

Single-step gate: when the engineer approves in 1c, publish immediately. No re-show.

1. **Locate or create the Confluence folder hierarchy** per `../shared/confluence_folder_setup.md`:
   - Customer folder under CUST → Region
   - `FDE Solutions - <Customer>` sub-folder
   - `Custom Analytics Agent - <Customer>` sub-folder ← `parentId` for all subsequent FDE pages
2. **Create the Current Project State page** per `../shared/current_project_state.md` (`Current Project State - <Customer>`).
3. **Publish the first-draft requirements doc** per `../shared/requirements_doc_pattern.md`:
   - Page title: `[CUST-FACING] Analytics Agent Requirements - <Customer>` (Confluence enforces unique titles per space — suffixing is mandatory; the `[CUST-FACING]` prefix marks pages the customer is meant to read — see `../shared/customer_docs_pattern.md`)
   - Body: the engineer-confirmed Phase 1c rendered body. See `agent-setup/references/requirements_doc.md` for the body template.
4. **Update Current Project State**: Phase 1 complete, requirements doc URL, customer notification date.
5. **Hand the URL to the FDE engineer** to share with the customer.
6. **End the session.** The customer reviews / edits asynchronously. Phases 2 + 3 can run in parallel during the wait window. Phase 4 resumes when the customer signals they're done.

**Persistence:** the inference bundle is **not** stored as a separate artifact. It IS the Confluence requirements doc. To recover in a later session (Phase 3 or 4), `getConfluencePage` on the requirements doc URL stored in **Current Project State**.

### Phase 2: Push Minimal Custom Analytics Agent

Phase 2 establishes the deployment pipeline (auth, project creation, push mechanics) using template-shipped placeholders. Customer-facing chat URL must NOT be shared yet.

Read `../shared/push_pattern_fresh_project.md` for the full push flow (clone, set tdx.json, `tdx llm project create`, `tdx agent push -y`). Read `agent-setup/SKILL.md` for the analytics-specific files-to-edit table.

**Important — file edit ordering:** apply the analytics-specific file edits in `agent-setup/SKILL.md` **between Step 3 (set tdx.json) and Step 6 (push) of the shared push pattern.**

Audience-specific:
- Template repo: `https://github.com/treasure-data/fde-custom-analytics-agent`
- Don't write the inference bundle to the 3 customer KBs yet — that happens in Phase 3 after Phase 1d has shared the doc with the customer. Phase 2 just pushes the template with shipped stub KBs.
- No `rm -rf` step (fresh project, no read-only platform agents)

After push, update **Current Project State**: Phase 2 complete, project name, push date.

> **Note:** the deployed agent runs on stub KBs (frontmatter-only, no customer content) until Phase 3 distills + pushes the populated KBs. The agent will be functional but generic — it'll fall back to `data_source_finder` on every schema question. **Don't share the chat widget URL with the customer until Phase 3 push completes** — the agent has no customer context yet and would appear underbaked.

### Phase 3: Distill KBs + Generate Test Cases (Round 1)

Read `../shared/test_cases_pattern.md` for the full test-case lifecycle (TC-IDs, Confluence page format, `tdx agent test`, `updateConfluencePage` mechanics).

If resuming in a new session, **first read Current Project State** to recover the project name and the requirements doc URL. Then **`getConfluencePage` on the requirements doc URL** — the pre-filled sections are the inference bundle.

**Round 1 runs against KBs populated from the Phase 1 inference bundle**, not stubs. Failures should be narrow — true business-context gaps the schema couldn't infer or edge cases.

Analytics-specific Phase 3 sequence:

1. **Distill the inference bundle into the 3 customer-specific KBs:**
   - `knowledge_bases/data_dictionary.md` — schema doc (table list + column list + types + gotchas) from the inference bundle's data-sources / schema-derivable sections.
   - `knowledge_bases/business_context.md` — business model, KPIs, key terms, segment naming conventions, exclusion rules from the inference bundle's business-context sections.
   - `knowledge_bases/sql_templates.md` — only if the inference bundle has §9 SQL templates. Otherwise leave the file as the shipped stub.
   - Strip customer-facing markers (`[INFERRED — please confirm]`, `[NEEDS YOUR INPUT]`), strip hedge phrases ("please confirm", "currently"), and convert into terse direct agent instructions. See pattern in audience-agent's `business_context_template.md` for distillation rules — same approach.
2. **Re-confirm pre-push state per `../shared/push_pattern_fresh_project.md` "Re-Pushing Later"** (no `rm -rf` step — fresh project), then push:
   ```bash
   tdx agent push -y
   ```
3. **Generate 10-15 test cases** per `../shared/test_cases_pattern.md`:
   - 5 test categories: schema discovery, SQL generation, multi-step analysis, dashboard generation, ambiguous/guardrail
   - Optional 6th category if SQL templates were in the inference bundle
   - Full category details + example prompts: `agent-setup/references/eval.md`
   - Use the inference bundle's real table + column names to write concrete test prompts.
4. **Create the Confluence test cases page (internal) AND the customer-facing Google Sheet** + mirror cases into local `test.yml`. Both surfaces live side-by-side throughout — Confluence is canonical, Sheet is a customer-friendly subset (no internal pass criteria). See `../shared/test_cases_pattern.md` Step 2 + Step 2b.
5. **Test-run confirmation gate.** Do NOT auto-run `tdx agent test`. Present the test cases:
   > Here are the 10-15 test cases I've generated. Review them at the Confluence page: <URL>. Reply 'run' when you're ready for me to execute `tdx agent test`.
   
   Wait for explicit approval.
6. **Run `tdx agent test`** only after explicit approval. Update **both** Round 1 Result columns: full-page replace on Confluence via `updateConfluencePage`, and write the same column into the customer-facing Sheet. Share the Sheet view-only with the customer's email(s) (first run only — subsequent rounds just refresh cells).
7. **Update Current Project State** at end of Phase 3: Confluence test cases page URL, customer-facing Google Sheet URL, Round 1 pass rate, failing TC-IDs.

### Phase 4: Customer Edits Returned + Round 2

Almost always a new session, after the customer signals they've reviewed the Confluence requirements doc. **First action: read Current Project State.**

Read `../shared/test_cases_pattern.md` for Round 2 + iteration loop. Read `agent-setup/SKILL.md` Phase 4 for the analytics-specific re-push checklist.

Analytics-specific updates:

1. Read the customer's reviewed requirements doc via `getConfluencePage` (URL in Current Project State).
2. **Re-distill the 3 customer-specific KBs from the customer's reviewed doc.** Same distillation rules as Phase 3. The customer's edits supersede the Phase 1 inferences for any section they touched.
3. If the customer edited or added SQL templates, update `knowledge_bases/sql_templates.md`. If the customer left §9 as `[NEEDS YOUR INPUT]` and didn't fill it in, leave the stub alone.
4. Re-confirm pre-push state per `../shared/push_pattern_fresh_project.md`.
5. `tdx agent push -y`.
6. **Test-run confirmation gate (Round 2).** Do NOT auto-run. Present:
   > KBs re-distilled from the customer's reviewed requirements and pushed. Reply 'run' when you're ready for me to execute `tdx agent test` for Round 2.
   
   Wait for explicit approval.
7. Run `tdx agent test` only after approval. Update Round 2 Result column on the Confluence page.

Failure-to-fix mapping: see `agent-setup/references/eval.md` Round 2 section.

### Phase 5: Customer-Specific Documentation

Read `../shared/customer_docs_pattern.md` for the 5-page set + create order + keep-current rules.

If resuming in a new session, **first read Current Project State**.

Before authoring the Behavior page, `Read` the local KBs (`knowledge_bases/business_context.md`, `knowledge_bases/data_dictionary.md`, and `knowledge_bases/sql_templates.md` if non-empty).

Analytics-specific page content: see `prod-docs/SKILL.md`.

## Sub-Folder Reference

| Folder / File | Contents |
|---------------|----------|
| `agent-setup/SKILL.md` | Analytics-specific Phase 2 + Phase 4 file edits (which files to touch, which to leave alone) |
| `agent-setup/references/requirements_doc.md` | The customer-facing requirements doc body template (sections + markers) |
| `agent-setup/references/eval.md` | Analytics-specific test categories + example prompts + failure-to-fix mapping |
| `prod-docs/SKILL.md` | Analytics-specific content for the Phase 5 documentation page set |
| `../shared/*.md` | Generic patterns reused by every solution under `general-skills/` |
