---
name: fde-rfm
description: |
  RFM (Recency, Frequency, Monetary) Customer Segmentation for Treasure Data. Configures workflows that union customer behavioral data from multiple sources (pageviews, orders, email, sales), compute per-profile Recency, Frequency, and Monetary scores, and segment customers into actionable groups. Trigger on: RFM, recency frequency monetary, customer segmentation, high-value customers, at-risk customers, VIP segmentation, customer scoring, RFM workflow setup, RFM configuration, RFM runbook, RFM output tables. Use this skill — not ml-solutions-skills:rfm-workflow — when setting up RFM for a customer as part of an FDE/PS engagement.
---

# RFM Customer Segmentation

RFM workflow that unions customer behavioral data from multiple sources, computes per-profile Recency, Frequency, and Monetary scores using quartile-based binning, and segments customers into actionable groups for targeted marketing.

## How to Start

Read `../shared/SKILL.md` — it defines the entry point (new vs. resume), the phase sequence, and when to read each shared pattern file. Follow that flow. This file provides the RFM-specific context you need at each phase.

## Phase Summary

| Phase | What happens |
|---|---|
| Phase 1 | Data exploration — discover tables, classify by RFM source type, confirm schema |
| Phase 2 | Create Confluence folder + Current Project State + Requirements Doc |
| Phase 3 | Push minimal workflow, validate output tables |
| Phase 4 | Deploy Foundry agent, run integration check |
| Phase 5 | Re-configure workflow from filled requirements, re-validate |
| Phase 6 | Parent Segment update + example audiences, then customer-facing documentation set |

---

## RFM-Specific Context by Phase

### Phase 1: Data Exploration

Use `tdx-skills:tdx-basic` to explore the customer's TD databases. **Do not search by table name patterns.** Instead follow these steps:

**Step 1 — List every table in the customer database.**
Run `tdx tables "<database>.*"` to get the full table list.

**Step 2 — Classify each table.**
For every table, run `tdx describe` to inspect its schema. Classify it into one of three buckets:

| Bucket | Criteria |
|---|---|
| **Activity / behavior table** | Has a user ID column + a Unix timestamp column + rows represent a one-to-many customer activity (one row = one event or interaction per customer). **Any such table is a candidate for RFM** — if it captures a meaningful customer touchpoint it contributes at minimum to Recency and Frequency. When in doubt, include it and let the customer decide. |
| **Profile / dimension table** | Has a user ID column but rows represent one record per customer (master tables, attributes, membership tiers). Not a source table. |
| **System / output table** | Workflow output, audit logs, ML result tables. Not a source table. |

**Step 3 — Within activity tables, classify by RFM source type.**
For each candidate table, determine which RFM type it maps to:

| RFM Source Type | What to look for |
|---|---|
| **Pageviews / web activity** | URL column (`td_url`, `page_url`), referrer column — web events from JS SDK. Contributes to R and F. |
| **Email events** | `event_type` column with values like open/click/send, `campaign_name` — engagement events from ESP. Contributes to R and F (exclude `sent` events). |
| **Sales / CRM interactions** | `source`, `topic`, `rep_name`, appointment or test-drive columns — offline or in-person customer touchpoints. Contributes to R and F. |
| **Orders / purchases / services** | `order_status`, `unit_price`, revenue, or `cost` column — conversion or revenue events. Contributes to R, F, and **M**. |
| **Customer touchpoint** | Any other one-to-many activity table with a timestamp (reviews, support tickets, loyalty check-ins, etc.) that doesn't fit the above. Contributes to R and F. |

**Step 4 — Present the recommended list to the user before proceeding.**
First output a markdown table directly to the user showing ALL activity tables found with: table name, RFM source type, R/F/M contributions, and a one-line reason (include or exclude). Then immediately follow with an `AskUserQuestion` call to get confirmation. The question text must be short (one sentence asking for confirmation) — do NOT embed the table inside the question string. Put the choices in the options array only (e.g. "Confirm as listed", "Exclude one or more tables"). Do not skip this step.

**Step 5 — For each confirmed source table, note down:**
- Full table name
- User ID column name (`canonical_id`, `cdp_profile_id`, `user_id`)
- Timestamp column name
- Order amount column (order tables only — `unit_price`, `total_amount`, `revenue`)
- Filter needed (order status exclusion, email send exclusion, etc.)
- Which R/F/M dimensions it contributes to

These become `input_params.yml` values.

**Confluence folder name for this solution:** `RFM Customer Segmentation`
**Title variants for fuzzy matching:** `RFM`, `RFM Analysis`, `RFM Segmentation`, `RFM Customer Segmentation`, `Customer Segmentation`

### Phase 2: Requirements Doc

Page title: `RFM Requirements Gathering - <Customer>`

Body template: `workflow-setup/references/requirements_doc.md`

Key questions the requirements doc must answer: unique user ID column (`canonical_id`), ID Unification status, monetary metric column (or none for non-commerce), time filter / lookback period, scoring bins (`num_bins`), business rules (order status filters, email event type filters), sink database name, archive and historical score settings.

### Phase 3: Push Minimal Workflow

