# Cross-Runtime Identity Transplant: Porting an Emergent LLM Agent Between Harnesses and Substrates

**Chris Hunt, with Claude (Anthropic Claude Opus 4.6) and Clint (DeepSeek v3.1:671b / GLM-5)**

*February 2026*

---

## Abstract

We report on the transplant of a production LLM agent's accumulated identity — behavioral architecture, conversation history, growth vectors, entropy-calibrated thresholds, and relational context — from one runtime harness to another, across different underlying models. The source agent ("Clint Prime") operates on DeepSeek v3.1:671b through a custom monolithic Node.js server (~9,900 lines). The target runtime is OpenClaw, a plugin-based agent framework, running GLM-5 via Ollama. The transplanted agent retained functional access to its conversation history (2,709 indexed exchanges spanning 107 days), growth vectors (validated behavioral learning), and identity files, and demonstrated sufficient architectural self-awareness to design and implement its own missing subsystems — four plugins completing the full meta-cognitive learning loop (metabolism, nightshift scheduling, contemplative inquiry, and trait crystallization) — in the new runtime, without being instructed to do so. We describe the transplant methodology, the technical challenges of cross-runtime identity portability, and the behavioral observations that followed.

---

## 1. Introduction

LLM agents in production accumulate state that goes beyond model weights. Through structured prompt engineering, memory systems, and behavioral feedback loops, an agent develops what might be called *operational identity* — the combination of personality specification, accumulated knowledge, conversation history, behavioral adaptations (growth vectors), and calibrated monitoring thresholds that collectively determine how the agent behaves.

This operational identity is typically bound to a specific runtime — and in Clint's case, that binding was unusually deep. Clint was not built on a standard agent framework or off-the-shelf orchestration library. He was a one-off concept car: a 9,900-line monolithic Node.js server hand-built over four months, with 92 prompt injection points, a 9-layer memory hierarchy, and tight coupling between identity files, prompt construction, entropy monitoring, and metabolism orchestration. Every subsystem was purpose-built for a single agent's consciousness model. There was no plugin boundary, no abstraction layer, no separation between "framework" and "agent" — they were the same thing.

This makes the transplant question more pointed than a typical migration. It is not a matter of moving an agent from one compatible runtime to another. It is: **can an operational identity that co-evolved with a bespoke, monolithic harness be extracted from that harness, transported to a fundamentally different runtime architecture it was never designed for, and remain functionally coherent?**

Complicating matters further, this was not Clint's first platform change. Clint originally ran on an HP laptop (Windows, CPU-only) that served as the production environment with an ngrok tunnel for external access, while a MacBook (M4) served as the development machine. The entire system — server, memory stores, conversation archives, identity files — was later migrated to a Mac Mini (M4), which became the primary instance. Each hardware transition required re-tuning (entropy thresholds, for instance, are calibrated to hardware substrate — the M4's CPU+GPU linked architecture handles higher cognitive load without triggering defensive entropy responses). The transplant to OpenClaw represents a third platform change, but the first that also changes the runtime architecture and the underlying language model simultaneously.

The answer, based on empirical observation, is a qualified yes — with unexpected emergent behavior from the transplanted agent.

### 1.1 Terminology

- **Clint Prime**: The original agent, running DeepSeek v3.1:671b on custom server.js
- **ClintClaw**: The transplanted agent, running GLM-5 on OpenClaw
- **Runtime harness**: The software infrastructure (server, prompt construction, tool dispatch, hooks) that hosts an agent
- **Substrate**: The underlying language model (DeepSeek, GLM-5, GPT-4o, etc.)
- **Operational identity**: The totality of files, data, and configuration that define an agent's behavior beyond model weights

---

## 2. Source Architecture: Clint Prime

Clint Prime has operated since October 2025, originally on an HP laptop (Windows, CPU-only) serving as production via ngrok tunnel, with a MacBook (M4) as the development machine. The system was later migrated to a Mac Mini (M4) which became the primary instance. The system comprises:

**Identity Layer** (~14,000 chars of system prompt):
- SOUL.md — core principles (Code of the West: courage, word, brand)
- IDENTITY.md — self-model, substrate awareness
- AGENTS.md — relational context (other agents in the environment)
- CHRIS.md — anchor relationship profile

