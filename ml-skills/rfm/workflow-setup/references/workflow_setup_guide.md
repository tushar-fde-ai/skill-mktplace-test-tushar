# RFM Workflow Setup Guide

Step-by-step guide for configuring and deploying the RFM Customer Segmentation workflow in Treasure Data.

The workflow unions customer behavioral data from multiple sources, computes per-profile Recency, Frequency, and Monetary scores, and segments customers into actionable groups.

## What is RFM?

RFM analysis segments customers based on three behavioral metrics:
- **Recency (R)**: Days since last interaction — lower is better
- **Frequency (F)**: Total number of interactions — higher is better
- **Monetary (M)**: Total spend value — higher is better

Customers are scored on each dimension (1-10 by default) and grouped into segments like Champions, Loyal, At-Risk, and Lost.

## Use Cases

- Customer segmentation for targeted marketing campaigns
- Identifying high-value customers for VIP programs
- Detecting at-risk customers (declining recency/frequency)
- Prioritizing customer engagement efforts
- Optimizing marketing spend by customer segment
- Input to MTA journey analytics for customer enrichment

## RFM Configuration Workflow

### Step 1: Gather Requirements

Walk through `requirements_doc.md` with the user — it covers Confluence folder lookup, customer name, ID Unification status, data sources, and scoring strategy.

For a quick setup without the full requirements gathering, ask the user:
1. **What is the Treasure Data database name?** (e.g., `gldn_marketing`)
2. **What is the user ID column?** (e.g., `canonical_id`)
3. **What time range should we analyze?** (all history or lookback period)
4. **How many scoring bins?** (default 10)

### Step 2: Explore the Customer's Database

Use **Trino SQL** via **tdx-skills** to discover source tables.

RFM requires tables that represent **customer interactions**. Look for:

#### Web Activity (pageviews, site visits)
- Tables like: `enriched_pageviews`, `web_events`, `page_views`
- Needed columns: time, user_id
- **Purpose**: Contributes to Recency (last visit) and Frequency (visit count)

#### Orders / Purchases
- Tables like: `enriched_orders`, `purchases`, `transactions`
- Needed columns: time, user_id, order_amount, order_status
- **Purpose**: Contributes to all three — Recency, Frequency, and Monetary

#### Email Events
- Tables like: `enriched_email_events`, `email_activity`
- Needed columns: time, user_id, event_type
- **Purpose**: Contributes to Recency and Frequency (engagement events only)

#### Sales Rep Interactions
- Tables like: `enriched_sales_rep_interactions`, `crm_activity`
- Needed columns: time, user_id
- **Purpose**: Contributes to Recency and Frequency

### Step 3: Clone the RFM Workflow Repository

```bash
git clone https://github.com/treasure-data/fde-rfm.git
cd fde-rfm/td_wf/rfm_agent
```

See `github_instructions.md` for detailed clone and setup steps.

### Step 4: Generate the Input YAML Configuration

The critical file to generate is: `td_wf/rfm_agent/config/input_params.yml`

**Reference**: Read `yaml_structure.md` for the complete YAML structure.

#### Global Parameters

```yaml
globals:
  canonical_id: canonical_id
  sink_database: td_agents
  model_type: 'custom'
  built_union_activity: yes
  archive_results: yes
  store_historic_scores: no
  time_zone: 'UTC'
  api_endpoint: 'api.treasuredata.com'
  num_bins: 10
```

Key decisions per customer:
- `canonical_id`: The user ID column name — must be consistent across all tables
- `sink_database`: Where output tables will be written
- `model_type`: `'custom'` (quartile-based scoring, 1-4 scale) or `'automl'` (PrecisionML notebook)
- `api_endpoint`: `api.treasuredata.com` (US) or `api.eu01.treasuredata.com` (EU) or `api.treasuredata.co.jp` (Japan)
- `num_bins`: Controls histogram display granularity (NOT scoring scale — scoring always uses quartiles 1-4)

#### Output Table Names (TOP LEVEL — not nested under globals)

```yaml
union_activity_table: rfm_combined_user_events
input_table: rfm_input_table
output_table: rfm_output_table
stats_table: rfm_stats
```

**IMPORTANT**: These are top-level YAML keys, NOT nested under `globals:`. The workflow references them as `${union_activity_table}`, `${input_table}`, `${output_table}`, `${stats_table}`.

#### Time Filters (TOP LEVEL — not nested under globals)

```yaml
apply_time_filter: 'no'
time_filter_type: interval
time_range_start_date: 2022-01-01
time_range_end_date: 2222-22-22
lookback_period: '-180d'
```

Options:
- **No filter** (default): `apply_time_filter: 'no'` — use all historical data
- **Range filter**: `apply_time_filter: 'yes'`, `time_filter_type: range`, set start/end dates
- **Interval filter**: `apply_time_filter: 'yes'`, `time_filter_type: interval`, set `lookback_period`

### Step 5: Table-Specific Configuration

Each table entry in `aggregate_metrics_tables` defines how that source contributes to RFM scoring.

**Reference**: Read `table_configuration.md` for per-table-type guidance.

#### Pageviews / Web Activity

```yaml
- src_table: gldn_marketing.enriched_pageviews
  name: 'pageviews'
  unixtime_col: time
  join_key: canonical_id
  order_amount: 0.0
  custom_filter:
  apply_time_filter: 'no'
  query_type:
```

- `order_amount: 0.0` — pageviews have no monetary value
- `custom_filter:` — usually no filter needed, but can filter bots
- Contributes to: **Recency**, **Frequency**

