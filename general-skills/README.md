# General Skills

Non-ML FDE solutions — custom AI agents and analytics tooling deployed for customers.

Each subfolder is a self-contained solution with its own `SKILL.md` entry point. The skill registry routes user requests directly to the matching solution — there is no router at this level.

| Folder | Solution | Status |
|--------|----------|--------|
| `custom-audience-agent/` | Custom AI Foundry agent for CDP parent segment / audience analysis | In Progress |
| `custom-analytics-agent/` | General-purpose analytics agent for any TD database | Scaffold only |
| `segment-analytics/` | Ad-hoc analytics + reporting templates for CDP segments | Scaffold only |

## Sub-folder convention

Each solution folder follows this pattern:
- `SKILL.md` — entry point
- `agent-setup/` — `SKILL.md` + `references/` for configuring and deploying the agent
- `prod-docs/` — production documentation and references
