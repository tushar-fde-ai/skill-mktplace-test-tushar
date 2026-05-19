# FDE Skills

Templatized skill library for deploying ML solutions and custom AI agents on Treasure Data, used by the FDE team inside Treasure Work / Claude Code.

## How routing works

Each solution folder (e.g. `ml-skills/mta/`, `general-skills/custom-audience-agent/`) contains a `SKILL.md` whose YAML frontmatter `description` is what the skill registry uses to match user requests. There is no central router file — the registry compares the user's prompt against every registered skill's description and loads the matching one directly.

```
fde-skills/
├── README.md                          ← you are here (human docs)
├── .claude-plugin/marketplace.json    ← registers leaf skills with the loader
│
├── ml-skills/
│   ├── README.md                      ← category overview, no routing
│   ├── rfm/         SKILL.md + workflow-setup/, agent-skills/, prod-docs/
│   ├── mta/         SKILL.md + workflow-setup/, agent-skills/, prod-docs/
│   ├── nba-scores/  SKILL.md + workflow-setup/, agent-skills/, prod-docs/
│   └── nbp/         SKILL.md + workflow-setup/, agent-skills/, prod-docs/
│
└── general-skills/
    ├── README.md
    └── custom-audience-agent/   SKILL.md + agent-setup/, prod-docs/
```

Sub-task folders (`workflow-setup/`, `agent-skills/`, `prod-docs/`) currently each carry their own `SKILL.md` for backward compatibility, but the entry point for any user request is the solution-level `SKILL.md`.

## Adding a new skill

1. Create a folder under `ml-skills/` or `general-skills/`.
2. Add a `SKILL.md` with YAML frontmatter:
   - `name` — must be globally unique; prefix with `fde-` (e.g. `fde-cltv`).
   - `description` — concrete trigger phrases the registry will match against.
3. Create sub-folders (`workflow-setup/`, `agent-skills/`, `prod-docs/`) and put supporting material in `references/`.
4. Add the new skill paths to `.claude-plugin/marketplace.json`.

## Repeating folder template

```
<solution>/
├── SKILL.md
├── workflow-setup/      # TD workflow configuration (ML skills only)
│   ├── SKILL.md
│   └── references/      # YAML templates, table config, GitHub instructions
├── agent-skills/        # Companion LLM agent (Foundry or Claude skill)
│   ├── SKILL.md
│   ├── foundry-agent/   # TD AI Foundry agent setup
│   └── <claude-skill>/  # Claude skill with domain-specific references
└── prod-docs/           # Production docs, runbooks, customer-facing material
    ├── SKILL.md
    └── references/
```