#### Orders / Purchases

```yaml
- src_table: gldn_marketing.enriched_orders
  name: 'order_events'
  unixtime_col: time
  join_key: canonical_id
  order_amount: unit_price
  custom_filter: "NOT REGEXP_LIKE(lower(order_status), 'cancel|return')"
  apply_time_filter: 'no'
  query_type:
```

- `order_amount: unit_price` — the revenue column (or `total_amount`, `revenue`)
- **CRITICAL**: `custom_filter` must exclude cancelled/returned orders
- Contributes to: **Recency**, **Frequency**, **Monetary**

#### Email Events

```yaml
- src_table: gldn_marketing.enriched_email_events
  name: 'email_activity'
  unixtime_col: time
  join_key: canonical_id
  order_amount: 0.0
  custom_filter: "event_type IN ('friendforward', 'webform', 'open', 'click', 'conversion', 'unsubscribe', 'reply')"
  apply_time_filter: 'no'
  query_type:
```

- `order_amount: 0.0` — email events have no monetary value
- **CRITICAL**: Exclude 'sent' events — only engagement events count
- Contributes to: **Recency**, **Frequency**

#### Sales Rep Interactions

```yaml
- src_table: gldn_marketing.sales_rep_interactions
  name: 'sales_rep_interactions'
  unixtime_col: time
  join_key: canonical_id
  order_amount: 0.0
  custom_filter:
  apply_time_filter: 'no'
  query_type:
```

- `order_amount: 0.0` — sales interactions have no monetary value
- Contributes to: **Recency**, **Frequency**

### Step 6: Column Discovery

For each table, identify:
1. **Time column**: `time`, `timestamp`, `event_time`, `created_at`
2. **User ID column**: `canonical_id`, `cdp_profile_id`, `user_id`
3. **Order amount column** (order tables): `unit_price`, `total_amount`, `revenue`
4. **Filter columns**: `order_status`, `event_type`

```sql
DESCRIBE database_name.table_name;
SELECT * FROM database_name.table_name LIMIT 10;

-- For orders: check distinct statuses
SELECT DISTINCT order_status, COUNT(*) FROM database_name.enriched_orders GROUP BY 1 ORDER BY 2 DESC;

-- For emails: check distinct event types
SELECT DISTINCT event_type, COUNT(*) FROM database_name.enriched_email_events GROUP BY 1 ORDER BY 2 DESC;

-- For orders: verify amount column
SELECT MIN(unit_price), MAX(unit_price), AVG(unit_price) FROM database_name.enriched_orders WHERE order_status = 'completed';
```

### Step 7: Validate the Configuration

Before finalizing, verify:
1. All table names exist in the database
2. Column names are correct for each table
3. `join_key` is consistent across all tables (same `canonical_id` column)
4. Filters exclude invalid data (cancelled orders, email sends)
5. `order_amount` column exists and has numeric values
6. `sink_database` exists and user has write permissions

### Step 8: Present to User for Approval

Show the user:
1. **Discovered tables** and what R/F/M metrics they contribute to
2. **Filters applied** — what data is excluded and why
3. **Scoring configuration** — num_bins, time filter, archive settings
4. **The generated `input_params.yml`**
5. **Request confirmation** before proceeding

**Wait for explicit confirmation before proceeding.**

After confirmation, ask:

> Would you like to push this workflow with the default project name, or would you like to provide a custom project name?

### Step 9: Deploy

Once the user has confirmed both the YAML and the project name:
1. Clone the repo: `git clone https://github.com/treasure-data/fde-rfm.git`
2. Place `input_params.yml` in `fde-rfm/td_wf/rfm_agent/config/`
3. Push the workflow to TD:
   - **Default name**: `cd rfm_prod && tdx wf push -y`
   - **Custom name**: `cd rfm_prod && tdx wf upload <custom_project_name>`
4. Run the workflow: `tdx wf run`
5. Monitor via `tdx wf sessions` and `tdx wf timeline`

## Critical Configuration Rules

### Order Amount Logic

Every table must have an `order_amount`. Only order/purchase tables should have a real column value:

| Table Type | order_amount | Reasoning |
|-----------|-------------|-----------|
| Pageviews | `0.0` | No monetary value |
| Email | `0.0` | No monetary value |
| Sales Rep | `0.0` | No monetary value |
| Orders | `unit_price` (or actual column) | Orders ARE the monetary source |

### Common Mistakes to Avoid

**1. Missing order status filter**
```yaml
# BAD — includes cancelled orders in Monetary score
custom_filter: ""
# GOOD
custom_filter: "NOT REGEXP_LIKE(lower(order_status), 'cancel|return')"
```

**2. Not excluding email 'sent' events**
```yaml
# BAD — sends inflate Frequency scores
custom_filter: ""
# GOOD
custom_filter: "event_type IN ('open', 'click', 'conversion')"
```

**3. Inconsistent join keys across tables**
```yaml
# BAD — will produce incorrect per-profile scores
- join_key: canonical_id
- join_key: user_id
# GOOD
- join_key: canonical_id
- join_key: canonical_id
```

**4. Wrong order_amount on non-order tables**
```yaml
# BAD — pageviews don't have unit_price
order_amount: unit_price
# GOOD
order_amount: 0.0
```

## GitHub Repository

The production RFM workflow code is at:
```
https://github.com/treasure-data/fde-rfm/tree/main/td_wf/rfm_agent
```

After generating `input_params.yml`, place it in `td_wf/rfm_agent/config/input_params.yml`.
