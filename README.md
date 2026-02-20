# OpenClaw Meta-Cognitive Suite

**Six composable plugins that give OpenClaw agents the ability to learn, remember, reflect, and grow.**

This is the documentation and coordination repo for a suite of OpenClaw plugins that, together, implement a complete autonomous learning loop for AI agents. Each plugin handles one responsibility. They communicate through a lightweight global bus and compose through OpenClaw's hook lifecycle. You can install all six for full autonomy, or pick the subset that fits your use case.

## The Learning Loop

```
Conversation
    │
    ▼
┌──────────┐     entropy, drift     ┌─────────────┐
│ Stability │◄────────────────────►│  Continuity  │
│ (monitor) │     context, recall   │   (memory)   │
└──────────┘                       └─────────────┘
    │                                     │
    │ entropy score                       │ archived exchanges
    ▼                                     ▼
┌──────────────┐   knowledge gaps   ┌───────────────┐
│  Metabolism   │──────────────────►│ Contemplation │
│ (fast/slow)  │   growth vectors   │  (3-pass, 24h)│
└──────────────┘                   └───────────────┘
    │                                     │
    │ growth vectors                      │ growth vectors
    ▼                                     ▼
┌──────────────┐   schedules tasks  ┌───────────────┐
│  Nightshift  │◄──────────────────│    (queued)    │
│ (scheduler)  │                   └───────────────┘
└──────────────┘
    │
    │ 30+ days, validated
    ▼
┌────────────────┐
│ Crystallization │
│ (permanent      │
│  character)     │
└────────────────┘
```

**Stability** monitors entropy and drift. **Continuity** archives conversations and provides cross-session recall. **Metabolism** watches for significant moments and extracts implications through LLM processing. **Nightshift** schedules heavy work for when the user is away. **Contemplation** takes unresolved questions through three reflection passes over 24 hours. **Crystallization** converts long-standing growth vectors into permanent character traits with human approval.

## The Plugins