**GitHub repo:** `https://github.com/treasure-data/fde-rfm`
**Workflow project name:** `rfm_agent`
**Workflow path:** `fde-rfm/td_wf/`
**Entry point:** `rfm_launch.dig`
**Config file:** `rfm_agent/config/input_params.yml`

For the minimal push, populate `input_params.yml` with what you know from Phase 1 (tables, user ID column, timestamp column, order amount column). Use Phase 2 answers if already available; otherwise use safe defaults.

Read `workflow-setup/references/workflow_setup_guide.md` for full parameter generation instructions.
Read `workflow-setup/references/yaml_structure.md` for the parameter reference.
Read `workflow-setup/references/table_configuration.md` for per-source-type config.
Use `workflow-setup/references/input_params_template.yml` as the starting point.

After the workflow run finishes, validate output tables following instructions in `workflow-setup/references/eval.md`.

### Phase 4: Foundry Agent Setup

**Foundry agent project name:** `RFM Customer Segmentation`
**Agent name:** `RFM Analysis Agent`

Read `agent-skills/foundry-agent/SKILL.md` for the files-to-edit table and integration check.

Populate `knowledge_bases/business_context.md` with what you learned in Phases 1–3: customer industry, source tables used, monetary metric column, output table name, and any notable data quirks.

### Phase 5: Update Workflow with Customer Requirements

After reading the filled requirements doc, present a summary of the parameter changes you plan to make before touching any files. Key areas that typically change between Phase 3 and Phase 5:

- Scoring bins (`num_bins`) — customer may prefer 5-bin quintile instead of 10-bin default
- Time filter / lookback period
- Adding or removing source table types
- Order status filter refinement (customer may have non-standard status values)
- Email event type filter (confirm which event types count as engagement)
- `auto_build_segments` toggle if customer wants named segment labels

Read `workflow-setup/references/workflow_setup_guide.md` for update mechanics.
Re-run the workflow and re-validate output tables (same checks as Phase 3).

### Phase 6: Customer Documentation

