# RFM Workflow — GitHub & Deployment Instructions

## Repository

The production RFM workflow code is maintained at:
```
https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod
```

The RFM-specific code lives in the `rfm_prod/` directory within this monorepo.

## Clone the Repository

```bash
git clone https://github.com/treasure-data-ps/ps_ml_analytics_team_solutions_prod.git
cd ps_ml_analytics_team_solutions_prod/rfm_prod
```

## Directory Structure

```
rfm_prod/
├── config/
│   └── input_params.yml              # Customer-specific configuration (YOU GENERATE THIS)
├── config.json                        # Dashboard datamodel configuration
├── body.txt                           # Error email notification template
├── queries/
│   ├── parse_table_params.sql         # Parses YAML aggregate_metrics_tables into queryable rows
│   ├── create_agg_input_table.sql     # Aggregates union table into rfm_input_table
│   ├── rfm_custom.sql                 # Quartile-based RFM scoring (model_type: 'custom')
│   ├── rfm_ln_sum_scoring.sql         # Log-based scoring (alternative)
│   ├── rfm_minmax_sum_scoring.sql     # Min-max scaling (alternative)
│   ├── rfm_stats_summary.sql          # Per-segment statistical summary
│   ├── rfm_stats_histogram.sql        # Histogram bin distributions
│   ├── stats_model_params.sql         # Per-source run metadata
│   ├── stats_global_session_filter.sql # Session ranking
│   ├── stats_historic_agg.sql         # Historical score aggregation
│   ├── union_activity/
│   │   ├── create_activity_union_agg.sql          # Union without time filter
│   │   ├── create_activity_union_time_interval.sql # Union with interval filter
│   │   └── create_activity_union_time_range.sql    # Union with range filter
│   └── union_src_tables/
│       └── insert_src_rable.sql       # Per-source insert into union table
├── python_files/
│   ├── autobuild_segments.py          # Auto-build customer segments
│   ├── create_datamodel.py            # Create TI dashboard datamodel
│   └── scan_parent_segments.py        # Scan parent segments
├── dashboard/
│   └── [PROD]RFMTemplate-TDUI.dash    # TI dashboard template
├── rfm_launch.dig                     # ENTRY POINT — main workflow orchestrator
├── rfm_union_all_activity.dig         # Union all source tables
├── rfm_custom.dig                     # Run custom quartile scoring
├── rfm_automl.dig                     # Run PrecisionML notebook scoring
├── rfm_agg_stats.dig                  # Generate dashboard stats tables
├── rfm_datamodel_create.dig           # Create TI datamodel
└── rfm_datamodel_build.dig            # Build/refresh TI datamodel
```

## Workflow Execution Flow

```
rfm_launch.dig (entry point)
├── 1. Create sink database if not exists
├── 2. rfm_union_all_activity.dig (if built_union_activity: yes)
│   ├── Parse YAML params → rfm_input_params table
│   ├── Create empty union table
│   ├── Loop through each source table (parallel)
│   │   └── Insert per-profile aggregated rows into union table
│   └── Create rfm_input_table (aggregate across sources)
├── 3. rfm_${model_type}.dig
│   └── rfm_custom.dig → quartile scoring → rfm_output_table
│       OR rfm_automl.dig → PrecisionML notebook
├── 4. rfm_agg_stats.dig
│   ├── rfm_stats (per-segment summary)
│   ├── rfm_stats_histogram (score distributions)
│   ├── rfm_stats_model_params (per-source metadata)
│   ├── rfm_stats_global_session_filter (session ranking)
│   └── rfm_stats_daily_agg (historical scores)
└── 5. rfm_datamodel_create.dig + rfm_datamodel_build.dig (if create_dashboard: yes)
```

## Configuration

Place the generated `input_params.yml` in `rfm_prod/config/input_params.yml`.

See `workflow_setup_guide.md` for the full configuration walkthrough and `yaml_structure.md` for the parameter reference.

## Deploy to Treasure Data

### Option 1: Default Project Name

Push using the folder name as the project name:

```bash
cd rfm_prod
tdx wf push -y
```

### Option 2: Custom Project Name

Push under a custom workflow project name:

```bash
cd rfm_prod
tdx wf upload <custom_project_name>
```

## Run the Workflow

```bash
tdx wf run
```

## Monitor Execution

```bash
# Check running sessions
tdx wf sessions --status running

# Follow the execution timeline
tdx wf timeline --follow

# Check attempt details
tdx wf attempt <attempt_id> tasks
tdx wf attempt <attempt_id> logs +<task_name>
```

## Re-deploy After Config Changes

1. Update `config/input_params.yml` with the new configuration
2. Push the updated workflow: `tdx wf push -y`
3. Run: `tdx wf run`
4. Verify output tables have expected row counts

## Access Requirements

- GitHub access to the `treasure-data-ps` organization
- TD CLI (`tdx`) installed and authenticated against the customer's TD instance
- Write permissions to the target `sink_database` in TD
