---
name: cldy
description: Load full Cloudability MCP context — environments, auth, tools, workflows, and architecture decisions. Use at session start or when working with any Cloudability data.
---

# Cloudability MCP Context

Load this skill at the start of any session involving Cloudability data. It provides full context on the MCP setup, environments, available tools, and established workflows.

## Architecture Reference

Read `/Users/kingyeazey/.kiro/skills/cldy/ARCHITECTURE.md` for design decisions, rationale, and how the system works together.

## MCP Servers (3 configured, 1 primary)

### 1. `cldy-mcp-local` — **PRIMARY** ⭐
- **Binary**: `/Users/kingyeazey/.local/bin/cldy-mcp-local`
- **Package**: `cldy_mcp_local-0.2.0` (installed via `uv tool install` from `~/Downloads/cldy-mcp-local/`)
- **Runtime**: Python (FastMCP)
- **Config**: `~/.kiro/settings/mcp.json` → `cldy-mcp-local` entry
- **Auth env vars**: `CLOUDABILITY_OPEN_TOKEN`, `CLOUDABILITY_ENVIRONMENT_ID`
- **Covers**: Cost reporting, usage reporting, Kubecost container allocation, rightsizing, RI planning, RI portfolio, anomaly detection, governance, business mappings, data freshness, views, tags
- **Key tools**:
  - `run_cost_report` / `run_usage_report` — Cost and usage reporting
  - `get_kubecost_workload_costs` — Kubernetes cost allocation (cluster, namespace, pod, node, label)
  - `kubecost_list_windows` — List valid time windows for Kubecost queries
  - `get_rightsizing_recommendations` — Multi-vendor rightsizing (AWS, Azure, GCP, Containers)
  - `list_anomalies` / `get_anomaly` — Anomaly detection
  - `list_views` / `get_view` — View management
  - `list_business_dimensions` / `list_business_metrics` — Business mappings
  - `get_ri_portfolio_summary` / `get_ri_utilization` / `list_ri_recommendations` — RI management
  - `list_governance_policies` — Governance and compliance
  - `get_data_freshness_summary` / `get_pipeline_failures` — Data pipeline health
  - `list_tag_mappings` — Tag normalization rules
  - `search_dimension_values` — Dimension value lookup for filters
  - `list_cost_measures` / `list_usage_measures` — Available dimensions and metrics
- **Why primary**: Unified server covering ALL Cloudability domains including Kubecost container allocation. Replaces both the Node.js and Python MCP servers for day-to-day queries.

### 2. Node.js MCP — `cloudability` (legacy, still available)
- **Path**: `/Users/kingyeazey/Projects/bob/cldy-mcp-server-main/`
- **Runtime**: Node.js
- **Covers**: Cost reporting, views, budgets, anomalies, business mappings, rightsizing, forecasts, estimates, account groups, users
- **Config**: `~/.kiro/settings/mcp.json` → `cloudability` entry
- **Key tools**: `cldy_cost_report_run`, `cldy_cost_report_enqueue`, `list_views`, `list_budgets`, `cldy_anomalies_list`, `cldy_rightsizing_list`, `cldy_forecast_get`
- **Note**: Still used by dashboard generator projects (`npm run generate`). For interactive queries, prefer `cldy-mcp-local`.

### 3. Python MCP — `cloudability-containers` (legacy, limited)
- **Path**: `/Users/kingyeazey/Projects/bob/cloudability-mcp-server-python/`
- **Runtime**: Python 3.14+ via `uv`
- **Covers**: Budget management, account listing, estimates, forecasts
- **Config**: `~/.kiro/settings/mcp.json` → `cloudability-containers` entry
- **Note**: The container-specific endpoints (clusters, usage, labels) are **410 Gone**. Use `cldy-mcp-local` `get_kubecost_workload_costs` for container data instead.

## Environments

Environment details (IDs, tokens) are stored locally in `~/cloudability-env-keys.md` and MUST NOT be committed to version control.

### To Switch Environments
1. Update `CLOUDABILITY_OPEN_TOKEN` and `CLOUDABILITY_ENVIRONMENT_ID` in `~/.kiro/settings/mcp.json` (all MCP entries: `cldy-mcp-local`, `cloudability`, and `cloudability-containers`)
2. Update `~/cloudability-env-keys.md` to mark active env
3. Restart Kiro CLI

### Token Notes
- Opentokens expire — if you get `401 unauthorized / invalid opentoken`, ask the user for a new one
- All MCP servers use the same token + env ID
- `cldy-mcp-local` uses env var `CLOUDABILITY_OPEN_TOKEN` (with underscore), Node.js uses `CLOUDABILITY_OPENTOKEN` (no underscore)

## GitHub Repos