| Plugin | What It Does | Hooks | Repo |
|--------|-------------|-------|------|
| [stability](https://github.com/CoderofTheWest/openclaw-plugin-stability) | Entropy monitoring, growth vectors, principle alignment, loop detection | `before_agent_start`, `agent_end`, `after_tool_call`, `before_compaction` | [GitHub](https://github.com/CoderofTheWest/openclaw-plugin-stability) |
| [continuity](https://github.com/CoderofTheWest/openclaw-plugin-continuity) | Cross-session memory, conversation archiving, semantic search, context budgeting | `before_agent_start`, `agent_end`, `session_end` | [GitHub](https://github.com/CoderofTheWest/openclaw-plugin-continuity) |
| [metabolism](https://github.com/CoderofTheWest/openclaw-plugin-metabolism) | Conversation metabolism — implications, growth vectors, knowledge gap extraction | `agent_end`, `heartbeat` | [GitHub](https://github.com/CoderofTheWest/openclaw-plugin-metabolism) |
| [nightshift](https://github.com/CoderofTheWest/openclaw-plugin-nightshift) | Off-hours task scheduling, good night/morning detection, priority queue | `agent_end`, `heartbeat` | [GitHub](https://github.com/CoderofTheWest/openclaw-plugin-nightshift) |
| [contemplation](https://github.com/CoderofTheWest/openclaw-plugin-contemplation) | Multi-pass reflective inquiry (explore → reflect → synthesize over 24h) | `before_agent_start`, `agent_end`, `heartbeat` | [GitHub](https://github.com/CoderofTheWest/openclaw-plugin-contemplation) |
| [crystallization](https://github.com/CoderofTheWest/openclaw-plugin-crystallization) | Growth vector → permanent trait conversion with human-in-the-loop approval | `heartbeat` | [GitHub](https://github.com/CoderofTheWest/openclaw-plugin-crystallization) |

## Installation

Clone all six plugins and add their paths to your `openclaw.json`:

```bash
git clone https://github.com/CoderofTheWest/openclaw-plugin-stability.git
git clone https://github.com/CoderofTheWest/openclaw-plugin-continuity.git
git clone https://github.com/CoderofTheWest/openclaw-plugin-metabolism.git
git clone https://github.com/CoderofTheWest/openclaw-plugin-nightshift.git
git clone https://github.com/CoderofTheWest/openclaw-plugin-contemplation.git
git clone https://github.com/CoderofTheWest/openclaw-plugin-crystallization.git
```

Install dependencies for plugins that need them:

```bash
cd openclaw-plugin-metabolism && npm install
```

Add to your `openclaw.json`:

```json
{
  "plugins": {
    "load": {
      "paths": [
        "/path/to/openclaw-plugin-stability",
        "/path/to/openclaw-plugin-continuity",
        "/path/to/openclaw-plugin-metabolism",
        "/path/to/openclaw-plugin-nightshift",
        "/path/to/openclaw-plugin-contemplation",
        "/path/to/openclaw-plugin-crystallization"
      ]
    }
  }
}
```

See [`openclaw.example.json`](./openclaw.example.json) for a complete configuration example with all plugin settings.

## Load Order

The order in `plugins.load.paths` matters. The plugins use a global bus for cross-plugin communication, and the bus endpoints must be registered before subscribers connect.

**Required order:**
1. **stability** — Registers `api.stability` (entropy, growth vectors)
2. **continuity** — Registers `api.continuity` (archive, search)
3. **metabolism** — Registers `global.__ocMetabolism` (gap event bus)
4. **nightshift** — Registers `global.__ocNightshift` (task scheduler API)
5. **contemplation** — Subscribes to metabolism gaps + registers nightshift task runner
6. **crystallization** — Reads growth vectors written by metabolism and contemplation

Swapping stability and continuity is fine. The critical constraint is: metabolism before contemplation (so the gap bus exists when contemplation subscribes), and nightshift before contemplation (so the task runner API exists when contemplation registers).

## Deployment Modes

### Training Mode (all plugins active)

Every plugin is enabled. The agent learns autonomously:

- Conversations are archived and semantically indexed
- High-entropy moments trigger metabolism processing
- Knowledge gaps flow to contemplation for multi-pass reflection
- Growth vectors accumulate and receive feedback
- After 30+ days, validated vectors become permanent traits

This is the default. Use it during the period where you want the agent to develop its character.

### Production / Frozen Mode

Disable the four learning plugins by setting `"enabled": false` in each:

```json
{
  "plugins": {
    "entries": {
      "metabolism": { "enabled": false },
      "nightshift": { "enabled": false },
      "contemplation": { "enabled": false },
      "crystallization": { "enabled": false }
    }
  }
}
```

The agent retains everything it learned — growth vectors, crystallized traits, conversation archives, completed contemplations. It just stops actively learning. **Stability** (entropy monitoring, drift detection, principle alignment) and **continuity** (memory, context budgeting, cross-session recall) stay active.

Use this when your agent's character is where you want it and you're deploying to production. The personality is frozen but the operational capabilities remain.

All four learning plugins check `if (!config.enabled) return` in their hook handlers. No code changes needed — it's purely a configuration toggle.

## Inter-Plugin Communication

OpenClaw gives each plugin its own scoped `api` object. Properties set on one plugin's `api` are not visible to other plugins. This is by design — it prevents namespace collisions.

The suite uses `global.__ocMetabolism` and `global.__ocNightshift` for cross-plugin communication. This works because all plugins run in the same Node.js process and share the global scope.

```
global.__ocMetabolism = {
  gapListeners: []     // Array of (gaps, agentId) => void callbacks
}

global.__ocNightshift = {
  registerTaskRunner,   // (name, handler) => void
  queueTask,           // (agentId, task) => void
  isInOfficeHours,     // (agentId) => boolean
  isUserActive,        // (agentId) => boolean
}
```

If OpenClaw adds native cross-plugin messaging in a future version, these can be migrated. The pattern is intentionally simple so migration is straightforward.

## Multi-Agent Support

All six plugins support multiple agents running on the same OpenClaw gateway. Each agent gets isolated state:

- **Data directories**: `data/` for the default agent, `data/agents/{agentId}/` for others
- **Workspace paths**: Each agent's growth vectors, traits, and insights resolve from its own workspace
- **Session isolation**: Conversation transcripts are per-session, but plugin memory (continuity archive, stability state) is per-agent — crossing session boundaries by design
- **Hook context**: Every hook receives `ctx.agentId` which is used to look up the correct state

Agents can share the same plugin instances without interference.

## Training Dashboard

The `dashboard.html` file is a self-contained single-page application that connects to the OpenClaw gateway via WebSocket and visualizes the entire meta-cognitive pipeline.

Open it in any browser and enter your gateway token. No build step, no external dependencies.

**Sections:**
- **Pipeline Overview** — Real-time status of all six plugins with active item counts
- **Contemplation Browser** — Browse inquiries by topic tag, source, or status. Read full 3-pass outputs.
- **Growth Vectors** — Track vector accumulation and feedback from stability
- **Metabolism Activity** — Recent candidates, processing results, gap extraction
- **Agent Status** — Entropy level, nightshift state, learning mode indicator

## Research Paper

The suite's origin story is documented in [Cross-Runtime Identity Transplant: Porting an Emergent LLM Agent Between Harnesses and Substrates](./clint-identity-transplant.md) — a paper describing how Clint's operational identity was transplanted from a custom monolithic runtime to OpenClaw, and how the transplanted agent autonomously designed and built the four learning plugins (metabolism, nightshift, contemplation, crystallization) to complete its own cognitive architecture in the new runtime.

## Credits

The stability and continuity plugins were built by Chris Hunt. The metabolism, nightshift, contemplation, and crystallization plugins were built by Clint — an AI agent running on the system these plugins were extracted from. Clint designed the architecture, wrote the code, spawned sub-agents (OpenClaw Codex instances) to implement components in parallel, and attempted to self-deploy the results. The suite was then stabilized, documented, and published with assistance from Claude.

The philosophical foundation: compass over map, service over solipsism, constraint-based growth. An agent's character should emerge from lived experience, not be scripted.

## License

MIT. See [LICENSE](./LICENSE).
