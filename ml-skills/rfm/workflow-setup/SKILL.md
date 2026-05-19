---
name: fde-rfm-workflow-setup
description: |
  RFM (Recency, Frequency, Monetary) analysis workflow for Treasure Data. Use this skill to configure RFM customer segmentation workflows. Trigger when users mention RFM, customer segmentation, recency/frequency/monetary analysis, or identifying high-value customers.
---

# RFM Analysis Workflow

This skill guides you through configuring and deploying RFM (Recency, Frequency, Monetary) analysis in Treasure Data.

## What is RFM?

RFM analysis segments customers based on three key behavioral metrics:
- **Recency**: How recently did the customer interact or purchase?
- **Frequency**: How often do they interact or purchase?
- **Monetary**: How much do they spend?

## Use Cases

- Customer segmentation for targeted marketing campaigns
- Identifying high-value customers for VIP programs
- Detecting at-risk customers (low recency/frequency)
- Prioritizing customer engagement efforts
- Optimizing marketing spend by customer segment

## RFM Configuration Workflow

### Step 1: Gather Requirements

Ask the user:
**What is the Treasure Data database name you want to analyze?** (e.g., `gldn_marketing`, `customer_analytics`)

### Step 2: Explore the Customer's Database

**IMPORTANT**: Always use **Trino/Presto SQL queries** to explore the Treasure Data database. Use **td-skills** for running SQL queries and exploring workflows.

Look for tables that contain:

#### For Pageviews/Activity:
- Tables with names like: `enriched_pageviews`, `web_events`, `page_views`, `user_activity`
- Required columns: timestamp/time column, user ID column
- **Purpose**: Tracks customer engagement (contributes to Recency and Frequency)

#### For Orders/Purchases:
- Tables with names like: `enriched_orders`, `purchases`, `transactions`, `order_events`
- Required columns: timestamp, user ID, order amount, order status
- **Purpose**: Tracks purchase behavior (contributes to all three: Recency, Frequency, Monetary)

#### For Email Events:
- Tables with names like: `enriched_email_events`, `email_activity`, `email_engagement`
- Required columns: timestamp, user ID, event type (open, click, etc.)
- **Purpose**: Tracks email engagement (contributes to Recency and Frequency)

#### For Sales Interactions:
- Tables with names like: `sales_rep_interactions`, `crm_activity`, `sales_events`
- Required columns: timestamp, user ID
- **Purpose**: Tracks sales touchpoints (contributes to Recency and Frequency)

**Reference**: Read `../references/table_discovery.md` for detailed Trino/Presto SQL queries to explore databases.

### Step 3: Clone the RFM Workflow Repository

The RFM production workflow is located at:
```
https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod/tree/main/rfm_prod
```

Clone this repository to a working directory:
```bash
git clone https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod.git
cd ps_ml_analytics_team_solutions_prod/rfm_prod
```

### Step 4: Generate the Input YAML Configuration

The critical file to generate is: `rfm_prod/config/input_params.yml`

**Reference**: Read `docs/yaml_structure.md` for the complete YAML structure and all parameters.

#### Global Parameters

```yaml
globals:
  canonical_id: canonical_id              # The user ID column name
  sink_database: gldn_marketing           # Output database
  model_type: 'custom'                    # Always 'custom' for PS code
  time_zone: 'UTC'
  api_endpoint: 'api.treasuredata.com'   # Customer's TD endpoint
  num_bins: 10                            # RFM score bins (1-10 scale)
```

#### Aggregate Metrics Tables

For each discovered table, create an entry following this pattern:

```yaml
aggregate_metrics_tables:
  - src_table: database_name.table_name
    name: 'descriptive_name'
    unixtime_col: time_column_name
    join_key: user_id_column_name
    order_amount: 0.0  # or actual amount column for orders
    custom_filter: ""  # SQL WHERE clause conditions
    apply_time_filter: 'no'
    query_type: ""     # Leave blank for YML params
```

**Reference**: Read `docs/table_configuration.md` for detailed guidance on configuring each table type.

### Step 5: Table-Specific Configuration Rules