| Repo | What | Local Path |
|------|------|-----------|
| `Yeazey/cloudability-kiro-skills` | All Kiro skills (cldy, execdash, archdash, containersdash, dailycheckin) | `~/.kiro/skills/` |
| `Yeazey/executive_cloud_summary_CLDYMCP` | Executive dashboard project | `~/cloudability-executive-dashboard/` |
| `Yeazey/Arch_Dash_CLDYMCP` | Multi-cloud architecture dashboard project | `~/cloudability-multicloud-dashboard/` |
| `Yeazey/kubernetes-container-cost-dashboard` | Container cost dashboard + HTML | `~/container-dashboard.html` |
| `Yeazey/FinOps_Checkin_CLDYMCP` | Daily check-in multi-agent project | `~/cloudability-dailycheckin/` |

## Available Skills (4)

| Skill | Trigger | What it does |
|-------|---------|-------------|
| `/execdash` | exec dashboard, cloud cost summary | Generates executive dashboard via `npm run generate` in cloudability-executive-dashboard/ |
| `/archdash` | multi-cloud, architecture, chip breakdown | Generates multi-cloud architecture dashboard via `npm run generate` in cloudability-multicloud-dashboard/ |
| `/containersdash` | container dashboard, k8s costs, idle resources | Pulls live container data via MCP, builds HTML dashboard |
| `/dailycheckin` | daily checkin, finops standup | Runs multi-agent FinOps daily standup, outputs markdown report |

## Container Data Strategy

The Cloudability container endpoints (`/containers/clusters`, `/containers/usage`, etc.) are **deprecated (410 Gone)**. Use `cldy-mcp-local` for all container cost queries:

1. **Kubecost allocation** (`cldy-mcp-local`): Use `get_kubecost_workload_costs` with aggregate dimensions (cluster, namespace, pod, node, label, controller). Supports time windows like "7d", "30d", "month", "lastmonth". Returns fairshare-allocated costs with idle/efficiency metrics.
2. **Standard cost reports** (`cldy-mcp-local`): Use `run_cost_report` with `container_cluster_name` and `container_namespace` dimensions for billing-reconciled costs. The "IDLE RESOURCES" namespace represents idle cost.

**Rule of thumb**:
- Use `get_kubecost_workload_costs` for workload attribution, team chargeback, efficiency analysis, pod/label-level breakdowns
- Use `run_cost_report` with container dimensions for billing reconciliation, idle analysis, cluster-level totals

## Cost Report Quick Reference

### Common Dimensions
`vendor`, `enhanced_service_name`, `region`, `vendor_account_name`, `container_cluster_name`, `container_namespace`, `date`, `year_week`, `year_month`, `instance_type`, `lease_type`

### Common Metrics
`unblended_cost`, `total_amortized_cost`, `public_on_demand_cost`, `usage_quantity`, `usage_hours`

### Filter Syntax
`filters: ["dimension_name==value"]` — operators: `==`, `!=`, `>`, `<`, `>=`, `<=`

### Key View
- Product Hierarchy view ID: `1706692` (used by executive dashboard)

## Workflow Patterns

### End-of-Session Checklist
After any session that modifies the Cloudability system:
1. **Update ARCHITECTURE.md** — Add new decisions (D00X format) with date, rationale, tradeoffs
2. **Update SKILL.md** — If new tools, environments, projects, or workflows were added
3. **Update env-keys doc** — If tokens or environment IDs changed (`~/cloudability-env-keys.md`) — NEVER push this file
4. **Push to GitHub** — All skills to `Yeazey/cloudability-kiro-skills`, dashboard projects to their respective repos
5. **Commit skill changes** — If SKILL.md or ARCHITECTURE.md changed, note it for the user

### Standard Workflows
1. **Quick cost check**: Use `run_cost_report` (cldy-mcp-local) with dimensions/metrics
2. **Container analysis**: Use `get_kubecost_workload_costs` (cldy-mcp-local) for fairshare/workload costs, or `run_cost_report` with `container_cluster_name`/`container_namespace` for billing costs
3. **Dashboard generation**: Each skill runs its own generator → produces HTML → opens in browser (these projects use Node.js MCP internally)
4. **Anomaly investigation**: `list_anomalies` (cldy-mcp-local) with date range and optional filters
5. **Rightsizing**: `get_rightsizing_recommendations` (cldy-mcp-local) for multi-vendor recommendations
6. **RI analysis**: `get_ri_portfolio_summary`, `list_ri_recommendations`, `get_ri_utilization` (cldy-mcp-local)
7. **Data freshness**: `get_data_freshness_summary`, `get_pipeline_failures` (cldy-mcp-local)
8. **New dashboard/tool**: Build it → create a skill → push to GitHub → update this doc
