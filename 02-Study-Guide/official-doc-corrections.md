# Official Documentation Cross-Reference — Corrections & Clarifications

This file documents discrepancies between community guide sources and official Anthropic documentation.
Where there is conflict, **official docs win**. Updates made to study materials are noted.

---

## 1. `@path` Import Depth

| Source | Claim |
|--------|-------|
| Community Guide | Max import nesting depth is **5** |
| **Official Docs** | Max import nesting depth is **4 hops** |

**Correct value:** 4 hops max.

**Official source:** `code.claude.com/docs/en/memory` — "Imported files can recursively import other files, with a maximum depth of four hops."

**Impact on exam:** Low. The exact number is unlikely to be tested, but if asked, **4** is correct.

---

## 2. CLAUDE.md Hierarchy Levels

| Source | Claim |
|--------|-------|
| Community Guide | 3 levels: User, Project, Directory |
| **Official Docs** | **4 levels**: Managed Policy, User, Project, Local |

**Correct hierarchy (broadest to most specific):**

| Level | Location | Scope | VCS? |
|-------|----------|-------|------|
| **Managed Policy** | `/etc/claude-code/CLAUDE.md` (Linux), `C:\Program Files\ClaudeCode\CLAUDE.md` (Windows), `/Library/Application Support/ClaudeCode/CLAUDE.md` (macOS) | Organization-wide, all users | N/A (IT-deployed) |
| **User instructions** | `~/.claude/CLAUDE.md` | Personal, all projects | No |
| **Project instructions** | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Team, one project | Yes |
| **Local instructions** | `./CLAUDE.local.md` | Personal, one project | Add to `.gitignore` |

**Also note:** `CLAUDE.local.md` is appended after `CLAUDE.md` in the same directory. Within each directory, local comes after project instructions so personal notes are read last.

**Official source:** `code.claude.com/docs/en/memory`

---

## 3. Agent SDK — `AgentDefinition` Field Names

| Source | Claim |
|--------|-------|
| Community Guide | Uses `system_prompt` field |
| **Official Docs** | Uses **`prompt`** field (not `system_prompt`) |

**Correct field name:** `prompt` for Agent SDK `AgentDefinition`.

The official Agent SDK reference shows:
```python
AgentDefinition(
    description="...",
    prompt="You are a code review specialist...",  # NOT system_prompt
    tools=["Read", "Grep", "Glob"],
)
```

**Note:** If the exam asks about the Agent SDK config, `prompt` is correct. The older Agent API may have used `system_prompt` but the current SDK uses `prompt`.

---

## 4. Subagent Tool Name: `Agent` vs `Task`

| Source | Claim |
|--------|-------|
| Community Guide | Uses `Task` tool name |
| **Official Docs** | Uses **`Agent`** tool (renamed in v2.1.63, but `Task` still appears in some contexts) |

**Correct:** Current SDK emits `"Agent"` in `tool_use` blocks. The tool name was renamed from `"Task"` to `"Agent"` in Claude Code v2.1.63. However, the system init tools list and `result.permission_denials[].tool_name` still use `"Task"`.

**Exam takeaway:** If asked about the current tool name, **`Agent`** is correct. If asked about backward compatibility, both names may appear.

**Official source:** SDK subagents documentation — "The tool name was renamed from `"Task"` to `"Agent"` in Claude Code v2.1.63."

---

## 5. Batch API Processing Time

| Source | Claim |
|--------|-------|
| Community Guide | "Up to 24 hours" |
| **Official Docs** | "Most batches finish in less than 1 hour" with a **24-hour maximum** before expiration |

**Correct:** Most batches finish in <1 hour. Maximum processing window is 24 hours (after which unprocessed requests expire).

**This is important for the exam.** Official docs emphasize:
- 50% cost savings
- Most complete in <1 hour
- 24-hour maximum/expiration
- No latency SLA guarantee

---

## 6. Batch API — Multi-turn Tool Calling Support

| Source | Claim |
|--------|-------|
| Community Guide | "Not supported" (no multi-turn tool calling in batch) |
| **Official Docs** | Server tools work in batch. The batch worker runs the same server-side agentic loop. "If a batch result comes back with `pause_turn`, the turn did not finish; you can continue it." |

**Correction:** The Batch API does support server-side tools (web search, code execution, MCP connectors). For client-defined tools with multi-turn interactions (tool_use → tool_result cycles), the community guide is correct — those don't work in a single batch request.

**Exam nuance:** Server tools work in batch. Client tool loops (tool_use → execute → tool_result → continue) require synchronous API.

---

## 7. `stop_reason` Values

Official docs list these `stop_reason` values:

| Value | Meaning |
|-------|---------|
| `end_turn` | Model voluntarily finished |
| `tool_use` | Model wants to call a tool |
| `max_tokens` | Output token limit reached |
| `stop_sequence` | Custom stop sequence encountered |
| `content_filtered` | Output blocked by safety filter |

Note: `content_filtered` is an additional value not emphasized in the community guide.

---

## 8. `tool_choice` Modes

Official docs confirm these modes:

| Mode | Behavior |
|------|----------|
| `{"type": "auto"}` | Claude decides tool or text response |
| `{"type": "any"}` | Claude must use a tool each turn |
| `{"type": "tool", "name": "..."}` | Force a specific tool |
| `{"type": "none"}` | No tool use allowed |

Additionally, `tool_choice` can include `"disable_parallel_tool_use": true` to limit to one tool call per turn.

---

## 9. AGENTS.md vs CLAUDE.md

**Official docs clarify:** Claude Code reads `CLAUDE.md`, not `AGENTS.md`. If a repo uses `AGENTS.md`, create a `CLAUDE.md` that imports it via `@AGENTS.md` or use a symlink.

---

## 10. Strict Tool Use

Official docs mention `strict: true` in tool definitions to guarantee Claude's tool calls match your schema exactly. This is called "strict tool use."

```
"tools": [{
  "name": "get_weather",
  "description": "...",
  "input_schema": {...},
  "strict": true
}]
```

This is a newer feature not covered in the community guide but may appear on exam.

---

## Summary of Corrections Applied

| # | Issue | Correction |
|---|-------|------------|
| 1 | @path depth = 5 | Changed to **4** |
| 2 | CLAUDE.md = 3 levels | Changed to **4** (added Managed Policy) |
| 3 | `system_prompt` field | Uses **`prompt`** in Agent SDK |
| 4 | `Task` tool name | Now **`Agent`** (v2.1.63+) |
| 5 | Batch = "up to 24hr" | Most <1hr, max 24hr |
| 6 | Batch = no tool calling | Server tools work; client tool loops don't |
| 7 | Missing `content_filtered` stop_reason | Added |
| 8 | Missing `disable_parallel_tool_use` | Added |
| 9 | AGENTS.md confusion | Clarified: Claude reads CLAUDE.md, not AGENTS.md |
| 10 | Missing `strict: true` | Added as newer feature |