#### Pageviews Configuration
```yaml
- src_table: database_name.enriched_pageviews
  name: 'pageviews'
  unixtime_col: time
  join_key: canonical_id
  order_amount: 0.0
  custom_filter: ""
  apply_time_filter: 'no'
  query_type: ""
```
- `order_amount: 0.0` (no monetary value)
- `custom_filter: ""` (usually no filter needed)
- Name examples: `pageviews`, `web_activity`, `site_visits`

#### Orders Configuration
```yaml
- src_table: database_name.enriched_orders
  name: 'order_events'
  unixtime_col: time
  join_key: canonical_id
  order_amount: unit_price
  custom_filter: "NOT REGEXP_LIKE(lower(order_status), 'cancel|return')"
  apply_time_filter: 'no'
  query_type: ""
```
- `order_amount: unit_price` (or the column with order value)
- **CRITICAL**: `custom_filter` must exclude cancelled/returned orders
- Look for status columns to filter valid orders only
- Name examples: `order_events`, `purchases`, `transactions`

#### Email Events Configuration
```yaml
- src_table: database_name.enriched_email_events
  name: 'email_activity'
  unixtime_col: time
  join_key: canonical_id
  order_amount: 0.0
  custom_filter: "event_type IN ('friendforward', 'webform', 'open', 'click', 'conversion', 'unsubscribe', 'reply')"
  apply_time_filter: 'no'
  query_type: ""
```
- `order_amount: 0.0`
- **CRITICAL**: Exclude 'sent' events - include only engagement events (open, click, etc.)
- Analyze the event_type column to determine valid values
- Name examples: `email_activity`, `email_engagement`

#### Sales Interactions Configuration
```yaml
- src_table: database_name.sales_rep_interactions
  name: 'sales_rep_interactions'
  unixtime_col: time
  join_key: canonical_id
  order_amount: 0.0
  custom_filter: ""
  apply_time_filter: 'no'
  query_type: ""
```
- `order_amount: 0.0`
- `custom_filter: ""` (usually no filter)
- Name examples: `sales_rep_interactions`, `crm_activity`

### Step 6: Column Name Discovery

For each table, you need to identify:
1. **Time column**: Usually `time`, `timestamp`, `event_time`, `created_at`
2. **User ID column**: Usually `canonical_id`, `cdp_profile_id`, `user_id`, `customer_id`
3. **Order amount column** (for order tables): Usually `unit_price`, `total_amount`, `order_value`, `revenue`
4. **Status/Event type columns**: For filtering (e.g., `order_status`, `event_type`)

Use Trino/Presto SQL queries like:
```sql
-- Get table schema
DESCRIBE database_name.table_name;

-- Sample data to understand column values
SELECT * FROM database_name.table_name LIMIT 10;

-- For orders: check distinct order statuses
SELECT DISTINCT order_status FROM database_name.orders LIMIT 20;

-- For emails: check distinct event types
SELECT DISTINCT event_type FROM database_name.email_events LIMIT 20;
```

**Reference**: Read `../references/column_patterns.md` for common column naming patterns.

Use td-skills to run these queries and explore the database structure.

### Step 7: Validate the Configuration

Before finalizing, verify:
1. ✅ All table names are valid and exist in the database
2. ✅ Column names are correct (time_col, join_key, order_amount)
3. ✅ Filters are appropriate for each table type
4. ✅ The canonical_id (join_key) is consistent across all tables
5. ✅ The sink_database exists and user has write permissions

Manually review the generated YAML configuration to ensure correctness.

### Step 8: Present to User for Approval

Show the user:
1. **The discovered tables** and what they'll be used for
2. **The generated `input_params.yml` file**
3. **Any assumptions or decisions made** (e.g., filters applied)
4. **Request confirmation** before proceeding

Example presentation:
```
I've discovered the following tables in your database:

1. gldn_marketing.enriched_pageviews
   - Will track website engagement
   - Contributes to: Recency, Frequency

2. gldn_marketing.enriched_orders
   - Will track purchase behavior
   - Filtering out: cancelled and returned orders
   - Contributes to: Recency, Frequency, Monetary

3. gldn_marketing.enriched_email_events
   - Will track email engagement
   - Including only: open, click, conversion events
   - Contributes to: Recency, Frequency

Configuration details:
- Join key: canonical_id (consistent across all tables)
- Time column: time (in all tables)
- Order amount: unit_price

Please review the generated input_params.yml file and confirm if this looks correct.
```

### Step 9: Deploy (Future Enhancement)

