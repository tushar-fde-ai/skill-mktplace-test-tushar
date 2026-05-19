# ML Skills

ML workflow skills for Treasure Data: customer segmentation, attribution, recommendations, engagement scoring.

Each subfolder is a self-contained solution with its own `SKILL.md` entry point. The skill registry routes user requests directly to the matching solution — there is no router at this level.

| Folder | Solution | Status |
|--------|----------|--------|
| `rfm/` | RFM (Recency, Frequency, Monetary) customer segmentation | Ready |
| `mta/` | Multi-Touch Attribution (Markov, Shapley, journey analytics) | Ready |
| `nba-scores/` | NBA — Next Best Channel / Time / Campaign engagement scores | Ready |
| `nbp/` | NBP — Next Best Product recommendations (Hivemall / PrecisionML) | Ready |

## Shared workflow pattern

Every ML solution follows roughly the same setup steps:
1. Ask for the customer's TD database name
2. Explore tables via Trino SQL (use `tdx-skills:tdx-basic`)
3. Identify columns (time, user_id, amounts, statuses)
4. Generate `input_params.yml` from the solution's reference templates
5. Clone the GitHub repo for that workflow
6. Validate config, present for approval, deploy

## Sub-folder convention

Each solution folder contains:
- `SKILL.md` — entry point, routing within the solution
- `workflow-setup/` — `SKILL.md` + `references/` for configuring the TD workflow
- `agent-skills/` — companion LLM agent (Foundry agent template + optional Claude skill)
- `prod-docs/` — production documentation and runbooks
