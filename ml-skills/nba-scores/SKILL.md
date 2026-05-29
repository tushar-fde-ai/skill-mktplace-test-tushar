---
name: fde-nba-scores
description: |
  NBA (Next Best Action) Engagement Scores for Treasure Data. Configures workflows that union customer activity, derive per-profile Next Best Channel / Next Best Time / Next Best Campaign affinity scores plus cart-abandon and new-visitor flags, and write the combined output into Audience Studio. Trigger on: NBA, Next Best Action, engagement scores, channel affinity, time of day affinity, cart abandon, new visitor, NBA dashboard, requirements gathering for NBA, NBA workflow setup, NBA configuration, NBA runbook.
---

# NBA Engagement Scores

Next Best Action (NBA) workflow that unions customer behavioral data from multiple sources, scores per-profile engagement across **Channel** (where), **Time of Day** (when), and **Campaign** (what), and combines the metrics into a single profile-level table that can be joined to a Parent Segment in Audience Studio.

## How to Start

Read `../shared/SKILL.md` — it defines the entry point (new vs. resume), the phase sequence, and when to read each shared pattern file. Follow that flow. This file provides the NBA-specific context you need at each phase.

## Phase Summary

| Phase | What happens |
|---|---|
| Phase 1 | Data exploration — discover tables, confirm schema |
| Phase 2 | Create Confluence folder + Current Project State + Requirements Doc |
| Phase 3 | Push minimal workflow, validate output tables |
| Phase 4 | Deploy Foundry agent, run integration check |
| Phase 5 | Re-configure workflow from filled requirements, re-validate |
| Phase 6 | Write customer-facing documentation set |

---

## NBA-Specific Context by Phase

### Phase 1: Data Exploration

Use `tdx-skills:tdx-basic` to explore the customer's TD databases. **Do not search by table name patterns.** Instead follow these steps:

**Step 1 — List every table in the customer database.**
Run `tdx tables "<database>.*"` to get the full table list.

**Step 2 — Classify each table.**
For every table, run `tdx describe` to inspect its schema. Classify it into one of three buckets:

| Bucket | Criteria |
|---|---|
| **Activity / behavior table** | Has a user ID column + a Unix timestamp column + rows represent individual customer events or interactions (one row = one event). Candidates for NBA source tables. |
| **Profile / dimension table** | Has a user ID column but rows represent one record per customer (master tables, attributes, membership tiers). Not a source table. |
| **System / output table** | Workflow output, audit logs, ML result tables. Not a source table. |

**Step 3 — Within activity tables, assess customer engagement relevance.**
For each activity table in bucket 1, ask: *does this event reflect a customer choosing to interact with the brand?*

| Include | Exclude |
|---|---|
| Pageviews, web clicks, searches, add-to-cart | Internal system events, bot traffic |
| Email opens, clicks, unsubscribes | Email sends / deliveries (not user-initiated) |
| Orders, purchases, reservations | Refunds, cancellations (negative outcome, not an engagement touch) |
| Support tickets, chat sessions, in-store visits | Background sync events, ETL audit rows |
| Loyalty activity, app sessions, survey responses | Derived / aggregated tables (already rolled up) |

**Step 4 — Present the recommended list to the user before proceeding.**
Show a table of ALL activity tables found with your recommendation (include / exclude) and a one-line reason for each. Ask the user to confirm or adjust before locking in the source table list. Do not skip this step.

**Step 5 — For each confirmed source table, note down:**
- Full table name
- User ID column name
- Timestamp column name
- Whether UTM or channel columns exist
- Whether any column can serve as a conversion signal

These become `input_params.yml` values.

**Confluence folder name for this solution:** `NBA Scores`
**Title variants for fuzzy matching:** `NBA`, `NBA Scores`, `NBA Engagement Scores`, `Next Best Action`

### Phase 2: Requirements Doc

Page title: `NBA Scores Requirements - <Customer>`

Body template: `workflow-setup/references/requirements_doc.md`

Key questions the requirements doc must answer: parent segment name, scoring strategy (`percentile` / `quartile` / `minmax`), cart-abandon window (days), new-visitor window (days), time-of-day granularity (4-bucket default), ESP / activation channel.