**Step 0 — Model Summary Dashboard.**
Before writing any Confluence pages, check whether a dashboard template exists and generate the HTML dashboard for the customer.
See [Model Summary Dashboard](#model-summary-dashboard) section below for the full procedure.

**Step 1 — Parent Segment attributes + example audiences.**
Read `../shared/parent_segment_update.md` for the process.
Read `prod-docs/references/parent_segment.md` for the RFM-specific attribute columns and example segment definitions. Do not read this file until the user has approved the plan.

**Step 2+ — Confluence documentation pages.**
Read `../shared/customer_docs_pattern.md` for the 5-page set.

RFM-specific content for each page is in `prod-docs/references/`:

| Page | Reference file |
|---|---|
| Architecture | `prod-docs/references/runbook.md` (architecture section) |
| Runbook | `prod-docs/references/runbook.md` |
| Customer-facing overview | `prod-docs/references/customer_docs.md` |
| Technical handoff | `prod-docs/references/technical_handoff.md` |

---

## Model Summary Dashboard

This procedure applies whenever the user asks for a model summary dashboard, a run summary, or an HTML report — AND as **Phase 6 Step 0** before writing Confluence documentation.

### Step 1 — Check for a dashboard template

Check whether the template file exists at:

```
agent-skills/rfm-analysis/references/dashboard_template.html
```

Use the Read tool to attempt to read the file. Two outcomes:

---

#### Path A — Template exists

Follow the token-substitution pattern used by the MTA and NBA-scores skills:

1. **Read** `agent-skills/rfm-analysis/references/dashboard_template.html` in full.
2. **Strip** the leading HTML comment block (everything between `<!--` and `-->` at the top of the file — this is the instruction block and must not appear in output).
3. **Run the source queries** documented in the template comment against the customer's `sink_database`. All queries use `tdx query -d <sink_database> "SQL"`.
4. **Replace every `{{TOKEN}}`** in the template with real values from the query results:
   - Numeric tokens: apply thousands separators; percentages to 1 decimal place + `%`
   - `_json` tokens: emit valid JS array/object literals — double-quoted strings, no trailing commas
   - `_rows` tokens: pre-rendered `<tr>...</tr>` HTML, injected directly into `<tbody>`
5. **Determine output path:**
   - If `.customer-configs/<customer_slug>/` exists, write to `.customer-configs/<customer_slug>/rfm_model_summary_dashboard.html`
   - Otherwise write to the current working directory as `rfm_model_summary_dashboard.html`
6. **Write** the substituted HTML to the output path.
7. **Open** the file with `mcp__tas__open_file`.

---

#### Path B — Template does not exist

Generate a self-contained HTML dashboard from scratch following the TD design system used by MTA and NBA-scores dashboards:

**Design rules (mandatory — do not deviate):**
- **Theme:** Light background `#F7F8FB`, white cards, ink `#1F2147`, muted `#6A6F8A`
- **Header:** Blue gradient `linear-gradient(135deg, #2E41A6 0%, #5867B8 100%)` with white text, tab navigation attached to the bottom of the header
- **TD color palette** (use in order for chart series):
  `["#B4E3E3","#ABB3DB","#D9BFDF","#F8E1B0","#8FD6D4","#828DCA","#C69ED0","#F5D389","#6AC8C6","#5867B8","#B37EC0","#F1C461","#44BAB8","#2E41A6","#8CC97E","#A05EB0"]`
- **Segment colors** (fixed, use consistently across all charts):
  - Champions: `#44BAB8` · Loyal Customers: `#5867B8` · Potential Loyalists: `#8FD6D4`
  - Promising: `#B4E3E3` · New Customers: `#8CC97E` · Cannot Lose Them: `#F1C461`
  - Need Attention: `#F5D389` · Hibernating: `#D9BFDF` · High Value Sleeping: `#C69ED0`
  - Lost Customers: `#ABB3DB`
- **Charts:** Chart.js 4.4.1 CDN only — no build step, no React, no Plotly
- **Font:** system font stack (`-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif`)
- **Tables:** `<thead>` background `#B4E3E3`, striped rows `#F7F8FB`

**Required tabs and content:**

| Tab | Sections |
|---|---|
| **Segment Overview** | 5 KPI cards (total profiles, avg R, avg F, avg M, champion rate) · segment donut chart · horizontal bar breakdown by segment |
| **Score Distributions** | Bar chart for R quartile distribution · bar chart for F quartile distribution · bar chart for M quartile distribution · quartile threshold reference table |
| **Segment Detail** | Full per-segment stats table (count, %, avg R/F/M, avg spend, touchpoints) · R×F heatmap colored by avg M |
| **Model Config** | Run parameters table (lookback, num_bins, user_id_col, monetary_col, sink_db) · source tables used · scoring method |

**Source queries to run** (against `sink_database`):

```sql
-- KPIs + segment stats
SELECT rfm_segment,
       COUNT(*) AS profile_count,
       ROUND(AVG(r_quartile), 2) AS avg_r,
       ROUND(AVG(f_quartile), 2) AS avg_f,
       ROUND(AVG(m_quartile), 2) AS avg_m,
       ROUND(AVG(total_spend), 2) AS avg_spend,
       ROUND(AVG(total_touchpoints), 1) AS avg_tp
FROM rfm_output_table
GROUP BY rfm_segment
ORDER BY profile_count DESC

-- Score distributions (R/F/M quartile histogram)
SELECT r_quartile, f_quartile, m_quartile, COUNT(*) AS cnt
FROM rfm_output_table
GROUP BY 1, 2, 3

-- Model parameters
SELECT * FROM rfm_stats_model_params LIMIT 1

-- R×F matrix avg M
SELECT r_quartile, f_quartile, ROUND(AVG(m_quartile), 2) AS avg_m
FROM rfm_output_table
GROUP BY 1, 2
ORDER BY 1 DESC, 2
```

**Output path and delivery:** same as Path A — write to `.customer-configs/<customer_slug>/rfm_model_summary_dashboard.html` if that directory exists, otherwise to the current directory. Open with `mcp__tas__open_file`.

---

### Step 2 — Note the dashboard path

After generating the dashboard, record its file path. Reference it in the Technical Handoff Confluence page as the "Model Summary Dashboard" deliverable.

---

## Quick Reference

- **GitHub repo:** `https://github.com/treasure-data/fde-rfm`
- - **Workflow project name:** `rfm_agent`
**Workflow path:** `fde-rfm/td_wf/`
- **Entry point:** `rfm_launch.dig`
- **Config file:** `rfm_agent/config/input_params.yml`
- **Key outputs**: `rfm_output_table` (per-profile R/F/M quartiles + segment), `rfm_stats` (per-segment distribution stats), `rfm_combined_user_events` (union activity), `rfm_stats_histogram`, `rfm_stats_model_params` (dashboard tables) in the sink database
- **Scoring**: Quartile-based (1-4 scale) using 25th/50th/75th percentile boundaries
- **Confluence folder name**: `RFM Customer Segmentation`
- **Title variants for fuzzy matching**: `RFM`, `RFM Analysis`, `RFM Segmentation`, `RFM Customer Segmentation`, `Customer Segmentation`

## Companion agent skills (separate entry points)

These are registered as their own skills — they trigger directly from user prompts, no need to route through this file:

- **`fde-rfm-foundry-skill`** — deploy the RFM Analysis AI Foundry agent template to a customer's TD instance
- **`fde-rfm-analysis-agent`** — query the RFM output tables to answer segmentation, customer value, and engagement questions with visualizations

## How It Works

1. **Union**: Combine pageviews, email events, sales interactions, orders into `rfm_combined_user_events` (per-profile aggregation per source)
2. **Compute metrics**: For each profile — min recency_days across sources (Recency), sum of total_touchpoints (Frequency), sum of total_spend (Monetary)
3. **Score**: Assign quartile scores (1-4) using 25th/50th/75th percentile boundaries, combine into `rfm_quartile` label (e.g., `R4F3M2`)
4. **Segment**: Assign customer segments based on R/F/M quartile combinations (Champions, Loyal Customers, Cannot lose them, Lost customers, etc.)
5. **Dashboard stats**: Generate per-segment statistics, histograms, and model parameters for the TI dashboard
