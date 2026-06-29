---
name: containersdash
description: Generate and open the Kubernetes Container Cost Dashboard. Use when the user asks for container dashboard, k8s costs, kubernetes spend, cluster costs, container optimization, idle resources, or namespace breakdown.
---

# Kubernetes Container Cost Dashboard

When triggered, pull live container cost data from Cloudability using `cldy-mcp-local` and generate an interactive HTML dashboard.

## Primary Data Source: `cldy-mcp-local`

Use these tools for container cost data:

### 1. Kubecost Workload Costs (preferred for allocation data)
- **Tool**: `get_kubecost_workload_costs`
- **Aggregate options**: `cluster`, `namespace`, `cluster,namespace`, `pod`, `node`, `controller`, `label`
- **Window options**: `7d`, `30d`, `90d`, `month`, `lastmonth`, `lastweek`, or RFC3339 range
- **Returns**: Fairshare-allocated costs with idle splitting, efficiency metrics, CPU/RAM/GPU/PV/network cost breakdown
- **Important**: Call `kubecost_list_windows` first if user hasn't specified a time window

### 2. Cost Reports (for billing-reconciled data)
- **Tool**: `run_cost_report`
- **Dimensions**: `container_cluster_name`, `container_namespace`, `vendor`, `region`, `year_week`
- **Metrics**: `unblended_cost`, `total_amortized_cost`
- **Use for**: Billing totals, idle analysis via "IDLE RESOURCES" namespace, vendor/region breakdown

## Steps

1. Pull container data using `cldy-mcp-local`:
   - **Cluster summary**: `get_kubecost_workload_costs` with `aggregate=cluster`, `window=30d`
   - **Namespace detail**: `get_kubecost_workload_costs` with `aggregate=cluster,namespace`, `window=30d`
   - **Weekly trend**: `get_kubecost_workload_costs` with `aggregate=cluster`, `window=30d`, `accumulate=false`
   - **Region/Vendor breakdown**: `run_cost_report` with dimensions=`container_cluster_name,vendor,region`

2. Build a self-contained HTML dashboard at `/Users/kingyeazey/container-dashboard.html` with Chart.js (CDN) and 4 tabs:
   - **Overview**: KPI cards (total spend, idle cost, idle %, cluster count), vendor donut, region bar chart, weekly trend line chart
   - **Top Clusters**: horizontal bar chart of top 20 clusters, detailed table with name/cost/amortized/region/vendor
   - **Optimization**: idle vs utilized stacked bar per cluster, top idle namespaces table, total potential savings callout, efficiency metrics
   - **KPIs**: key metric cards, cost distribution chart, week-over-week change indicators

3. Open the dashboard in the browser:
   ```bash
   open /Users/kingyeazey/container-dashboard.html
   ```

## Notes

- `get_kubecost_workload_costs` is the preferred tool — it provides fairshare allocation with idle splitting
- Fall back to `run_cost_report` with container dimensions for billing-reconciled totals
- Cluster naming convention reveals vendor: `*-akp-*` = AWS EKS, `*-oke-*` = OCI OKE
- GitHub repo: `Yeazey/kubernetes-container-cost-dashboard`
