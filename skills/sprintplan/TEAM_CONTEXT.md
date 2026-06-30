# FinOps Team Context

> Fill out this file to customize sprint planning for your team.
> The sprint planning skill reads this to generate contextual, relevant sprint plans.

---

## Team Structure

```yaml
team_name: "Cloud FinOps"
team_size: 3
model: "hub-and-spoke"  # centralized | distributed | hub-and-spoke
maturity: "walk"         # crawl | walk | run
reports_to: "CTO"       # CTO | CIO | CFO | VP Engineering
```

## Sprint Configuration

```yaml
sprint_length: "monthly"      # biweekly | monthly
planning_day: "first_monday"  # first_monday | first_tuesday | last_friday_prior
sprint_goal_style: "outcome"  # outcome ($ saved) | throughput (actions completed) | hybrid
velocity_target: 4            # optimization actions per sprint (Foundation recommends >= 4)
```

## Cloud Scope

```yaml
vendors:
  - aws
  - azure
  - oci
  - gcp

saas_in_scope: true  # Include Snowflake, Databricks, MongoDB in planning?
containers_in_scope: true  # Include Kubernetes/container optimization?
ai_in_scope: true  # Include AI/ML workload planning (Bedrock, OpenAI, Foundry Models)?
```

## Key Stakeholders & Teams

```yaml
stakeholders:
  - name: "Platform Engineering"
    contact: ""  # Slack handle or name
    owns: "Infrastructure, Kubernetes, CI/CD"
    meeting_cadence: "weekly"

  - name: "SRE"
    contact: ""
    owns: "Production reliability, autoscaling, incident response"
    meeting_cadence: "weekly"

  - name: "Data Science"
    contact: ""
    owns: "ML workloads, GPU instances, training jobs"
    meeting_cadence: "biweekly"

  - name: "Finance / FP&A"
    contact: ""
    owns: "Budgets, forecasts, chargeback, executive reporting"
    meeting_cadence: "monthly"

  - name: "Engineering Leadership"
    contact: ""
    owns: "Architecture decisions, team priorities, resource allocation"
    meeting_cadence: "monthly"

  - name: "Procurement"
    contact: ""
    owns: "Contracts, EDPs, vendor negotiations, SaaS renewals"
    meeting_cadence: "quarterly"
```

## Priority Weights

```yaml
priority_weights:
  cost_of_inaction: 10
  commitment_risk: 9
  budget_breach: 8
  anomaly_severity: 7
  governance_gap: 6
  recommendation_staleness: 5
  team_engagement_decay: 4
  new_service_adoption: 3
```

## Work Categories & Allocation

```yaml
sprint_allocation:
  optimization_usage: 25
  optimization_rate: 15
  governance: 20
  visibility: 15
  enablement: 10
  tooling: 10
  forecasting: 5
```

## KPI Targets

```yaml
kpi_targets:
  waste_rate_pct: 5
  commitment_coverage_pct: 70
  commitment_utilization_pct: 90
  tag_compliance_pct: 90
  forecast_variance_pct: 10
  recommendation_action_rate: 50
  anomaly_response_hours: 4
```

## Communication Preferences

```yaml
communication:
  primary_channel: "slack"
  slack_channel: ""
  win_of_week_channel: ""
  escalation_path: ""
  report_format: "markdown_in_chat"
  export_to_jira: false
```

## Planning Horizons

```yaml
horizons:
  immediate: "7d"
  sprint: "30d"
  upcoming: "90d"
  strategic: "180d"
```

## Custom Recurring Items

```yaml
recurring_items:
  - title: "Monthly cost allocation report"
    cadence: "monthly"
    owner: "FinOps Lead"
    week: 1

  - title: "Quarterly business review prep"
    cadence: "quarterly"
    owner: "FinOps Lead"
    week: 3

  - title: "Weekly anomaly triage"
    cadence: "weekly"
    owner: "FinOps Team"
    day: "monday"
```

## Notes

```
- Fill in team-specific context here
- Example: "We're mid-migration from AWS to Azure"
- Example: "Databricks contract renews in September"
```