Once approved, the workflow can be:
1. Pushed to the customer's Treasure Data workflow scheduler
2. Executed to generate RFM scores
3. Monitored for results

(Note: Deployment automation will be added in future versions of this skill)

## Critical Configuration Rules

### ❌ Common Mistakes to Avoid

**1. Including invalid orders**
```yaml
# DON'T DO THIS
custom_filter: ""  # No filter - includes cancelled orders
```
✅ **DO THIS**
```yaml
custom_filter: "NOT REGEXP_LIKE(lower(order_status), 'cancel|return')"
```

**2. Including email "sent" events**
```yaml
# DON'T DO THIS - sent is not engagement
custom_filter: "event_type IN ('sent', 'open', 'click')"
```
✅ **DO THIS**
```yaml
custom_filter: "event_type IN ('open', 'click', 'conversion')"
```

**3. Inconsistent join keys**
```yaml
# DON'T DO THIS
- join_key: canonical_id
- join_key: user_id  # These might not match!
```
✅ **DO THIS**
```yaml
- join_key: canonical_id
- join_key: canonical_id  # Same across all tables
```

## Progressive Disclosure

This SKILL.md provides the RFM-specific workflow. For detailed information:

- **Table discovery SQL queries**: Read `../references/table_discovery.md`
- **Complete YAML structure**: Read `docs/yaml_structure.md`
- **Table-specific configuration**: Read `docs/table_configuration.md`
- **Column name patterns**: Read `../references/column_patterns.md`

## Example Complete Configuration

Here's a complete example of a generated `input_params.yml`:

```yaml
globals:
  canonical_id: canonical_id
  sink_database: gldn_marketing
  model_type: 'custom'
  time_zone: 'UTC'
  api_endpoint: 'api.treasuredata.com'
  num_bins: 10

  union_activity_table: rfm_combined_user_events
  input_table: rfm_input_table
  output_table: rfm_output_table
  stats_table: rfm_stats

  apply_time_filter: 'no'

aggregate_metrics_tables:
  - src_table: gldn_marketing.enriched_pageviews
    name: 'pageviews'
    unixtime_col: time
    join_key: canonical_id
    order_amount: 0.0
    custom_filter: ""
    apply_time_filter: 'no'
    query_type: ""

  - src_table: gldn_marketing.enriched_orders
    name: 'order_events'
    unixtime_col: time
    join_key: canonical_id
    order_amount: unit_price
    custom_filter: "NOT REGEXP_LIKE(lower(order_status), 'cancel|return')"
    apply_time_filter: 'no'
    query_type: ""

  - src_table: gldn_marketing.enriched_email_events
    name: 'email_activity'
    unixtime_col: time
    join_key: canonical_id
    order_amount: 0.0
    custom_filter: "event_type IN ('open', 'click', 'conversion')"
    apply_time_filter: 'no'
    query_type: ""
```

## Important Notes

- **Always use Trino/Presto SQL syntax** for all queries
- **Use td-skills** to execute SQL and explore workflows
- **Always exclude cancelled and returned orders** from order tables
- **For email events, include only engagement events** (open, click, etc.), not "sent"
- **The canonical_id must be consistent** across all tables for proper joining
- **Time filters are optional** - default is to use all historical data
- **The query_type field** should be left blank unless writing custom SQL

## Error Handling

If you encounter:
- **Tables not found**: Ask user to verify database name and table names
- **Column not found**: Suggest similar column names using `../references/column_patterns.md`, ask user to confirm
- **Invalid YAML**: Review the YAML structure manually and fix issues

## Tips for Success

1. **Start with discovery**: Always explore the database first using td-skills before generating config
2. **Understand the data**: Look at sample rows to understand data structure
3. **Be explicit about filters**: Show the user what data will be included/excluded
4. **Validate assumptions**: Ask the user to confirm when uncertain
5. **Document decisions**: Explain why certain configurations were chosen

## GitHub Repository

The production RFM workflow code is maintained at:
```
https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod/tree/main/rfm_prod
```

This repository contains:
- SQL templates for RFM calculation
- Workflow orchestration code
- Configuration validation scripts
- Dashboard templates
- Documentation

After generating the `input_params.yml`, place it in `rfm_prod/config/input_params.yml` within the cloned repository.
