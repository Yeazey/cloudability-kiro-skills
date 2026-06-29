# Cloudability Kiro Skills

Kiro CLI skills for IBM Cloudability — FinOps dashboards, container analytics, daily standups, and MCP context loading.

## Installation

Copy any skill folder into `~/.kiro/skills/`:

```bash
# Clone the repo
git clone https://github.com/Yeazey/cloudability-kiro-skills.git

# Copy skills you want
cp -r cloudability-kiro-skills/cldy ~/.kiro/skills/
cp -r cloudability-kiro-skills/containersdash ~/.kiro/skills/
cp -r cloudability-kiro-skills/execdash ~/.kiro/skills/
cp -r cloudability-kiro-skills/archdash ~/.kiro/skills/
cp -r cloudability-kiro-skills/dailycheckin ~/.kiro/skills/
```

## Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| `cldy` | `/cldy`, session start | Loads full Cloudability MCP context — environments, auth, tools, workflows, architecture decisions |
| `containersdash` | container dashboard, k8s costs, idle resources | Generates interactive K8s container cost dashboard with cluster/namespace/idle analysis |
| `execdash` | exec dashboard, cloud cost summary | Generates executive FinOps dashboard with MTD spend, trends, rightsizing, anomalies |
| `archdash` | multi-cloud, architecture, chip breakdown | Generates multi-cloud architecture dashboard with IaaS/PaaS/AI split and chip breakdown |
| `dailycheckin` | daily checkin, finops standup | Runs multi-agent FinOps daily standup with prioritized actions and deep-dive data |

## Prerequisites

- [Kiro CLI](https://kiro.dev) installed
- Cloudability MCP server configured (Node.js and/or Python)
- Valid Cloudability API credentials (opentoken + environment ID)

## MCP Server Setup

These skills work with two Cloudability MCP servers:

1. **Node.js MCP** — Cost reporting, views, budgets, anomalies, rightsizing, forecasts
2. **Python MCP** ([eelzinaty/cloudability-mcp-server](https://github.com/eelzinaty/cloudability-mcp-server)) — Advanced container cost allocation (fairshare model)

See `cldy/SKILL.md` for full MCP configuration details.

## Architecture

The `cldy/ARCHITECTURE.md` file documents all design decisions, system topology, and rationale. It's a living document updated as the system evolves.

## License

MIT