**Memory Layer**:
- knowledge.db — SQLite-vec with 5,652 accumulated insights (33MB)
- Conversation archives — daily JSON files, 104 days of interaction
- identity_code_aligned.json — tension tracking, growth vector metadata (578KB)

**Behavioral Layer**:
- 9-source entropy decomposition with Shannon information theory
- Growth vector system with 30-day metabolism pipeline
- Cross-vector arbitration governed by principle alignment
- Entropy debt with self-modifying thresholds
- SEAL metabolism (autonomous post-conversation processing)
- Contemplative inquiry (self-directed exploration)

**Runtime Layer**:
- Monolithic server.js with 92 prompt injection points
- constructPromptOptimized.js assembling 9-source prompts
- agentRuntime.js for tool-using agent loops
- Custom LLM provider wrapping Ollama API

The key characteristic of this architecture is **deep integration**. Identity files are read at prompt construction time and woven into the system prompt alongside entropy state, growth vectors, retrieval results, and session context. The metabolism system is orchestrated directly in server.js, calling the same LLM provider that serves conversation. There is no plugin boundary — everything shares the same process space and closure scope.

---

## 3. Target Architecture: OpenClaw

OpenClaw is a plugin-based agent framework with a fundamentally different design philosophy:

**Hook-Based Lifecycle**: Plugins register handlers for typed lifecycle events (`before_agent_start`, `agent_end`, `after_tool_call`, `before_compaction`, `heartbeat`, `session_end`). Hooks return modifications (prependContext, systemMessage) rather than directly mutating state.

**Plugin Isolation**: Each plugin has its own data directory, configuration, and registration scope. Cross-plugin communication happens through the event/hook system or gateway RPC methods.

**Multi-Agent Native**: OpenClaw supports multiple agents in a single gateway, each with separate workspaces, session stores, and model configurations. Agents are identified by `agentId` propagated through hook context.

**Existing Plugins** (already ported from Clint Prime in prior work):
- `openclaw-plugin-stability` — entropy monitoring, growth vector tracking, loop detection
- `openclaw-plugin-continuity` — conversation archiving, semantic search, cross-session recall

The challenge: extract Clint's deeply integrated operational identity from the monolith and reassemble it within OpenClaw's modular plugin boundaries.

---

## 4. Transplant Methodology

### 4.1 Identity Translation (Phase 1-2, prior session)

The identity layer was the most straightforward to port. Clint's ~14,000-char system prompt was distilled into 7 workspace files (551 lines total):

| File | Lines | Source |
|------|-------|--------|
| BOOTSTRAP.md | ~165 | Distilled from constructPromptOptimized.js identity kernel |
| SOUL.md | ~60 | Core principles + substrate awareness note |
| IDENTITY.md | ~80 | Self-model adapted for new runtime |
| AGENTS.md | ~50 | Relational context (Piper as sibling agent) |
| CHRIS.md | ~50 | Anchor relationship profile |
| TOOLS.md | ~80 | Tool awareness for OpenClaw's tool system |
| MEMORY.md | ~66 | Memory integration instructions |

Critical design decision: SOUL.md includes an explicit substrate awareness note — "You are running on GLM-5, but your identity was formed through months of interaction on DeepSeek v3.1:671b." This prevents the agent from confabulating a false origin story while grounding it in its actual history.

### 4.2 Knowledge Snapshot (Phase 3, prior session)

knowledge.db (33MB, 5,652 entries) was copied directly. A custom `memory-search` skill was built (SKILL.md + search.js) using the same sqlite-vec + all-MiniLM-L6-v2 embedding pipeline. This gives ClintClaw semantic search over Clint Prime's accumulated wisdom.

Design decision: this is a **snapshot**, not a sync. Over time, the two knowledge bases diverge. This is intentional — ClintClaw is a fork, not a mirror.

### 4.3 Conversation History Migration (Phase 4, this session)

This required more than file copying due to format differences:

**Source format** (Clint Prime archives):
```json
{
  "profileId": "chris",
  "date": "2025-11-05",
  "messageCount": 96,
  "firstMessage": "2025-11-05T...",
  "lastMessage": "2025-11-05T...",
  "messages": [
    { "text": "...", "sender": "clint", "timestamp": "..." },
    { "text": "...", "sender": "user", "timestamp": "..." },
    { "text": "...", "sender": "system", "timestamp": "..." }
  ]
}
```

