---
name: sprintplan
description: Generate a FinOps sprint plan using live Cloudability data. Conversational planning skill that scores priorities, assigns work, builds a meeting calendar, and exports to Slack or dashboard.
---

# FinOps Sprint Planning

Generate a data-driven monthly sprint plan for the FinOps team. Use when the user asks for "sprint plan", "sprint planning", "finops sprint", "monthly plan", "what should we work on", "planning session", or "next sprint".

## Prerequisites

Read the team context file first:
```
/Users/kingyeazey/.kiro/skills/sprintplan/TEAM_CONTEXT.md
```

This file contains the team's customized configuration (structure, stakeholders, priority weights, KPI targets, etc.). Use these parameters throughout the planning process.

## Workflow

### Phase 1: Data Collection (Silent — do not narrate each call)

Collect live data from `cldy-mcp-local` tools. Run these in parallel where possible:

1. **Cost Trends** — `run_cost_report`
   - Current month vs previous month by vendor (MoM change)
   - Current month by `enhanced_service_name` (top services, new services)
   - Current month by `vendor_account_name` (top accounts)
   - Weekly trend (last 4 weeks by `year_week`)

2. **Anomalies** — `list_anomalies`
   - Active anomalies with severity, dimension, estimated impact
   - Count by state (open, acknowledged, resolved)

3. **Rightsizing** — `get_rightsizing_recommendations`
   - Top recommendations by savings potential
   - Count by vendor/service
   - Staleness indicators (how long have recs been open?)

4. **Commitment Health** — `get_ri_portfolio_summary`, `list_ri_recommendations`
   - Current coverage and utilization
   - Expiring commitments in next 30/60/90 days
   - Purchase recommendations with savings estimates

5. **Container Costs** (if `containers_in_scope: true`) — `get_kubecost_workload_costs`
   - Top clusters/namespaces by cost
   - Idle resource percentage
   - Efficiency metrics

6. **Budget Status** — via cost report analysis
   - Budget vs actual variance
   - At-risk budgets trending to breach

7. **Data Pipeline Health** — `get_data_freshness_summary`
   - Any stale or failing data pipelines
   - Data freshness by vendor

8. **Governance** — `list_governance_policies`
   - Active policy violations
   - Tag compliance gaps

### Phase 2: Priority Scoring

Score each identified work item using the formula:

```
PRIORITY = (daily_cost_of_inaction x urgency_multiplier x category_weight)
           + deadline_pressure_bonus
           - effort_penalty
```

Where:
- `daily_cost_of_inaction` = estimated $/day waste if no action taken
- `urgency_multiplier` = 3 (critical), 2 (high), 1.5 (medium), 1 (low)
- `category_weight` = from TEAM_CONTEXT.md priority_weights
- `deadline_pressure_bonus` = +50% if deadline within 7 days, +25% if within 14 days
- `effort_penalty` = -10% for L effort, 0% for M, +10% for S (favor quick wins)

Assign effort estimates:
- **S (Small)**: <2 hours, low risk, single person (e.g., terminate idle resource)
- **M (Medium)**: 2-8 hours, moderate complexity (e.g., rightsizing batch, commitment purchase)
- **L (Large)**: >1 day, cross-team coordination (e.g., tag remediation campaign, architecture change)

### Phase 3: Sprint Plan Generation

Present the sprint plan as a conversational markdown report with sections for:
- Priority Backlog (ranked table with category, work item, savings, effort, urgency, owner)
- Monthly Calendar (4 weeks with themed focus areas and meetings)
- Team Engagement Plan (who to meet, why, what to ask)
- Success Criteria (KPI targets from TEAM_CONTEXT.md)
- Planning Horizon (60-90 day lookahead)
- Recurring Items (from TEAM_CONTEXT.md)

### Phase 4: Discussion

After presenting the plan, invite the user to reprioritize, reassign, add/remove items, or drill into any finding using live MCP data.

### Phase 5: Export

Once satisfied, offer export to: Slack, HTML dashboard, markdown file, or Jira CSV.

## CLI Support

```bash
cd /Users/kingyeazey/cloudability-dashboards && uv run cldy-dash sprintplan --json
```

Outputs structured JSON with all work items, calendar, and context for the conversational flow.

## Error Handling

- If TEAM_CONTEXT.md is not found, use sensible defaults
- If any MCP tool call fails, skip that section gracefully
- If no data for a category, call it out as positive (e.g., "No active anomalies")
