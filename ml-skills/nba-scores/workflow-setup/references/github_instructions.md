# NBA Engagement Scores — GitHub & Deployment Instructions

This document covers cloning the workflow repo, placing the generated `input_params.yml`, and pushing the workflow to a TD account.

## Repository

**Public repo**: `https://github.com/treasure-data-ps/nba_eng_scores`

The repo contains two top-level projects:
- `td_wf/` — the digdag workflow (TD project name: `nba_eng_prod`)
- `foundry_agent/` — the AI Foundry NBA Insights Agent (TD project name: `NBA Engagement Scores`)

This document covers the workflow side (`td_wf/`). For the agent, see `agent-skills/foundry-agent/SKILL.md`.

## Step 1: Clone the Repository

```bash
git clone https://github.com/treasure-data-ps/nba_eng_scores.git
cd nba_eng_scores/td_wf
```

## Step 2: Inspect the Project Structure

```
td_wf/
├── tdx.json                                 # TD workflow project metadata
├── nba_eng_launch.dig                       # MAIN workflow entry point
├── nba_union_all_activity.dig               # Union of all source tables → nba_combined_user_events
├── nba_next_best_time_metrics.dig           # Time-of-day affinity scoring
├── nba_next_best_channel_metrics.dig        # Channel affinity scoring (UTM-driven)
├── nba_next_best_campaign_metrics.dig       # Cart-abandon, new-visitor, ad-engagement flags
├── nba_metrics_table.dig                    # Combine all temp tables → nba_combined_metrics_final
├── nba_dash_stats.dig                       # Populate the three nba_dash_* tables
├── nba_datamodel_create.dig                 # (Optional) Create TI datamodel
├── nba_datamodel_build.dig                  # (Optional) Refresh/build TI datamodel
├── nba_cleanup_runner.dig                   # Drop temp tables when cleanup_temp_tables: 'yes'
├── config/
│   └── input_params.yml                     # ← THE FILE YOU NEED TO GENERATE
├── queries/                                 # Per-step SQL templates (reusable across customers)
│   ├── dash/
│   ├── next_best_action_metrics/
│   ├── next_best_campaign/
│   ├── next_best_channel/
│   ├── next_best_time/
│   └── stats_archive/
├── sql/                                     # Workflow-internal SQL
│   ├── final/
│   ├── nb_time/
│   ├── sample_tables/
│   └── union/
├── python_files/                            # Helper scripts (auto-build segments, datamodel create)
├── dashboard/
│   └── NBA_Engagement_Scores.dash           # TI dashboard template (uploaded manually)
└── body.txt                                 # Email body for failure notifications
```

You should NOT need to edit anything in `queries/`, `sql/`, or `python_files/` for a standard customer deployment. Custom channel taxonomies (e.g., adding `tiktok_jp` as a distinct channel) require editing `queries/next_best_channel/` SQL templates — flag this to the user as a customization beyond standard config.

## Step 3: Place the Generated `input_params.yml`

Drop the YAML you generated into:

```
nba_eng_scores/td_wf/config/input_params.yml
```

Overwrite the existing file. The existing one is the last customer's config — committing it to git is fine if you're working from a customer-specific branch, but DO NOT push customer-specific configs back to `main`.

## Step 4: Authenticate to the Customer's TD Account

```bash
tdx auth setup            # if not already configured
tdx use <profile_name>    # switch to the customer's profile/account
tdx databases             # confirm you're in the right account by listing databases
```

## Step 5: Push the Workflow

From inside `nba_eng_scores/td_wf`:

### Default project name (`nba_eng_prod`)

```bash
tdx wf push -y
```

This uses the project name baked into `tdx.json` (`nba_eng_prod`).

### Custom project name

If the customer wants a different project name (rare, but used when running multiple isolated NBA pipelines per region or per brand):

```bash
tdx wf upload <custom_project_name>
```

`tdx wf upload` pushes under a different project name without renaming the local folder.

## Step 6: Set Workflow Secrets (if needed)

If the customer's `api_endpoint` is non-default (EU01, AP02, etc.), confirm `tdx` is targeting that region. The workflow itself reads `api_endpoint` from `input_params.yml`, but `tdx` uses your local profile for authentication.

## Step 7: Run the Workflow

```bash
tdx wf run nba_eng_prod.nba_eng_launch
```

For the very first run, monitor closely:

```bash
tdx wf sessions nba_eng_prod --status running
tdx wf timeline nba_eng_prod.nba_eng_launch --follow
```

If a step fails:

```bash
tdx wf attempt <attempt_id> tasks
tdx wf attempt <attempt_id> logs +<failed_task_name>
```

## Step 8: Verify Output Tables

After a successful run, confirm in the customer's account:

```sql
-- Final per-profile metrics
SELECT COUNT(*), COUNT(DISTINCT canonical_id) FROM <sink_database>.nba_combined_metrics_final;

-- Dashboard tables (one row per session_id)
SELECT MAX(session_id) FROM <sink_database>.nba_dash_model_metrics;
SELECT COUNT(*) FROM <sink_database>.nba_dash_stats_summary WHERE session_id = (SELECT MAX(session_id) FROM <sink_database>.nba_dash_model_metrics);
SELECT * FROM <sink_database>.nba_dash_source_tables WHERE session_id = (SELECT MAX(session_id) FROM <sink_database>.nba_dash_model_metrics);
```

## Step 9: (Optional) Schedule Recurring Runs

`nba_eng_launch.dig` ships with a monthly schedule by default:

```yaml
timezone: UTC
schedule:
  monthly>: 3,06:15:00       # 3rd of the month at 06:15 UTC
```

To change cadence or disable, edit the top of `nba_eng_launch.dig` and re-push. For schedule syntax, see https://docs.digdag.io/scheduling_workflow.html.

## Step 10: (Optional) Upload the TI Dashboard

If `create_dashboard: 'yes'` was set, the workflow created the `nba_engagement_scores_automated` datamodel. To wire the dashboard:

1. Open the TI console → Dashboards → Upload
2. Upload `nba_eng_scores/td_wf/dashboard/NBA_Engagement_Scores.dash`
3. When prompted, point the dashboard's data source to the `nba_engagement_scores_automated` datamodel
4. Save and share with the customer

## Common Failure Modes

| Symptom | Most likely cause | Fix |
|---------|-------------------|-----|
| `Table not found: <db>.<table>` | Source table doesn't exist or is in a different database | Verify `src_table` values in `input_params.yml` |
| All `cart_abandon` flags = 0 | `abandon_regexp` doesn't match the customer's add-to-cart event_type | Update `next_best_campaign.abandon_regexp` (and the `_string` copy) |
| All channel scores collapse to 'others' | Pageview UTM params are mostly NULL | Lower `top_k_channel_perc`, or fall back to referrer-based channel extraction |
| Memory error in scoring step | `lookback_period` too long for available compute | Shrink to `-90d` and rerun |
| `nba_combined_metrics_final` profile count is much smaller than Parent Segment | Source-table profiles differ from Parent Segment audience | Check the `customers` source table covers the same profile universe as the Parent Segment |
