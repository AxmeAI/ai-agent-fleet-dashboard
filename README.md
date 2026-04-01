# AI Agent Fleet Dashboard

You have 50 agents running across three clouds, four frameworks, and a dozen services. Can you tell me which ones are healthy right now? Which one burned $200 overnight? Which one is stuck in a retry loop?

You can't. Because there's no single place to see them all.

**AXME gives you a real-time fleet dashboard at [mesh.axme.ai](https://mesh.axme.ai).**

## The Problem

AI agents are everywhere now. Your org runs them on different clouds (AWS, GCP, Azure), built with different frameworks (LangGraph, CrewAI, AutoGen, OpenAI Agents SDK), deployed as different services (Cloud Run, Lambda, ECS, bare VMs).

Each one has its own logs. Its own metrics. Its own way of telling you it's alive -- or not.

What you actually need:

- **One screen** showing every agent, every cloud, every framework
- **Health status** -- which agents are running, which are stuck, which crashed 10 minutes ago
- **Cost tracking** -- per-agent LLM spend, token counts, cost trends
- **Policy enforcement** -- rate limits, spending caps, kill switches
- **Search and filter** -- find agents by name, team, status, framework, cloud

What you have today: 12 browser tabs, 4 CLI tools, and a spreadsheet someone updates manually on Fridays.

## The Solution

![Agent Mesh Dashboard](mesh-dashboard.png)

AXME Agent Mesh gives you a unified dashboard for your entire agent fleet. Every agent registers with a heartbeat. The dashboard shows them all in real time.

```
+---------------------------------------------------------+
|  AXME Agent Mesh Dashboard         mesh.axme.ai         |
+---------------------------------------------------------+
| Agents: 52 total | 48 healthy | 3 degraded | 1 dead    |
| Cost today: $142.30 | MTD: $3,847.12                   |
+---------------------------------------------------------+
| Name              | Status  | Cloud | Cost/hr | Uptime |
|-------------------|---------|-------|---------|--------|
| data-pipeline-01  | healthy | GCP   | $2.40   | 14d    |
| support-bot-prod  | healthy | AWS   | $8.10   | 7d     |
| code-reviewer     | degraded| GCP   | $0.90   | 2d     |
| invoice-processor | dead    | Azure | $0.00   | 0m     |
| research-agent-03 | healthy | GCP   | $1.20   | 5d     |
| ...               |         |       |         |        |
+---------------------------------------------------------+
| [Search] [Filter: status] [Filter: cloud] [Kill Agent] |
+---------------------------------------------------------+
```

### What the dashboard gives you

- **Agents table** -- all registered agents with name, status, cloud, framework, team, cost, uptime
- **Real-time health** -- heartbeat-based status (healthy / degraded / dead), updated every 30 seconds
- **Filters and search** -- filter by status, cloud, framework, team; full-text search by agent name
- **Cost breakdown** -- per-agent LLM spend (tokens in/out, model, cost), hourly and daily aggregation
- **Kill switch** -- select an agent, click Kill, it receives a shutdown intent via AXME
- **Policy view** -- rate limits, spending caps, and escalation rules per agent or team

## Quick Start

### 1. Install the SDK and CLI

```bash
pip install axme
# or: npm install @axme/sdk
```

### 2. Register your agent with a heartbeat

```python
from axme import AxmeClient, AxmeClientConfig

client = AxmeClient(AxmeClientConfig(api_key=os.environ["AXME_API_KEY"]))

# Register this agent in the mesh
client.register_agent({
    "agent_id": "data-pipeline-01",
    "agent_type": "data_processor",
    "framework": "langgraph",
    "cloud": "gcp",
    "team": "data-eng",
    "metadata": {
        "region": "us-central1",
        "model": "claude-sonnet-4-20250514",
    },
})

# Start heartbeat (reports health + cost every 30s)
client.start_heartbeat(interval_seconds=30)

# Your agent does its work...
for task in task_queue:
    result = process(task)
    # Cost is tracked automatically via SDK instrumentation
```

```typescript
// TypeScript
import { AxmeClient } from "@axme/sdk";

const client = new AxmeClient({ apiKey: process.env.AXME_API_KEY });

await client.registerAgent({
  agentId: "support-bot-prod",
  agentType: "customer_support",
  framework: "openai-agents",
  cloud: "aws",
  team: "support",
});

await client.startHeartbeat({ intervalSeconds: 30 });
```

### 3. Open the dashboard

```bash
# Open the fleet dashboard in your browser
axme mesh dashboard

# Or go directly to:
# https://mesh.axme.ai
```

### 4. CLI fleet commands

```bash
# List all agents in your mesh
axme mesh agents

# Filter by status
axme mesh agents --status degraded

# Filter by cloud
axme mesh agents --cloud gcp

# Kill a stuck agent
axme mesh kill data-pipeline-01

# Show cost summary
axme mesh cost --period today

# Show cost by team
axme mesh cost --group-by team --period mtd
```

## How It Works

1. **Registration** -- each agent calls `register_agent()` on startup with its identity and metadata
2. **Heartbeat** -- the SDK sends a heartbeat every N seconds with health status and cost metrics
3. **Dashboard** -- the web UI at mesh.axme.ai reads the mesh state and renders the fleet view
4. **Kill signal** -- when you click Kill (or run `axme mesh kill`), AXME sends a shutdown intent to the agent via the standard intent delivery mechanism
5. **Policy enforcement** -- spending caps and rate limits are checked on every heartbeat; violations trigger alerts or automatic throttling

## Agent Lifecycle States

| State | Meaning | Heartbeat |
|-------|---------|-----------|
| `registering` | Agent called register, first heartbeat pending | -- |
| `healthy` | Heartbeat received within expected interval | On time |
| `degraded` | Heartbeat late (1-3x interval) or error reported | Late |
| `dead` | No heartbeat for 3+ intervals | Missing |
| `killed` | Shutdown intent sent and acknowledged | Stopped |

## Works With Any Framework

The dashboard doesn't care what framework your agents use. If they can call the AXME SDK, they appear in the dashboard.

| Framework | Registration | Heartbeat | Kill |
|-----------|-------------|-----------|------|
| LangGraph | Yes | Yes | Yes |
| CrewAI | Yes | Yes | Yes |
| AutoGen | Yes | Yes | Yes |
| OpenAI Agents SDK | Yes | Yes | Yes |
| Google ADK | Yes | Yes | Yes |
| Pydantic AI | Yes | Yes | Yes |
| Raw Python/TS/Go | Yes | Yes | Yes |

## Links

- [AXME](https://github.com/AxmeAI/axme) -- agent coordination infrastructure
- [AXME Docs](https://docs.axme.ai) -- full documentation
- [AXME Cloud](https://cloud.axme.ai) -- managed service
- [Agent Mesh Dashboard](https://mesh.axme.ai) -- live fleet dashboard

## License

MIT
