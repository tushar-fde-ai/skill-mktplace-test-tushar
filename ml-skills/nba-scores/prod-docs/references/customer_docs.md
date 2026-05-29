# NBA Engagement Scores — Customer-Facing Documentation

**Confluence folder setup** (locating the customer folder, creating FDE Solutions and NBA sub-folders) is handled by `../../../shared/confluence_folder_setup.md`. Pass `solution-folder-name: NBA Engagement Scores` and title variants: `NBA Engagement Scores`, `Next Best Action`, `NBA`, `Engagement Scores`.

**Phase 6 documentation page set** (5 standard Confluence pages, create order, keep-current rules) is handled by `../../../shared/customer_docs_pattern.md`.

---

## NBA-Specific Page Content

The sections below provide the NBA-specific content for each of the 5 Phase 6 pages. Read alongside `../../../shared/customer_docs_pattern.md` which defines the page titles, create order, and update rules.

### 1. Architecture

Title: `NBA Engagement Scores Architecture - <Customer>`

Content to include:
- TD Workflow project name (default: `nba_eng_prod`, or customer-specific name if overridden)
- GitHub repo: `https://github.com/treasure-data-ps/nba_eng_scores`
- Workflow entry point: `nba_eng_launch.dig`
- Config file: `td_wf/config/input_params.yml`
- Companion Foundry agent project name (default: `NBA Engagement Scores`)
- Scoring strategy deployed (`percentile` / `quartile` / `minmax`)
- Source tables configured (list each `src_table` entry)
- Output table joined to Parent Segment: `nba_combined_metrics_final`
- Dashboard: whether TI dashboard was deployed (`create_dashboard: yes/no`)

### 2. Behavior Summary

Title: `NBA Engagement Scores Behavior - <Customer>`

Plain-English narrative for non-technical stakeholders. Before authoring, read `../../../shared/customer_docs_pattern.md` guidance on keeping this accessible.

Cover:
- What data sources feed the model and the lookback window used
- How Next Best Channel scores are derived (UTM parsing + channel regex)
- How Next Best Time scores are derived (daypart buckets and the customer's chosen boundaries)
- How Cart Abandon is defined for this customer (lookback window, add-to-cart event name)
- How New Visitor is defined (max age, max page visits, paid-traffic exclusions)
- How scores are activated — which Parent Segment the output is joined to, and example child segment use cases

### 3. Eval Results

Title: `NBA Engagement Scores Eval Results - <Customer>`

Content (per `../../../shared/customer_docs_pattern.md`):
- Link to the workflow validation results (output table row counts, score distribution checks from `workflow-setup/references/eval.md`)
- Profile count in `nba_combined_metrics_final` vs Parent Segment profile count
- Cart abandon flag rate and new visitor flag rate
- Channel score null rate (flag if >70% null — indicates low UTM coverage)
- Date of last successful workflow run
- Any known limitations (e.g., low UTM coverage limiting channel scoring quality)

### 4. Runbook

Title: `NBA Engagement Scores Runbook - <Customer>`

Point to `runbook.md` in this same folder for the full operational reference. This Confluence page should surface:
- Re-deploy procedure: update `input_params.yml` → `tdx wf push -y` → `tdx wf run nba_eng_prod.nba_eng_launch`
- Re-run validation SQL (from `runbook.md` Validate Output section)
- Common failure modes table (from `runbook.md`)
- Workflow schedule (cron expression and timezone)
- Secret key location: `tdx wf secrets --project nba_eng_prod`

### 5. Access & Ownership

Title: `NBA Engagement Scores Access & Ownership - <Customer>`

Content (standard across all solutions — per `../../../shared/customer_docs_pattern.md`):
- FDE engineer owner
- Customer stakeholders with TD access
- Slack channel for support/questions
- GitHub repo location for the workflow config
- Escalation path for platform-level issues
