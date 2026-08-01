# One Page Per Domain — Claude Certified Architect Foundations

Five compact reference cards, one per domain. Designed for rapid lookup.

---

# DOMAIN 1 — Foundations (15-20%)

## Top 5 Concepts

1. **Agent loop** — Observe → Reason → Act (respond/tool) → Observe result → Repeat. Check `stop_reason` every cycle.
2. **Context window** — Finite token space. Critical info at start/end to avoid lost-in-the-middle.
3. **System prompt construction** — CLAUDE.md + `.claude/rules/*.md` + skill instructions merge in order.
4. **Planning mode** — Structured plan first, then execute. Human reviews plan before action.
5. **Extended thinking** — More tokens for deeper reasoning. Tradeoff: accuracy vs cost/latency.

## Key Numbers

| Parameter | Value |
|-----------|-------|
| Context window (Sonnet) | 200K tokens |
| `max_tokens` default | 8192 |
| Extended thinking max | 128K tokens |

## Decision Tree: Which Mode?

```
Task complexity?
├── Simple, single-step → Direct execution
├── Multi-step, low risk → Agentic mode (auto tool_choice)
├── Multi-step, high risk → Planning mode first
└── Needs deep reasoning → Extended thinking
```

## Exam Traps

- ❌ **Trap:** Thinking `end_turn` means error. No — it means Claude finished normally.
- ❌ **Trap:** Assuming system prompt rules are optional hints. No — they are binding instructions.
- ❌ **Trap:** Putting examples in the middle of a long prompt. Claude will miss them.

## Memory Tricks

- **Agent Loop = SORA:** See → Observe → Reason → Act
- **Lost-in-the-middle = Oreo:** Cream (important stuff) at start or end, cookie (filler) in middle
- **stop_reason = STOP sign:** Always check before moving

---

# DOMAIN 2 — Tool Use & Skills (20-25%)

## Top 5 Concepts

1. **tool_choice modes** — `auto` (default, Claude decides), `any` (must use tool), `tool` (force specific), `none` (no tools).
2. **Skills framework** — YAML frontmatter + instructions in `.claude/skills/`. Bundles tools + prompts.
3. **Structured output** — `--output-format json` (wrap in JSON) vs `--json-schema` (enforce schema).
4. **allowed-tools** — Skill-level tool restriction. Same concept as `allowed_tools` in AgentDefinition but different scope.
5. **Tool definition** — JSON Schema for parameters. `argument-hint` improves quality.

## Key Numbers

| Parameter | Detail |
|-----------|--------|
| Tools per request | No hard limit, practical ~10-20 |
| `tool_choice: any` | Claude MUST use a tool each turn |
| `tool_choice: auto` | Claude decides on its own |

## Decision Tree: Which tool_choice?

```
Need guaranteed structured output every turn?
├── Yes → tool_choice: any
├── Need a specific tool only → tool_choice: tool (specify name)
└── No preference / mixed → tool_choice: auto

Need JSON output?
├── Any JSON shape → --output-format json
└── Exact schema required → --json-schema (or Pydantic)
```

## Exam Traps

- ❌ **Trap:** Confusing skill `allowed-tools` (frontmatter) with AgentDefinition `allowed_tools`. Same concept, different objects.
- ❌ **Trap:** Using `--output-format json` when you need `--json-schema`. First just wraps, second validates.
- ❌ **Trap:** Assuming `tool_choice: any` works without tools defined. It fails — no tool available.

## Memory Tricks

- **tool_choice modes = AAAN:** Auto, Any, tool, None
- **Skills = Tool Backpack:** Define tools + rules, activate when needed
- **JSON Schema = Blueprint:** `--output-format json` = put in box, `--json-schema` = enforce shape

---

# DOMAIN 3 — Model Context Protocol (MCP) (20-25%)

## Top 5 Concepts

1. **MCP architecture** — MCP Server (process) ↔ Client (Claude SDK/Code) via Stdio or SSE transport.
2. **MCP Tools vs Resources** — Tools = actions (write/execute), Resources = data (read only).
3. **.mcp.json configuration** — Defines server name, transport, command, args, env vars. Platform overrides supported.
4. **Error handling** — `isError` flag on tool result/resource read. Transient errors retryable, others need intervention.
5. **MCP Servers** — Can use Stdio (subprocess, local) or SSE (HTTP, remote). Each server runs independently.

## Key Numbers

| Parameter | Detail |
|-----------|--------|
| Transport types | Stdio, SSE |
| .mcp.json location | Project root |
| Platform overrides | e.g., `darwin-arm64` key for Mac Silicon |

