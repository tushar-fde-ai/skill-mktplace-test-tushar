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
https://github.com/treasure-data-ps/custom-audience-agent
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
| "the customer reviewed their requirements" / "their answers came back" | Phase 4 |
| "let's run tests on the agent" | Phase 3 (Round 1) if `business_context.md` hasn't been customer-validated yet, Phase 4 (Round 2) if it has — check Current Project State |
| "the agent's responses need tweaking" / "fix this failing test case" | Phase 4 iteration loop |
| "write the docs / Confluence pages for the agent" | Phase 5 |
| "push a fresh audience agent for [parent segment]" | Phase 1 → Phase 2 |
| "create the requirements doc for the customer" | Phase 1 (it's part of Phase 1d) |

When in doubt, **read Current Project State first** — it's authoritative.

## Workflow Sequence

The flow is **explore → propose draft → push → test → merge → document.** Phase 1 does deep schema exploration and **publishes a first-draft Confluence requirements doc** (pre-filled from inferences). The customer reviews/edits the draft on Confluence in parallel while Phases 2 and 3 run. The customer's edits flow back into the agent in Phase 4. **Test runs are gated** — the LLM never auto-runs `tdx agent test`; it presents the cases and waits for the FDE engineer to say go.

This flow spans multiple sessions. Phase 1 ends with the customer being asked to review/edit the Confluence draft; the FDE engineer typically resumes in a later session at Phase 4 (after the customer has finished reviewing).

| Phase | What happens | Customer-visible? |
|---|---|---|
| 1 | Project discovery + schema exploration → engineer-confirmed inference bundle → publish first-draft Confluence requirements doc | **Yes** at Phase 1d |
| 2 | Push minimal agent (placeholder business_context). Pipeline sanity check. | No |
| 3 | Distill business_context.md from the inference bundle, push KB, generate test cases, gated Round 1 run | No |
| 4 | Customer returns reviewed Confluence doc → re-distill business_context.md → push → gated Round 2 run | Customer's edits flow back |
| 5 | Customer-specific Confluence documentation set | Yes |

### Phase 1: Project Discovery + Schema Exploration + Publish Draft Requirements Doc

Phase 1 has four sub-steps: project lookup, schema exploration, engineer confirmation gate, and Confluence publication.

#### 1a. Project discovery

Read `../shared/push_pattern_td_managed.md` Step 1 for the project discovery flow (`tdx llm project list ... | grep "TD-Managed:"` with fallback).

1. Ask for the **parent segment name** and the **customer name** (used for Confluence page naming).
2. Project discovery as per shared push pattern → record the exact `TD-Managed: <Parent Segment>` project name.

#### 1b. Deep schema exploration

Skills to load before this step:
- `tdx-skills:parent-segment-analysis` — for `tdx ps desc` and query patterns against the parent segment
- `sql-skills:trino` — for TD-specific SQL functions (`TD_TIME_RANGE`, `TD_TIME_ADD`, `TD_SCHEDULED_TIME`, `TD_INTERVAL`) used in the §9 SQL templates
- `tdx-skills:tdx-basic` — if not already loaded, for `tdx sg list` syntax (covered below)

Run a thorough scan against the parent segment to produce material for Phase 1d (Confluence page) and Phase 3 (`business_context.md`):

1. **Full schema** — `tdx ps desc <parent-segment-name> -o` to enumerate the `customers` table + every `behavior_*` table.
2. **Sample rows** from each table (5-10 per table) to understand value formats and content. If there are many tables (>10), prioritize the `customers` table + the highest-row-count `behavior_*` tables; sample only 3-5 rows from the rest.
3. **Distribution queries** on the `customers` table for likely-relevant columns:
   - Categorical enums (e.g., `customer_segment`, `preferred_channel`, `churn_risk`) — `SELECT col, COUNT(*) GROUP BY 1 ORDER BY 2 DESC`
   - Numeric ranges — `SELECT MIN(col), AVG(col), MAX(col), APPROX_PERCENTILE(col, 0.9) FROM customers`
   - Date columns — `SELECT MIN(col), MAX(col)` for tenure
4. **Behavior table summaries** — for each `behavior_*` table: row count, distinct customer count, distinct values of any `event_type` / `outcome` / `status` columns, distinct values of `make` / `category` / similar dimensional fields.
5. **Existing segments + folder structure** — run `tdx sg list "<Parent Segment Name>"` to enumerate existing segments and their folder names. The folder names ARE the customer's existing segment-naming convention (e.g., `lifecycle/`, `service/`, `ev/`). If it errors with "no context", set context first with `tdx use "<Parent Segment Name>"` and retry.

Use the queried results to **draft an inference bundle** with one entry per requirements-doc section (§1–§9):

| Section | Inferred from |
|---|---|
| §1 Business Model | preferred_make values + behavior tables (e.g., test drives + recall notices + 10 auto makes → "automotive retailer") |
| §2 Business KPIs | numeric columns (`lifetime_value`), enum columns (`churn_risk`), behavior tables suggesting common metrics (test-drive → purchase, email opens, service retention, recall closure) |
| §3 Key Terms | enum values worth defining (`customer_segment` values: VIP / At-Risk / Regular / New, with their counts) + behavior-derived terms (e.g., "Lapsed Service" if a service behavior table exists) |
| §4 Priority Attributes | top ~10 columns from `customers` by usefulness, with concrete distribution stats (range, mean, median, top values) |
| §5 Segment Naming Conventions | existing folder names + segment titles |
| §6 Exclusions | PII columns auto-flagged by name pattern; behavior-table-implied exclusions (e.g., `event_type = 'Unsubscribed'` for email-activation segments); marketability rules from columns like `preferred_channel`, `marketing_opt_in` |
| §7 Target Users & Tone | conservative defaults — primary users: marketing managers, service ops, customer insights; tone: neutral professional marketing-analyst voice |
| §8 Output Preferences | conservative defaults — mix of summary text + Plotly chart + small results table; bar/line/pie chart-type guidance |
| §9 SQL Templates | up to 3 templates derived from schema patterns. Include only templates the schema actually supports (e.g., a "lapsed N days" template if there's a service/transaction behavior table; an engagement-rate template if there's an `event_type` column with multiple states; an "intent conquest" template if there's both an inventory-state column and a behavior table indicating intent) |

Mark fields you can't confidently infer with the canonical customer-facing markers — `[INFERRED — please confirm]` for confident guesses you want the customer to verify, `[NEEDS YOUR INPUT]` for fields the schema didn't reveal at all. Do NOT use `<TODO>` — the canonical markers are what the body template uses end-to-end.

#### 1c. Engineer confirmation gate

Present the inference bundle to the FDE engineer **before** publishing to Confluence. **Render the full Confluence body in chat as if previewing the page** — show the engineer exactly what the customer will receive (with all `[INFERRED — please confirm]` and `[NEEDS YOUR INPUT]` markers in place, all `[Inference from Phase 1 §X: ...]` authoring instructions resolved). The engineer scrolls through the rendered preview, calls out any inference that's wrong, and confirms once the bundle is ready.

> Here's the inference bundle I've drafted from the parent segment schema, rendered as the Confluence requirements doc body that will be published. The customer will review/edit/extend on Confluence — their edits flow back into `business_context.md` in Phase 4. Format follows the canonical example: https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/4981719206/
>
> [...full rendered body of the requirements doc, all 9 sections...]
>
> Review section-by-section. Reply 'publish' to send to Confluence as the customer-shareable first draft, or specify edits.

Apply any edits the engineer requests, then proceed to 1d on the next user turn (with explicit "publish" approval).

#### 1d. Publish first-draft Confluence requirements doc

Single-step gate: when the engineer approves in 1c, publish immediately. No re-show.

1. **Locate or create the Confluence folder hierarchy** per `../shared/confluence_folder_setup.md`:
   - Customer folder under CUST → Region
   - `FDE Solutions - <Customer>` sub-folder
   - `Custom Audience Agent - <Customer>` sub-folder ← `parentId` for all subsequent FDE pages
2. **Create the Current Project State page** per `../shared/current_project_state.md` (`Current Project State - <Customer>`).
3. **Publish the first-draft requirements doc** per `../shared/requirements_doc_pattern.md`:
   - Page title: `Audience Agent Requirements - <Customer>` (Confluence enforces unique titles per space — suffixing is mandatory)
   - Body: the engineer-confirmed Phase 1c rendered body (all 9 sections, customer-facing markers intact). See `agent-setup/references/requirements_doc.md` for the body template + canonical example: https://treasure-data.atlassian.net/wiki/spaces/CUST/pages/4981719206/
4. **Update Current Project State**: Phase 1 complete, requirements doc URL, customer notification date.
5. **Hand the URL to the FDE engineer** to share with the customer.
6. **End the session.** The customer reviews / edits asynchronously (typically days to weeks). Phases 2 + 3 can run in parallel during the wait window. Phase 4 resumes when the customer signals they're done.

**Persistence:** the inference bundle is **not** stored as a separate artifact. It IS the Confluence requirements doc — the published body equals the bundle. To recover in a later session (Phase 3 or 4), `getConfluencePage` on the requirements doc URL stored in **Current Project State**.

### Phase 2: Push Minimal Custom Audience Agent

Phase 2 establishes the deployment pipeline (auth, project binding, push mechanics) using the shipped placeholder. The deployed agent is a no-op until Phase 3 distills + pushes `business_context.md`. Customer-facing chat URL must NOT be shared yet.

Read `../shared/push_pattern_td_managed.md` for the full push flow. Read `agent-setup/SKILL.md` for the audience-specific files-to-edit table.

**Important — file edit ordering:** the audience-specific file edits in `agent-setup/SKILL.md` must be applied **between Step 3 (set tdx.json) and Step 5 (delete TD-Managed dirs) of the shared push pattern**. Specifically: flip `model: gpt-4.1` → `model: claude-4.5-sonnet` in all three Custom + Clone `agent.yml` files; remove the `get_segment_draft_rules` tool entry from `Custom Audience Agent/agent.yml`; edit `Custom Audience Agent/prompt.md` per the (a)/(b)/(c) instructions; delete `knowledge_bases/get_segment_draft_rules.md`. **Without this ordering, the FDE engineer risks pushing gpt-4.1 agents to the customer.**

Audience-specific:
- Template repo: `https://github.com/treasure-data-ps/custom-audience-agent`
- Don't write the inference bundle to `business_context.md` yet — that happens in Phase 3 after Phase 1d has shared the doc with the customer. Phase 2 just pushes the template with the shipped placeholder.
- Integration check substring: `name: "Custom Audience Agent"`
- Read-only dirs to delete: `TD-Managed: Marketing Copilot`, `TD-Managed: Data Source Finder`, `TD-Managed: Questions Suggester`

After push, update **Current Project State**: Phase 2 complete, project name, push date.

> **Note:** the deployed agent runs on the shipped placeholder (`You are an expert analyst.`) until Phase 3 distills + pushes `business_context.md`. **Don't share the chat widget URL with the customer until Phase 3 push completes** — the agent has no customer context yet and would appear broken.

### Phase 3: Generate Test Cases (Round 1 — Pre-Filled Draft Context)

Read `../shared/test_cases_pattern.md` for the full test-case lifecycle (TC-IDs, Confluence page format, `tdx agent test`, `updateConfluencePage` mechanics).

If resuming in a new session, **first read Current Project State** to recover the project name and the requirements doc URL. Then **`getConfluencePage` on the requirements doc URL** — the 9 pre-filled sections are the inference bundle.

**Round 1 runs against `business_context.md` populated from the Phase 1 inference bundle**, not an empty placeholder. Failures should be narrow — true business-context gaps the schema couldn't infer (specific business rules, jargon the schema doesn't reveal) or edge cases.

Audience-specific Phase 3 sequence (overrides the generic shared flow):

1. **Distill `business_context.md` from the Phase 1 inference bundle** — the bundle is recoverable by re-reading the published Confluence requirements doc. `business_context.md` is **agent-facing**, not a copy of the Confluence content. Strip the customer-facing markers (`[INFERRED — please confirm]`, `[NEEDS YOUR INPUT]`), strip hedge phrases ("please confirm", "currently"), and convert into terse direct instructions. See `agent-setup/references/business_context_template.md` for the distillation rules and Confluence-to-KB examples.

2. **Re-confirm pre-push state per `../shared/push_pattern_td_managed.md` "Re-Pushing Later"** — `TD-Managed: *` directories absent locally (a fresh `git clone` or `git pull` may have reintroduced them; `rm -rf` again if so), integration reference still intact. Then push:
   ```bash
   tdx agent push -y
   ```
   The deployed agent now reads the inference-bundle context for Round 1.

3. **Generate 10-15 test cases** per `../shared/test_cases_pattern.md`:
   - 5 test categories: schema discovery, attribute queries, behavior aggregations, segment draft creation, ambiguous/guardrail
   - Optional 6th category if SQL templates were in the inference bundle
   - Full category details + example prompts: `agent-setup/references/eval.md`
   - Use the inference bundle's real column names + sampled values to write concrete test prompts.

4. **Create the Confluence test cases page** + mirror cases into local `test.yml` per `../shared/test_cases_pattern.md`.

5. **Test-run confirmation gate.** Do NOT auto-run `tdx agent test`. Present the test cases to the FDE engineer:
   > Here are the 10-15 test cases I've generated. Review them at the Confluence page: <URL>. Reply 'run' when you're ready for me to execute `tdx agent test`.
   
   Wait for explicit approval. The FDE engineer may edit cases before approval — re-read `test.yml` if so.

6. **Run `tdx agent test`** only after explicit approval. Update Round 1 Result column on the Confluence page via `updateConfluencePage`.

7. **Update Current Project State** at end of Phase 3: test cases page URL, Round 1 pass rate, failing TC-IDs.

### Phase 4: Customer Edits Returned + Round 2

Almost always a new session, after the customer signals they've reviewed the Confluence requirements doc. **First action: read Current Project State.**

Read `../shared/test_cases_pattern.md` for Round 2 + iteration loop. Read `agent-setup/SKILL.md` Phase 4 for the audience-specific re-push checklist.

Audience-specific updates:

1. Read the customer's reviewed requirements doc via `getConfluencePage` (URL in Current Project State). The current state of the doc is the customer-validated source of truth — sections the customer didn't edit still contain the Phase 1 inference (with the original `[INFERRED — please confirm]` marker still in place), sections they edited contain their version.

2. **Re-distill `business_context.md` from the customer's reviewed doc.** Same distillation rules as Phase 3 (strip markers, strip hedge phrases, convert to direct agent instructions — see `agent-setup/references/business_context_template.md`). The customer's edits supersede the Phase 1 inferences for any section they touched. Sections the customer left untouched continue to be the Phase 1 inference (still useful — distill it too).

3. If §9 of the doc has SQL templates (whether the customer kept the inference, edited, or replaced), distill into `knowledge_bases/sql_templates.md` (see `agent-setup/references/sql_templates_template.md`). Drop SQL templates the customer marked as `[NEEDS YOUR INPUT]` and didn't fill in.

4. Re-confirm pre-push state: `TD-Managed: *` directories absent locally, integration reference intact.

5. `tdx agent push -y`.

6. **Test-run confirmation gate (Round 2).** Do NOT auto-run. Present:
   > `business_context.md` re-distilled from the customer's reviewed requirements and pushed. Reply 'run' when you're ready for me to execute `tdx agent test` for Round 2.
   
   Wait for explicit approval.

7. Run `tdx agent test` only after approval. Update Round 2 Result column on the Confluence page.

Failure-to-fix mapping: see `agent-setup/references/eval.md` Round 2 section.

### Phase 5: Customer-Specific Documentation

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