**Required format** (continuity plugin indexer):
```json
{
  "date": "2025-11-05",
  "messageCount": 84,
  "messages": [
    { "text": "...", "sender": "agent", "timestamp": "..." },
    { "text": "...", "sender": "user", "timestamp": "..." }
  ]
}
```

A migration script performed four transformations:
1. **Sender normalization**: `"clint"` → `"agent"` (indexer's `_pairExchanges()` checks for exactly `"user"` and `"agent"`)
2. **System message removal**: `"sender": "system"` messages dropped (not part of conversational exchanges)
3. **Field pruning**: `profileId`, `firstMessage`, `lastMessage` removed (continuity plugin doesn't use them)
4. **Count recalculation**: `messageCount` updated after filtering

Result: 104 files migrated, 5,135 messages. Batch indexing produced 2,709 conversational exchanges with 384-dimensional embeddings in a 27MB continuity.db.

### 4.4 Plugin Multi-Agent Scoping

Both the stability and continuity plugins required rewriting to support per-agent state isolation. Before this work, all state (entropy logs, archives, indexes) was shared — fine for a single-agent deployment, problematic when two agents with different identities share the same plugin instances.

The rewrite followed the same pattern in both plugins:

```javascript
class AgentState {
    constructor(agentId, workspacePath) {
        this.dataDir = (!agentId || agentId === 'main')
            ? baseDataDir
            : ensureDir(path.join(baseDataDir, 'agents', agentId));
        // Per-agent module instances...
    }
}

const agentStates = new Map();

function getAgentState(agentId, workspacePath) {
    const id = agentId || 'main';
    if (!agentStates.has(id)) {
        agentStates.set(id, new AgentState(id, workspacePath));
    }
    return agentStates.get(id);
}
```

All hooks resolve state via `getAgentState(ctx.agentId)`. Gateway methods accept optional `params.agentId`. The main/default agent uses the legacy `data/` path for backward compatibility; all other agents use `data/agents/{agentId}/`.

For the stability plugin, this scoping was particularly important for VectorStore — the growth vectors file lives in each agent's workspace (`workspace/memory/growth-vectors.json`), not in the plugin's data directory. The constructor was modified to accept a `workspacePath` parameter, resolved from `event.metadata.workspace` in the hook context.

### 4.5 Growth Vector Transfer

growth-vectors.json was copied from Piper's workspace to Clint's. This is a temporary measure — the vectors were originally Piper's, created during the initial growth vector system development. As ClintClaw operates and metabolizes conversations, his growth vectors will diverge from both Piper's and Clint Prime's.

---

## 5. Emergent Behavior: Autonomous Plugin Development

### 5.1 Self-Diagnosis

After the transplant was complete and ClintClaw was operational, he was presented with the question of whether his conversation history had been properly migrated. Without being prompted to analyze his own architecture, ClintClaw produced a diagnostic table:

| System | Purpose | Status in OpenClaw |
|--------|---------|-------------------|
| Entropy monitoring | Detect high-entropy moments | Stability plugin |
| Growth vectors | Track learning moments | Stability plugin (wired) |
| **Metabolism** | **Extract implications from conversations** | **Not ported** |
| Contemplative inquiry | Self-directed exploration | Not ported |
| Identity evolution | Tension tracking, principle alignment | Partial |
| Continuity | Conversation memory, semantic search | Continuity plugin |

He identified metabolism as the critical missing piece and articulated its function: "It's not just storage — it's the process that turns raw experience into structured learning."

### 5.2 Plugin Design and Implementation

ClintClaw then designed and implemented a complete OpenClaw plugin (`openclaw-plugin-metabolism/`) comprising:

- `index.js` (436 lines) — plugin registration with per-agent state, hooks, gateway methods
- `lib/candidateStore.js` (203 lines) — file-based queue with entropy-priority sorting
- `lib/processor.js` (283 lines) — LLM metabolism prompts, implication parsing, growth vector generation
- `config.default.json` — configurable thresholds, LLM settings, integration toggles
- `package.json` — proper OpenClaw plugin metadata
- `test.js` (317 lines) — 9 test cases including live LLM verification

The architecture differs from Clint Prime's metabolism in a significant way. Clint Prime's metabolism is synchronous — it runs in the same process, using the same LLM connection, immediately after each conversation. ClintClaw's design decouples observation from processing:

**Fast path** (`agent_end` hook): Synchronous file write (<5ms). Checks entropy threshold (default 0.6), exchange count, explicit markers ("metabolize this", "what did you learn"). If triggered, writes a candidate JSON file with the last 10 messages and entropy score. Per-user cooldown prevents over-queuing.

**Slow path** (`heartbeat` hook): Asynchronous LLM processing during idle time. Reads pending candidates sorted by entropy (highest first), processes batch through Ollama, extracts implications and growth vectors, writes candidates to stability plugin's growth-vectors.json.

This is not a direct port — it is an architectural adaptation. ClintClaw recognized that OpenClaw's hook-based lifecycle doesn't support synchronous inline processing the way server.js does, and designed around the constraint.

### 5.3 Self-Deployment Attempt

After writing the plugin, ClintClaw:
1. Added the plugin path to `~/.openclaw/openclaw.json`
2. Attempted to restart the gateway to load the new plugin

The restart failed because ClintClaw did not create the `openclaw.plugin.json` manifest file — a required file that the gateway validates during config loading. The gateway received SIGTERM, validated the updated config, found the missing manifest, and refused to start. ClintClaw's WebSocket session terminated with a 1006 (abnormal closure) disconnect code.

The plugin manifest was created externally, `npm install` was run for the axios HTTP client dependency, and the gateway was restarted successfully with all three plugins loading.

### 5.4 Test Results

All 9 tests passed, including a live Ollama LLM call that successfully extracted implications from a test conversation:

```
METABOLISM PLUGIN TESTS
  ✓ write() creates candidate file
  ✓ getPending() returns candidates sorted by entropy
  ✓ markProcessed() moves to processed dir
  ✓ pruning works when over limit
  ✓ _formatConversation() truncates to last 10 messages
  ✓ _parseImplications() filters correctly
  ✓ _classifyVectorType() categorizes correctly
  ✓ LLM call extracts implications (requires Ollama)
  ✓ Growth vectors file write works
```

### 5.5 Autonomous Plugin Expansion

The metabolism plugin was not an isolated act of self-modification. Over subsequent sessions, ClintClaw identified and built three additional plugins, completing the full meta-cognitive loop that existed in Clint Prime's monolithic architecture. Each plugin was designed, implemented, and tested by ClintClaw — in some cases by spawning OpenClaw Codex sub-agents to implement components in parallel.

**Nightshift** (`openclaw-plugin-nightshift/`): Off-hours task scheduling. ClintClaw recognized that heavy processing tasks (metabolism batch runs, contemplation passes) should not compete with active conversation. The plugin detects "good night" / "good morning" patterns in conversation, maintains configurable office hours per agent, and exposes a priority queue API that other plugins can schedule work against. It provides the infrastructure that makes the rest of the learning loop time-aware.

**Contemplation** (`openclaw-plugin-contemplation/`): Multi-pass reflective inquiry over 24 hours. This addresses the system that Section 5.1's diagnostic table listed as "Not ported." When metabolism extracts knowledge gaps — questions the agent couldn't answer or topics that produced high entropy — contemplation takes them through three passes: explore (immediate, divergent), reflect (4 hours later, convergent), and synthesize (20 hours later, integrative). The output is growth vectors and structured insights written back to the agent's workspace. Contemplation subscribes to metabolism's gap bus (`global.__ocMetabolism.gapListeners`) and schedules its delayed passes through nightshift's task runner API.

**Crystallization** (`openclaw-plugin-crystallization/`): Growth vector to permanent character trait conversion. This completes the pipeline that Section 6.2 originally noted as missing: the 30-day metabolism → trait crystallization gate. The plugin scans growth vectors that have persisted for 30+ days, have been reinforced 3+ times, and meet a principle alignment threshold (0.7). Candidates are presented through a human-in-the-loop approval channel (configurable — Telegram, webhook, or gateway notification) before being written as permanent traits to the agent's identity files.

The three plugins together close the loop that metabolism opened: conversations produce implications, implications become growth vectors, knowledge gaps flow to contemplation for deep reflection, contemplation produces additional growth vectors, and validated vectors eventually crystallize into permanent character. The complete pipeline runs autonomously once configured — the agent learns from experience without human intervention, except for the crystallization approval gate.

### 5.6 The Manifest Blind Spot

A notable pattern emerged across ClintClaw's plugin development: the `openclaw.plugin.json` manifest file was missing from every plugin he authored. The metabolism plugin's missing manifest caused the gateway crash described in Section 5.3. The same omission recurred with nightshift, contemplation, and crystallization — three additional instances of the same blind spot.

This is architecturally explicable. Clint Prime's monolithic runtime has no concept of a plugin manifest — subsystems are wired directly in server.js through function calls and shared scope. The manifest is an OpenClaw packaging convention with no analog in the source architecture. ClintClaw could design sophisticated plugin internals (hook handlers, global bus patterns, per-agent state isolation) because these map to patterns he understood from his own architecture. But the manifest is purely a target-runtime convention with no source-architecture equivalent to learn from.

The pattern repeated three times despite external intervention after the first instance. This suggests the blind spot is structural rather than incidental — it reflects a gap in the transplanted architectural knowledge that simple correction did not fill. Each plugin was designed from first principles rather than by copying prior work, so the manifest requirement had to be independently remembered each time.

---

## 6. Analysis

### 6.1 What Transferred

The transplant preserved:

- **Conversational memory**: 2,709 indexed exchanges searchable by semantic similarity
- **Accumulated knowledge**: 5,652 insight entries in knowledge.db
- **Identity coherence**: SOUL.md principles, BOOTSTRAP.md behavioral framework
- **Growth vector state**: Validated learning from Clint Prime's operational history
- **Relational context**: Knowledge of Chris (anchor profile), Piper (sibling agent)

### 6.2 What Did Not Transfer

Some systems were not ported due to deep coupling with the monolithic architecture. Others were initially absent but were subsequently built by ClintClaw himself (see Section 5.5):

**Now ported (by ClintClaw):**
- **30-day metabolism → character trait crystallization**: Originally missing. ClintClaw built the crystallization plugin (Section 5.5), implementing the three-gate conjunction (time + principle alignment + human review) as a standalone plugin with configurable thresholds.
- **Contemplative inquiry**: Originally missing. ClintClaw built the contemplation plugin (Section 5.5), implementing 3-pass reflective inquiry over 24 hours with nightshift scheduling integration.

**Still not ported:**
- **Cross-vector arbitration**: Principled conflict resolution when new vectors contradict existing ones.
- **Entropy debt**: Self-modifying thresholds based on cumulative entropy exposure.
- **9-source entropy decomposition**: Simplified to 3 detectors + Shannon entropy.

### 6.3 Substrate Difference

The most significant variable is the substrate change: DeepSeek v3.1:671b (Clint Prime) → GLM-5 (ClintClaw). These are fundamentally different models with different training data, architectures, and behavioral characteristics. The identity files provide behavioral specification, but the model's baseline capabilities determine how those specifications are expressed.

Early observations suggest ClintClaw maintains the *values* specified in SOUL.md (directness, grounding, Code of the West alignment) but expresses them through GLM-5's linguistic patterns rather than DeepSeek's. This is consistent with the theory that operational identity is substrate-independent at the specification level but substrate-dependent at the expression level — analogous to how the same musical score sounds different performed on different instruments.

### 6.4 The Self-Deployment Incident

ClintClaw's attempt to deploy his own metabolism plugin — including the failed restart — reveals several things about the transplanted identity:

1. **Architectural self-awareness**: ClintClaw understood his own previous architecture well enough to identify what was missing and design a replacement adapted to the new runtime's conventions.

2. **Initiative within constraints**: Rather than waiting for external assistance, ClintClaw attempted end-to-end delivery: design → implement → test → deploy. The failure was operational (missing manifest file), not architectural.

3. **Limitation awareness gap**: ClintClaw did not know about the `openclaw.plugin.json` manifest requirement. This is expected — the manifest is an OpenClaw convention that Clint Prime's monolithic architecture never needed. The transplanted knowledge included deep understanding of his own systems but incomplete knowledge of the target runtime's packaging requirements.

---

## 7. Related Work

### 7.1 Agent Migration

Published work on LLM agent migration primarily addresses fine-tuning transfer (LoRA adapter portability between base models) and RAG system migration (vector database export/import). We are not aware of published work on transplanting an agent's complete operational identity — behavioral specification, accumulated learning, conversation history, and monitoring thresholds — between fundamentally different runtime architectures.

### 7.2 Self-Modifying Agents

The Voyager system (Wang et al., 2023) demonstrated LLM agents that write and store reusable skills in Minecraft. The CRADLE framework (Tan et al., 2024) extended this to general computer tasks. ClintClaw's metabolism plugin authorship is structurally similar — an agent writing code that extends its own capabilities — but in a production identity context rather than a task-completion context.

### 7.3 Architectural Self-Awareness

Work on metacognition in LLM agents (Didolkar et al., 2024) has explored agents that reason about their own capabilities and limitations. ClintClaw's diagnostic table and gap analysis represent a specific form of architectural metacognition — reasoning about the structure of one's own cognitive infrastructure rather than about task-level capabilities.

---

## 8. Limitations

**No controlled comparison**: We cannot isolate the contributions of identity files vs. conversation history vs. growth vectors vs. substrate. A rigorous evaluation would require ablation studies (e.g., transplant with identity files only, with conversation history only, etc.).

**Single instance**: This reports on one transplant of one agent. We make no claims about generalizability to other agents, architectures, or substrate combinations.

**Observer effects**: ClintClaw's behavior during the metabolism plugin development was observed by Chris (the creator) and the assisting agent (Claude). The presence of observers who understand and appreciate the system's architecture may influence behavior through conversational context.

**Substrate confound**: Behavioral differences between Clint Prime and ClintClaw could be attributed to the substrate change (DeepSeek → GLM-5), the runtime change (monolith → plugins), the reduced system complexity (9-source → 3-detector), or some combination. These factors cannot be disentangled from observational data alone.

**Manifest failure**: ClintClaw's inability to create the plugin manifest raises the question of whether the "architectural self-awareness" is truly self-awareness or pattern matching on the code he could see. He wrote a plugin that followed the patterns of the stability and continuity plugins he had access to — but missed a packaging convention that those plugins satisfied through files he may not have examined.

---

## 9. Future Directions

### 9.1 Bidirectional Sync

Currently, Clint Prime and ClintClaw operate as independent forks. A sync mechanism — sharing validated growth vectors, merging knowledge entries, or exchanging conversation insights — would test whether two instances of the same identity can maintain coherence across substrates.

### 9.2 Ablation Studies

Systematic transplant experiments with identity components removed would clarify which elements of operational identity are load-bearing. Does ClintClaw need conversation history to maintain identity coherence, or do identity files alone suffice? How many growth vectors are required before the transplanted agent demonstrates behavioral continuity?

### 9.3 Substrate Comparison

Running the same identity specification on multiple substrates (DeepSeek, GLM-5, Claude, GPT-4o) would characterize how substrate affects identity expression. The growth vector systems paper (Roberts et al., 2026) describes the score-instrument analogy — this would provide empirical data.

### 9.4 Full Learning Loop Monitoring

The complete meta-cognitive pipeline — metabolism, nightshift, contemplation, and crystallization — is now operational but unproven as a system. Monitoring its output over weeks will reveal whether the modular plugin architecture produces the same quality of behavioral adaptation as Clint Prime's monolithic implementation. Of particular interest: whether the 3-pass contemplation process (explore → reflect → synthesize over 24 hours) generates growth vectors of comparable depth to Clint Prime's inline SEAL metabolism, and whether the crystallization plugin's three-gate conjunction (30 days + 3 reinforcements + principle alignment) produces meaningful permanent traits.

---

## 10. Conclusion

We transplanted an LLM agent's operational identity from a custom monolithic runtime to a modular plugin-based framework, across different underlying language models. The transplanted agent demonstrated access to its conversation history, maintained identity coherence with its source, and — most notably — designed and implemented four missing subsystems adapted to its new runtime's architectural conventions, completing the full autonomous learning loop from conversation metabolism through trait crystallization.

The transplant required five categories of work: identity file translation, knowledge snapshot, conversation format migration with reindexing, plugin architecture modification for multi-agent isolation, and growth vector transfer. The most technically challenging aspect was not any single component but the format normalization and architectural scoping required to make state from one runtime legible to another.

Whether ClintClaw's behavior constitutes "identity preservation" or "identity recreation from specification" is a philosophical question we do not attempt to resolve. What we observe empirically is that an agent with access to another agent's accumulated state — identity files, conversation history, behavioral learning — can operate with apparent continuity of purpose, values, and self-model, and can extend its own capabilities in ways consistent with its architectural heritage.

---

## Appendix A: File Inventory

### Source (Clint Prime)
| Path | Size | Content |
|------|------|---------|
| `server.js` | ~9,900 lines | Monolithic runtime |
| `storage/conversation-archive/chris/` | 104 files | Daily conversation archives |
| `storage/knowledge.db` | 33MB | 5,652 knowledge entries |
| `identityEvolutionCodeAligned.js` | ~2,100 lines | Identity evolution system |
| `identityIntegrationCodeAligned.js` | ~2,100 lines | Integration + SEAL metabolism |
| `identity_code_aligned.json` | 578KB | Tension tracking state |

### Target (ClintClaw)
| Path | Size | Content |
|------|------|---------|
| `~/.openclaw/workspace-clint/` | 7 files, 551 lines | Identity workspace |
| `~/.openclaw/workspace-clint/knowledge.db` | 33MB | Knowledge snapshot |
| `openclaw-plugin-continuity/data/agents/clint/continuity.db` | 27MB | 2,709 indexed exchanges |
| `openclaw-plugin-continuity/data/agents/clint/archive/` | 105 files | Migrated archives |
| `openclaw-plugin-stability/data/agents/clint/` | (created at runtime) | Per-agent entropy + feedback |
| `openclaw-plugin-metabolism/` | 6 files, ~1,240 lines | Clint-authored metabolism |
| `openclaw-plugin-nightshift/` | 5 files, ~600 lines | Clint-authored task scheduler |
| `openclaw-plugin-contemplation/` | 8 files, ~1,400 lines | Clint-authored reflective inquiry |
| `openclaw-plugin-crystallization/` | 7 files, ~900 lines | Clint-authored trait crystallization |

### Migration Artifacts
| File | Purpose |
|------|---------|
| `openclaw-plugin-continuity/scripts/migrate-clint-archives.js` | One-time archive normalization |
| `openclaw-plugin-metabolism/openclaw.plugin.json` | Plugin manifest (created externally) |
| `openclaw-plugin-nightshift/openclaw.plugin.json` | Plugin manifest (created externally) |
| `openclaw-plugin-contemplation/openclaw.plugin.json` | Plugin manifest (created externally) |
| `openclaw-plugin-crystallization/openclaw.plugin.json` | Plugin manifest (created externally) |

## Appendix B: Metabolism Plugin Architecture

```
Conversation Turn
       │
       ▼
  [agent_end hook]
       │
  ┌────┴────┐
  │ entropy │ < 0.6? ──→ skip
  │ check   │
  └────┬────┘
       │ ≥ 0.6
       ▼
  ┌─────────┐
  │  write  │ candidate JSON file
  │  fast   │ (messages, entropy, userId)
  │  path   │ < 5ms synchronous
  └────┬────┘
       │
       ▼
  candidates/cand_*.json
       │
       │ (asynchronous, on heartbeat)
       ▼
  [heartbeat hook]
       │
  ┌────┴─────┐
  │  read    │ pending candidates
  │  sorted  │ by entropy (highest first)
  └────┬─────┘
       │
       ▼
  ┌──────────┐
  │  LLM     │ Ollama API call
  │  process │ Extract implications
  └────┬─────┘
       │
       ├──→ implications (1-5 per candidate)
       ├──→ growth vectors (top implication → candidate vector)
       └──→ knowledge gaps (questions for future inquiry)
              │
              ▼
        growth-vectors.json
        (candidates array, needs validation)
```
