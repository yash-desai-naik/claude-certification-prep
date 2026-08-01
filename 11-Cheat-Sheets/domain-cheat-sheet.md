# Domain Cheat Sheet — Claude Certified Architect Foundations

One-page last-minute revision. All critical facts in compact tables.

---

## 1. Domain Weightage

| Domain | Focus | Weight |
|--------|-------|--------|
| **Domain 1** | Foundations — Agent arch, context, prompts, planning | 15-20% |
| **Domain 2** | Tool Use & Skills — Definitions, tool_choice, structured output | 20-25% |
| **Domain 3** | MCP — Protocol, servers, resources, tools, .mcp.json | 20-25% |
| **Domain 4** | Workflow & Multi-Agent — Loops, hub-and-spoke, chaining | 15-20% |
| **Domain 5** | Testing, Eval & Production — Batch API, eval, error handling | 15-20% |

---

## 2. stop_reason Values

| Value | Meaning | Action |
|-------|---------|--------|
| `end_turn` | Claude finished naturally | Terminate agent loop |
| `tool_use` | Claude wants to call a tool | Execute tool, feed back result |
| `max_tokens` | Hit token generation limit | Truncated — increase `max_tokens` or retry |
| `stop_sequence` | Custom stop sequence matched | Handle per application logic |

---

## 3. tool_choice Modes

| Mode | Behavior | When to Use |
|------|----------|-------------|
| `auto` | Claude decides tool vs text | Default. General purpose |
| `any` | Must use a tool every turn | Need structured output every round |
| `tool` | Force a specific tool | Single-task routing |
| `none` | No tool use allowed | Pure text generation |

---

## 4. CLAUDE.md Hierarchy

```
Loaded into system prompt, in order:

1. CLAUDE.md (project root)
2. .claude/rules/*.md (alphabetical)
3. Skill definitions (when activated)
4. Session-specific instructions (at runtime)
```

- CLAUDE.md is **always loaded** if present.
- `.claude/rules/` files merge into system prompt.
- `.claude/commands/` defines custom slash commands (not auto-loaded).
- `.claude/skills/` contains skill bundles (activated per-need).

---

## 5. Skills Frontmatter Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Skill identifier |
| `description` | string | Yes | Trigger description |
| `allowed-tools` | list | No | Tools permitted (e.g., `[Read, Grep]`) |
| `model` | string | No | Override model for this skill |

Compare with `AgentDefinition.allowed_tools` — same purpose, different scope.

---

## 6. Error Categories & Retryability

| Category | Examples | Retryable? |
|----------|----------|------------|
| **Transient** | Rate limit, timeout, server error | Yes — exponential backoff |
| **Auth** | Invalid key, expired token | No — fix credentials |
| **Validation** | Bad schema, invalid params | No — fix request |
| **Not Found** | Missing resource, wrong path | No — fix reference |
| **Overload** | Context window exceeded | Maybe — reduce input |

Batch API: Retries handled server-side for transient errors.

---

## 7. Batch API vs Sync API Decision

| Factor | Batch API | Sync API |
|--------|-----------|----------|
| Latency | Hours (polling) | Milliseconds |
| Cost | **50% discount** | Full price |
| Max requests | 100,000 per batch | Per-minute rate limits |
| SLA | 24-hour completion | Real-time |
| Use case | Bulk eval, offline processing | Chat, interactive tools |
| Error handling | Poll results, check errors array | Catch on the fly |

**Decision rule:** Need real-time? → Sync. Need throughput at half cost? → Batch.

---

## 8. Escalation Triggers

| Valid to escalate to human | NOT valid to escalate |
|---------------------------|----------------------|
| Security-sensitive action (delete, permission change) | "I'm confused by this task" — retry or rephrase |
| Destructive file operation (rm -rf, DROP TABLE) | Simple ambiguity that more context resolves |
| Out-of-scope request | First attempt failure — self-correct first |
| Production deployment approval | Minor edit confirmation — use direct execution |
| Credential/secret management | Any task within defined tool permissions |

---

## 9. Key Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|-------------|-------------|-----|
| Overly permissive `allowed-tools` | Security risk, accidental damage | Apply least privilege explicitly |
| All info in middle of prompt | Lost-in-the-middle degrades performance | Put critical info at start or end |
| No `stop_reason` check in agent loop | Loop runs forever or exits prematurely | Always check `stop_reason` after response |
| Using `tool_choice: any` when no tool is available | Wasted turn, Claude forced to fail | Use `auto` unless guaranteed tool need |
| Single agent for everything | Context window fills, task quality drops | Decompose via subagents or multi-pass |
| No progressive summarization in long sessions | Context overflow, model loses earlier work | Run `/compact` periodically |
| Forgetting Batch API `custom_id` uniqueness | Result mapping breaks | Ensure unique `custom_id` per batch entry |

---

## 10. High-Probability Exam Topics

1. **stop_reason logic** in the agent loop — given a scenario, what happens next?
2. **tool_choice comparison** — when to pick `auto` vs `any` vs `tool`
3. **Batch API constraints** — 50% cost, 100K cap, 24h SLA, custom_id requirement
4. **CLAUDE.md hierarchy** — what loads first, what overrides what
5. **MCP vs native tools** — tradeoffs, setup, resource vs tool distinction
6. **Skills frontmatter vs AgentDefinition** — overlapping but different scope
7. **Hub-and-spoke vs fixed pipeline** — which pattern for which scenario
8. **Human-in-the-loop vs direct execution** — safety boundaries
9. **Progressive summarization / /compact** — context management technique
10. **PreToolUse vs PostToolUse hooks** — interception vs observation
