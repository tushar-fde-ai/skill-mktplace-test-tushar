---
name: fde-rfm-workflow-setup
description: |
  Configure and deploy the RFM (Recency, Frequency, Monetary) customer segmentation workflow in Treasure Data. Covers requirements gathering, database exploration, YAML generation, and deployment. Trigger when users want to set up RFM, configure RFM, generate input_params.yml for RFM, or deploy the RFM workflow.
---

# RFM Workflow Setup

Step-by-step guide for configuring and deploying the RFM Customer Segmentation workflow in Treasure Data.

The workflow unions customer behavioral data from multiple sources, computes per-profile Recency, Frequency, and Monetary scores, and segments customers into actionable groups.

## What is RFM?

RFM analysis segments customers based on three behavioral metrics:
- **Recency**: How recently did the customer interact or purchase?
- **Frequency**: How often do they interact or purchase?
- **Monetary**: How much do they spend?

**Key outputs:**
- Per-profile R/F/M scores (1-10 scale by default)
- Customer segment labels (Champions, Loyal, At-Risk, Lost, etc.)
- Distribution statistics for each metric
- Union activity table combining all source data

## Use Cases

- Customer segmentation for targeted marketing campaigns
- Identifying high-value customers for VIP programs
- Detecting at-risk customers (low recency/frequency)
- Prioritizing customer engagement efforts
- Optimizing marketing spend by customer segment

## RFM Configuration Workflow

### Step 1: Gather Requirements

Walk through `references/requirements_doc.md` with the user — it covers Confluence folder lookup, customer name, ID Unification status, data sources, and scoring strategy.

For a quick setup without the full requirements gathering, ask the user:
1. **What is the Treasure Data database name?** (e.g., `gldn_marketing`)
2. **What is the user ID column?** (e.g., `canonical_id`)
3. **What time range should we analyze?** (all history or lookback period)
4. **How many scoring bins?** (default 10 for 1-10 scale)

### Step 2: Explore the Customer's Database

Use **Trino SQL** via **tdx-skills** to discover source tables.

RFM requires tables that represent **customer interactions**. Look for:

#### Web Activity (pageviews, site visits)
- Tables like: `enriched_pageviews`, `web_events`, `page_views`
- Needed columns: time, user_id
- **Purpose**: Contributes to Recency and Frequency

#### Orders / Purchases
- Tables like: `enriched_orders`, `purchases`, `transactions`
- Needed columns: time, user_id, order_amount, order_status
- **Purpose**: Contributes to Recency, Frequency, and Monetary

#### Email Events
- Tables like: `enriched_email_events`, `email_activity`
- Needed columns: time, user_id, event_type
- **Purpose**: Contributes to Recency and Frequency

#### Sales Rep Interactions
- Tables like: `enriched_sales_rep_interactions`, `crm_activity`
- Needed columns: time, user_id
- **Purpose**: Contributes to Recency and Frequency

### Step 3: Clone the RFM Workflow Repository

**Reference**: Read `references/github_instructions.md` for detailed clone and setup steps.

```bash
git clone https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod.git
cd ps_ml_analytics_team_solutions_prod/rfm_prod
```

### Step 4: Generate the Input YAML Configuration

The critical file to generate is: `rfm_prod/config/input_params.yml`

**Reference**: Read `references/yaml_structure.md` for the complete YAML structure.
**Reference**: Read `references/workflow_setup_guide.md` for the full step-by-step generation guide.

### Step 5: Table-Specific Configuration

Each table entry in `aggregate_metrics_tables` defines how that source contributes to RFM scoring.

**Reference**: Read `references/table_configuration.md` for per-table-type guidance.

### Step 6: Column Discovery

For each table, identify:
1. **Time column**: `time`, `timestamp`, `event_time`
2. **User ID column**: `canonical_id`, `cdp_profile_id`, `user_id`
3. **Order amount column** (order tables): `unit_price`, `total_amount`, `revenue`
4. **Filter columns**: `order_status`, `event_type`

```sql
DESCRIBE database_name.table_name;
SELECT * FROM database_name.table_name LIMIT 10;
SELECT DISTINCT order_status FROM database_name.orders LIMIT 20;
SELECT DISTINCT event_type FROM database_name.email_events LIMIT 20;
```

### Step 7: Validate the Configuration

Before finalizing, verify:
1. All table names exist in the database
2. Column names are correct (time_col, join_key, order_amount)
3. `join_key` is consistent across all tables
4. Filters exclude invalid data (cancelled orders, email sends)
5. `sink_database` exists and user has write permissions

### Step 8: Present to User for Approval

Show the user:
1. **Discovered tables** and their role (which R/F/M metrics they contribute to)
2. **Filters applied** — what data is excluded and why
3. **The generated `input_params.yml`**
4. **Request confirmation** before proceeding

### Step 9: Deploy

Once confirmed:
1. Place `input_params.yml` in `rfm_prod/config/input_params.yml`
2. Push the workflow to TD: `tdx wf push -y`
3. Run the workflow: `tdx wf run`
4. Monitor via `tdx wf sessions` and `tdx wf timeline`

## Critical Configuration Rules

### Common Mistakes to Avoid

**1. Including cancelled/returned orders**
```yaml
# BAD
custom_filter: ""
# GOOD
custom_filter: "NOT REGEXP_LIKE(lower(order_status), 'cancel|return')"
```

**2. Including email "sent" events**
```yaml
# BAD
custom_filter: "event_type IN ('sent', 'open', 'click')"
# GOOD
custom_filter: "event_type IN ('open', 'click', 'conversion')"
```

**3. Inconsistent join keys**
```yaml
# BAD
- join_key: canonical_id
- join_key: user_id
# GOOD — same across all tables
- join_key: canonical_id
- join_key: canonical_id
```

## GitHub Repository

The production RFM workflow code is at:
```
https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod/tree/main/rfm_prod
```