## Decision Tree: MCP vs Native Tool?

```
Feature to add?
├── Local, simple, one-off → Native tool (inline)
├── External API/database → MCP Server (SSE for remote, Stdio for local)
├── Reusable across projects → MCP Server
└── Need auth, state, or complex logic → MCP Server
```

## Exam Traps

- ❌ **Trap:** Thinking MCP Resources are the same as MCP Tools. Resources = data, Tools = actions.
- ❌ **Trap:** Forgetting `.mcp.json` platform override syntax. Different OS paths may need different configs.
- ❌ **Trap:** Assuming MCP servers are persistent daemons. They're processes spawned per session.

## Memory Tricks

- **MCP = USB for AI:** Plug in any tool/data source via standard port
- **Tools vs Resources = Write vs Read:** Tools do, Resources have
- **Stdio = Local pipe, SSE = HTTP pipe**

---

# DOMAIN 4 — Workflow Design & Multi-Agent (15-20%)

## Top 5 Concepts

1. **Hub-and-Spoke** — Coordinator agent delegates to specialized subagents, aggregates results.
2. **Fixed pipeline / Prompt chaining** — Predefined sequential steps, each output feeds next.
3. **Dynamic decomposition** — Runtime task splitting into subtasks, often via Task tool + subagents.
4. **Subagent lifecycle** — AgentDefinition → spawn (Task tool) → execute → return result → terminate.
5. **Multi-pass review** — Multiple subagents review same output from different angles (security, correctness, style).

## Key Numbers

| Concept | Detail |
|---------|--------|
| Subagent isolation | Each subagent has own context window |
| Task tool | Spawns subagent with AgentDefinition |
| `context: fork` | Branch conversation for parallel exploration |

## Decision Tree: Which Architecture?

```
Task structure?
├── Sequential steps, known order → Fixed pipeline / Prompt chaining
├── Parallel subtasks, need coordinator → Hub-and-spoke
├── Unknown structure, runtime decisions → Dynamic decomposition
├── Need quality → Add multi-pass review
└── Simple, single agent → Direct agent loop
```

## Exam Traps

- ❌ **Trap:** Assuming more subagents = always better. Each subagent adds cost and context overhead.
- ❌ **Trap:** Confusing hub-and-spoke with dynamic decomposition. Hub-and-spoke = known roles, dynamic = discovered at runtime.
- ❌ **Trap:** Not terminating subagents. Each spawned subagent consumes resources until done.

## Memory Tricks

- **Hub-and-Spoke = Manager:** Boss (coordinator) delegates to team (subagents)
- **Fixed Pipeline = Assembly Line:** Step A → Step B → Step C
- **Dynamic Decomposition = Improv:** Figure out steps as you go

---

# DOMAIN 5 — Testing, Evaluation & Production (15-20%)

## Top 5 Concepts

1. **Batch API** — 100K requests/batch, 50% discount, poll for results, 24h SLA, requires `custom_id`.
2. **Error handling** — Transient (retry) vs non-transient (fix request). Batch retries handled server-side.
3. **Provenance** — Track which agent/tool produced what. Critical for audit and debugging.
4. **Evaluation methodology** — Coverage annotations, stratified random sampling, systematic evaluation.
5. **Human-in-the-loop** — Safety pattern. Human approves high-risk actions before execution.

## Key Numbers

| Parameter | Value |
|-----------|-------|
| Batch API discount | 50% |
| Max requests per batch | 100,000 |
| Completion SLA | 24 hours |
| Rate limit (Batch) | No per-minute penalties |
| `--resume` | Restores previous session |

## Decision Tree: Batch vs Sync?

```
Use case?
├── Real-time, interactive → Sync API
├── Bulk offline processing → Batch API
├── Cost-sensitive, can wait → Batch API
├── Need results immediately → Sync API
└── Large-scale evaluation → Batch API (+ stratified sampling)
```

## Exam Traps

- ❌ **Trap:** Forgetting `custom_id` in Batch API requests. It's required for result mapping.
- ❌ **Trap:** Using Batch API for real-time features. Batch has hours of latency.
- ❌ **Trap:** Assuming evaluation means testing everything. Use stratified sampling, not exhaustive.
- ❌ **Trap:** Missing coverage annotations in evaluation. Without them, you don't know what was tested.

## Memory Tricks

- **Batch = Bulk Discount:** 50% off, but you wait
- **custom_id = Tracking Number:** Maps request to result
- **Provenance = Receipt Trail:** Who did what, when
- **Stratified Sampling = Representative Pie:** Slice from each category, not random scoop