### Phase 3: Push Minimal Workflow

**GitHub repo:** `https://github.com/treasure-data-ps/nba_eng_scores`
**Workflow project name:** `nba_eng_prod`
**Workflow path:** `nba_eng_scores/td_wf/`
**Entry point:** `nba_eng_launch.dig`
**Config file:** `nba_eng_scores/td_wf/config/input_params.yml`

For the minimal push, populate `input_params.yml` with what you know from Phase 1 (tables, user ID column, timestamp column). Use Phase 2 answers if already available; otherwise use safe defaults.

Read `workflow-setup/references/workflow_setup_guide.md` for full parameter generation instructions.
Read `workflow-setup/references/yaml_structure.md` for the parameter reference.
Read `workflow-setup/references/table_configuration.md` for per-source-type config.
Use `workflow-setup/references/input_params_template.yml` as the starting point.

After the workflow runs, validate output tables per `workflow-setup/references/eval.md`.

**Key output table to check:** `nba_combined_metrics_final` — confirm it has rows and the expected score columns.
**Dashboard tables to check:** `nba_dash_stats_summary`, `nba_dash_model_metrics`, `nba_dash_source_tables`.

### Phase 4: Foundry Agent Setup

**Foundry agent project name:** `NBA Engagement Scores`
**Agent name:** `NBA Insights Agent`

Read `agent-skills/foundry-agent/SKILL.md` for the files-to-edit table and integration check.

Populate `business_context.md` with what you learned in Phases 1–3: customer industry, source tables used, scoring strategy chosen, output table name, and any notable data quirks.

### Phase 5: Update Workflow with Customer Requirements

After reading the filled requirements doc, present a summary of the parameter changes you plan to make before touching any files. Key areas that typically change between Phase 3 and Phase 5:

- Scoring strategy (customer may prefer `quartile` for simpler marketer communication)
- Cart-abandon and new-visitor window lengths
- Adding or removing source table types
- Adjusting channel regex rules if the customer has non-standard UTM naming

Read `workflow-setup/references/workflow_setup_guide.md` for update mechanics.
Re-run the workflow and re-validate output tables (same checks as Phase 3).

### Phase 6: Customer Documentation

Read `../shared/customer_docs_pattern.md` for the 5-page set.

NBA-specific content for each page is in `prod-docs/references/`:

| Page | Reference file |
|---|---|
| Architecture | `prod-docs/references/runbook.md` (architecture section) |
| Runbook | `prod-docs/references/runbook.md` |
| Customer-facing overview | `prod-docs/references/customer_docs.md` |
| Technical handoff | `prod-docs/references/technical_handoff.md` |

---

## Scoring Strategies Reference

Set via `scoring_logic` in `input_params.yml`. Surface this to the customer during Phase 2 requirements gathering.

| Strategy | When to use |
|---|---|
| `percentile` | Default. Score = percentile rank within audience. Best for even distributions. |
| `quartile` | Coarse 4-bucket banding. Easier to explain to marketers. |
| `minmax` | Hivemall min-max scaling. Best when activity is heavily skewed. |

## How It Works (for context)

1. **Union activity** — pageviews, email events, sales interactions, orders → `nba_combined_user_events` (sessionized by `session_length`).
2. **Next Best Channel** — UTM params + channel regex → per-profile affinity score per channel.
3. **Next Best Time** — activity bucketed into morning / afternoon / evening / overnight → per-profile daypart affinity.
4. **Next Best Campaign** — business-rule flags: cart-abandon (cart add without conversion in window) and new-visitor (recent first visit, low page count, no purchase).
5. **Combine** — all metric tables joined on `${unique_user_id}` → `nba_combined_metrics_final`.
6. **Dashboard stats** — config + per-source stats + score distributions appended to `nba_dash_*` tables.

## Companion Agent Skills

These are registered as their own skills and trigger directly from user prompts:

- **`fde-nba-scores-foundry-skill`** — deploy the NBA Insights Agent to a customer's TD instance
- **`fde-nba-scores-insights-agent`** — query `nba_dash_*` tables to summarize runs, compare runs, explain score distributions
