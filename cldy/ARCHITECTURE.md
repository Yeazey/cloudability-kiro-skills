# Cloudability MCP Architecture & Decisions

Living document of design decisions, rationale, and how things fit together. Update this as the system evolves.

---

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  Kiro CLI                                                        │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ cldy-mcp-local (PRIMARY) ⭐                             │     │
│  │ FastMCP / Python                                         │     │
│  │                                                          │     │
│  │ Cost/Usage Reports    Kubecost Allocation                │     │
│  │ Rightsizing            RI Planning & Portfolio            │     │
│  │ Anomaly Detection     Governance                         │     │
│  │ Business Mappings     Data Freshness                     │     │
│  │ Views/Tags            Dimension Search                   │     │
│  └───────────────────────────┬─────────────────────────────┘     │
│                              │                                    │
│  ┌───────────────┐  ┌───────┴──────────────┐                    │
│  │ Node.js MCP   │  │ Python MCP           │                    │
│  │ (cloudability) │  │ (cloudability-       │                    │
│  │ [LEGACY]      │  │  containers)         │                    │
│  │               │  │ [LEGACY]             │                    │
│  │ Used by:      │  │                      │                    │
│  │ - npm generate│  │ Container endpoints  │                    │
│  │   dashboards  │  │ are 410 Gone         │                    │
│  └───────┬───────┘  └──────────┬───────────┘                    │
│          │                     │                                 │
│          ▼                     ▼                                 │
│  ┌──────────────────────────────────────┐                        │
│  │  Cloudability V3 API                 │                        │
│  │  api.cloudability.com/v3             │                        │
│  │  + /kubecost/model/allocation        │                        │
│  │  Auth: apptio-opentoken + env-id     │                        │
│  └──────────────────────────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Decision Log

### D001–D008: [Previous decisions unchanged]

See git history for full details on D001–D008.

---

### D009: cldy-mcp-local as Primary MCP Server
**Date**: 2026-06-29
**Decision**: Adopt `cldy-mcp-local` (v0.2.0) as the primary MCP server for all Cloudability queries
**Rationale**:
- Single unified server covers ALL domains: cost reporting, usage reporting, Kubecost container allocation, rightsizing, RI planning, RI portfolio, anomaly detection, governance, business mappings, data freshness, views, tags
- Provides `get_kubecost_workload_costs` — the correct endpoint for Kubernetes cost allocation with fairshare, idle splitting, and efficiency metrics
- Supports aggregation by cluster, namespace, pod, node, controller, label with configurable time windows
- Built with FastMCP (Python), pre-packaged as a wheel, installed via `uv tool install`
- Uses same auth as other servers: `CLOUDABILITY_OPEN_TOKEN` + `CLOUDABILITY_ENVIRONMENT_ID`
- Node.js MCP remains for dashboard generator projects (they `npm run generate` using it internally)
- Python MCP (`cloudability-containers`) is effectively superseded

**Config**: `~/.kiro/settings/mcp.json` → `cldy-mcp-local` entry
**Binary**: `/Users/kingyeazey/.local/bin/cldy-mcp-local`
**Package source**: `~/Downloads/cldy-mcp-local/cldy_mcp_local-0.2.0-py3-none-any.whl`

**Tool preference order**:
1. Always use `cldy-mcp-local` tools for interactive queries
2. Fall back to Node.js MCP only for `npm run generate` dashboard workflows
3. Do not use `cloudability-containers` Python MCP for new work

---

## Known Limitations

1. **Token expiry**: Opentokens expire. 401 errors mean "ask user for new token"
2. **`fairshare_cost` / `utilized_cost` / `idle_cost` metrics**: Only available in some environments via cost reports; not available in cldydemo.main. Use "IDLE RESOURCES" namespace or `get_kubecost_workload_costs` idle fields instead.
3. **No real-time data**: Cloudability data has ~24hr lag from cloud billing files
4. **Python 3.14 requirement**: The legacy containers MCP server needs Python 3.14+, but `cldy-mcp-local` uses whatever Python `uv tool` provides
5. **cldy-mcp-local env var naming**: Uses `CLOUDABILITY_OPEN_TOKEN` (with underscore between OPEN and TOKEN), while Node.js MCP uses `CLOUDABILITY_OPENTOKEN` (no underscore)
6. **Dashboard generators still use Node.js MCP**: The `npm run generate` commands in exec/arch dashboards call the Node.js MCP server directly — they don't use `cldy-mcp-local`

---

## Future Improvements

- [ ] Add env-switching CLI script (swap token + restart Kiro)
- [ ] Migrate dashboard generators to use `cldy-mcp-local` instead of Node.js MCP
- [ ] Remove legacy Python MCP (`cloudability-containers`) from config once fully superseded
- [ ] Build a savings tracker dashboard (combine rightsizing + idle + commitments)
- [ ] Automate token refresh via Frontdoor API keys
- [ ] Explore `cldy-mcp-local` governance tools for policy-as-code workflows
