# Reverse Engineered Curriculum — CCA-F Study Guide

---

A learning objective reference matrix for each domain. Every micro-concept extracted from the official curriculum. Use this to track what you know and what you still need to study.

**Legend:**

| Column | Meaning |
|---|---|
| **Importance** | How critical this concept is for real-world Claude development |
| **Difficulty** | How hard it is to understand and remember |
| **Probability** | Likelihood this concept appears on the exam |

---

## Domain 1: Agent Architecture and Orchestration (27%)

| # | Learning Objective / Micro-Concept | Importance | Difficulty | Probability |
|---|---|---|---|---|
| 1.1 | **Agent loop lifecycle: send request → check stop_reason → execute tools → return results** | Critical | Medium | High |
| 1.2 | `stop_reason: tool_use` signals model wants to call a tool | Critical | Easy | High |
| 1.3 | `stop_reason: end_turn` signals complete response, no tools needed | Critical | Easy | High |
| 1.4 | Model-driven decision making (let the model decide next action) vs hard-coded decision trees (developer pre-defines all paths) | Critical | Medium | High |
| 1.5 | **Anti-pattern: parsing assistant text for completion** — never parse the model's generated text to determine if it's done | Critical | Medium | High |
| 1.6 | **Anti-pattern: arbitrary iteration limits** — don't hard-code max iterations; let the model decide when it's done | High | Medium | High |
| 1.7 | Hub-and-spoke architecture for multi-agent systems — one coordinator, multiple subagents | Critical | Medium | High |
| 1.8 | Coordinator responsibilities: decomposition (break task), delegation (assign to subagents), aggregation (combine results), error handling (catch and recover) | Critical | Medium | High |
| 1.9 | Subagents have **isolated context** — they don't see each other's conversations | Critical | Medium | High |
| 1.10 | **Task tool** — primary mechanism for spawning subagents from Claude Code | Critical | Medium | High |
| 1.11 | **Explicit context passing** — subagents do NOT inherit parent context; you must explicitly pass what they need | Critical | Medium | High |
| 1.12 | **Parallel spawning** — multiple Task calls run subagents in parallel | High | Medium | Medium |
| 1.13 | **AgentDefinition configuration** — how you define agent role, tools, and instructions | High | Medium | Medium |
| 1.14 | Programmatic enforcement (code-level guards) vs prompt guidance (ask politely via system prompt) | Critical | Hard | High |
| 1.15 | Structured handoff protocols during escalation — predefined flows for passing control between agents | High | Medium | Medium |
| 1.16 | **PostToolUse hook** — runs after tool execution, useful for normalizing tool output | Critical | Medium | High |
| 1.17 | Pre-tool-use interception hooks — run before tool executes, useful for validation or modification | High | Hard | Medium |
| 1.18 | Deterministic guarantees (code-enforced, always happens) vs probabilistic compliance (prompt-enforced, may fail) | Critical | Medium | High |
| 1.19 | Fixed pipelines (prompt chaining) — predetermined sequence of steps, good for known workflows | High | Medium | High |
| 1.20 | Dynamic adaptive decomposition — model decides how to break down the task at runtime | High | Hard | Medium |
| 1.21 | Multi-pass code review — running multiple review passes with different focus areas | High | Medium | Medium |
| 1.22 | `--resume` flag — resumes a previous session from its last state | Critical | Easy | High |
| 1.23 | `fork_session` — create a branch from a previous session point | High | Medium | Medium |
| 1.24 | New session vs resume decision — when to start fresh vs pick up where you left off | High | Medium | High |

### Domain 1 — Quick Summary

- Agentic loop is the **foundation** — know every step
- Anti-patterns are **exam favorites** — they test if you know what NOT to do
- Coordinator-subagent is the **primary multi-agent pattern** tested
- Deterministic > probabilistic when reliability matters

---

## Domain 2: Tool Design and MCP Integration (18%)

| # | Learning Objective / Micro-Concept | Importance | Difficulty | Probability |
|---|---|---|---|---|
| 2.1 | **Descriptions are the primary selection mechanism** for LLMs — the model reads descriptions to decide which tool to call | Critical | Medium | High |
| 2.2 | Write tool descriptions that include: input formats, examples, edge cases, boundaries | Critical | Medium | High |
| 2.3 | Avoid overlapping descriptions — similar descriptions confuse the model's tool selection | High | Medium | High |
| 2.4 | **isError flag** — MCP protocol field that marks a tool response as an error | Critical | Medium | High |
| 2.5 | Error categories: transient (retryable), validation (fix input), business (policy violation), permission (access denied) | Critical | Medium | High |
| 2.6 | Distinguish retryable errors (network timeout, rate limit) vs non-retryable errors (invalid input, permission denied) | Critical | Medium | High |
| 2.7 | **Generic errors** (uninformative strings) vs **structured errors** (with category, code, details, retryable flag) | High | Medium | High |
| 2.8 | **Too many tools reduces reliability** — more tools = more confusion in selection | Critical | Easy | High |
| 2.9 | **Principle of least privilege** — give each agent only the tools it needs, nothing more | Critical | Easy | High |
| 2.10 | `tool_choice: auto` — model decides whether to use a tool | Critical | Easy | High |
| 2.11 | `tool_choice: any` — model must use a tool (but picks which one) | Critical | Medium | High |
| 2.12 | `tool_choice: forced` — model must use a specific tool | Critical | Medium | High |
| 2.13 | Replace general tools (e.g., a generic `search`) with constrained alternatives (e.g., `search_users` with specific schema) | High | Medium | Medium |
| 2.14 | Project `.mcp.json` vs user `~/.claude.json` — project config is scoped to project, user config applies globally | Critical | Medium | High |
| 2.15 | Environment variable substitution for secrets in MCP configs — `$VAR` syntax, never hardcode secrets | Critical | Medium | High |
| 2.16 | Community MCP servers (pre-built, shared) vs custom MCP servers (built for your specific use case) | Medium | Easy | Medium |
| 2.17 | MCP resources as content catalogs — expose read-only data (docs, schemas, configs) as resources | High | Medium | Medium |
| 2.18 | **Built-in tools:** Read, Write, Edit, Bash, Grep, Glob — know what each does and when to use them | Critical | Easy | High |

### Domain 2 — Quick Summary

- **Descriptions matter most** — this is how LLMs pick tools, so exam questions test your ability to write good descriptions
- **isError and structured errors** — exam will test whether you know the right error category
- **tool_choice: auto vs any vs forced** — know the difference cold
- **MCP config files** (.mcp.json vs ~/.claude.json) and env var substitution are frequently tested

---

## Domain 3: Claude Code Configuration and Workflows (20%)

| # | Learning Objective / Micro-Concept | Importance | Difficulty | Probability |
|---|---|---|---|---|
| 3.1 | 3-level hierarchy for CLAUDE.md: **user** (global), **project** (repo root), **directory** (subfolder-specific) | Critical | Medium | High |
| 3.2 | `@path` syntax for file imports within CLAUDE.md — reference external files for reusable content | Critical | Medium | High |
| 3.3 | `.claude/rules/` directory — topic-focused rules that load based on context | Critical | Medium | High |
| 3.4 | Rules with YAML frontmatter and paths for conditional loading — only load when certain paths are active | High | Hard | Medium |
| 3.5 | `.claude/commands/` — custom slash commands (user-defined actions) | Critical | Medium | High |
| 3.6 | `.claude/skills/` — packaged skills (reusable agent capabilities) | Critical | Medium | High |
| 3.7 | Commands vs skills distinction — commands are lightweight slash-triggered actions, skills are full agent capabilities | High | Medium | High |
| 3.8 | Project scope vs user scope for commands/skills | High | Medium | Medium |
| 3.9 | SKILL.md frontmatter: `context: fork` (runs in isolated session), `allowed-tools` (restrict tools), `argument-hint` (usage hint) | Critical | Medium | High |
| 3.10 | Personal skills override project skills with same name — user skills take precedence | High | Easy | Medium |
| 3.11 | **Glob patterns in `.claude/rules/`** — apply rules only to matching file paths | High | Medium | Medium |
| 3.12 | Path-specific rules **save context and tokens** — only load rules that are relevant to current files | Critical | Medium | High |
| 3.13 | **Planning mode** — investigation only, no file changes. Use for complex tasks needing analysis before action | Critical | Easy | High |
| 3.14 | **Direct execution** — simple, well-understood changes. No planning phase needed | Critical | Easy | High |
| 3.15 | **Explore subagent** — spawn an isolated subagent for research without polluting main context | High | Medium | Medium |
| 3.16 | Concrete input/output examples — show the model exact before/after to guide refinement | High | Medium | Medium |
| 3.17 | Test-driven iteration — write tests first, then refine until tests pass | High | Medium | Medium |
| 3.18 | Interview pattern — ask clarifying questions before generating code | Medium | Medium | Low |
| 3.19 | `/compact` command — compress conversation history to save context | High | Easy | Medium |
| 3.20 | `/memory` command — store/retrieve persistent information across sessions | High | Easy | Medium |
| 3.21 | `-p` flag — non-interactive mode for programmatic/CI use | Critical | Easy | High |
| 3.22 | `--output-format json` — get structured JSON output for parsing | Critical | Medium | High |
| 3.23 | `--json-schema` — define a schema for structured output | High | Medium | Medium |
| 3.24 | Session context isolation for code review — each review in its own session to avoid context bleed | High | Medium | Medium |
| 3.25 | **Batch API** — 50% cost savings, up to 24hr processing, **NO multi-turn tool calling** | Critical | Medium | High |

### Domain 3 — Quick Summary

- CLAUDE.md hierarchy (user → project → directory) and `.claude/rules/` are heavily tested
- Know the difference between commands (slash-triggered) and skills (full agent capabilities)
- Planning mode vs direct execution — exam loves this distinction
- Batch API tradeoffs (cost savings vs no multi-turn tool calls, 24hr max) are exam favorites
- `-p`, `--output-format json`, `--json-schema` — know these for CI/CD questions

---

## Domain 4: Prompt Engineering and Structured Output (20%)

| # | Learning Objective / Micro-Concept | Importance | Difficulty | Probability |
|---|---|---|---|---|
| 4.1 | **Explicit criteria** vs vague instructions — specific rules produce more reliable outputs | Critical | Easy | High |
| 4.2 | False positive impact on trust — every incorrect output erodes confidence in the system | Critical | Medium | High |
| 4.3 | Define severity criteria with examples (e.g., critical: security issue, minor: formatting issue) | High | Medium | Medium |
| 4.4 | **Few-shot prompting > textual descriptions** — examples outperform instructions for consistency | Critical | Medium | High |
| 4.5 | Use few-shot examples for ambiguous scenarios — show model how to handle edge cases | High | Medium | High |
| 4.6 | Use few-shot examples for output formatting — show exact desired output structure | High | Easy | Medium |
| 4.7 | Few-shot examples distinguishing acceptable vs problematic — show both good and bad examples | High | Medium | Medium |
| 4.8 | **JSON schemas eliminate syntax errors, NOT semantic errors** — output will be valid JSON but may be wrong content | Critical | Medium | High |
| 4.9 | Required vs optional fields in JSON schema — model may omit optional fields | High | Medium | Medium |
| 4.10 | Nullable fields, enums with "other" and "unclear" values — handle unexpected inputs gracefully | High | Medium | Medium |
| 4.11 | `tool_choice: auto` vs `any` vs `forced` for structured output — which to use when | Critical | Medium | High |
| 4.12 | **Retry-with-error-feedback** — when output fails validation, retry with the error message as feedback | Critical | Medium | High |
| 4.13 | When retry is ineffective — if the information the model needs is simply absent in context, retrying won't help | High | Medium | High |
| 4.14 | Self-correction pattern (stated_total vs calculated_total) — ask model to verify its own arithmetic | High | Hard | Medium |
| 4.15 | **Pydantic for validation** — Python library for validating structured outputs against schemas | High | Medium | Medium |
| 4.16 | Synchronous API for blocking checks — real-time validation that waits for response | High | Easy | Medium |
| 4.17 | **Batch API for overnight/weekly** — non-blocking, cheaper, no multi-turn | Critical | Medium | High |
| 4.18 | `custom_id` for failure handling — tag each batch request so you know which one failed | High | Medium | Medium |
| 4.19 | SLA planning — choosing sync vs batch based on how fast you need results | High | Medium | Medium |
| 4.20 | **Independent review instances avoid confirmation bias** — each review runs without seeing other reviews | Critical | Medium | High |
| 4.21 | Multi-pass review for large PRs — break into passes (security → style → logic) | High | Medium | Medium |

### Domain 4 — Quick Summary

- Explicit criteria + few-shot examples = best prompt design strategy
- JSON schemas fix syntax, not semantics — know this distinction
- Retry-with-error-feedback is the standard pattern for handling bad output
- Batch API is for non-urgent, bulk processing (no multi-turn tool calling)
- Independent review instances prevent confirmation bias — exam tests this

---

## Domain 5: Context Management and Reliability (15%)

| # | Learning Objective / Micro-Concept | Importance | Difficulty | Probability |
|---|---|---|---|---|
| 5.1 | **Progressive summarization risks** — numbers, dates, and precise details get lost when summarizing | Critical | Medium | High |
| 5.2 | **Lost-in-the-middle effect** — model performs worse on information in the middle of long context | Critical | Medium | High |
| 5.3 | Extract facts into separate block — pull critical facts out of long narratives into structured summaries | High | Medium | High |
| 5.4 | Trim verbose tool outputs — long tool responses waste context, truncate or summarize them | High | Medium | High |
| 5.5 | Escalation triggers: explicit request (user asks to escalate), policy gaps (no rule for this situation), inability to progress (stuck) | Critical | Medium | High |
| 5.6 | **Immediate escalation** (escalate at first uncertainty) vs **attempt-then-escalate** (try once, then escalate on failure) | Critical | Medium | High |
| 5.7 | Ask for additional identifiers on ambiguous matches — when model can't distinguish between similar entities | High | Medium | Medium |
| 5.8 | **Unreliable: sentiment analysis, self-rated confidence** — models are bad at judging their own confidence | High | Medium | Medium |
| 5.9 | **Structured error context** — pass error details (type, message, recoverable flag) to the next agent/handler | Critical | Medium | High |
| 5.10 | Distinguish access failures (can't reach data) from valid empty results (data legitimately empty) | High | Medium | High |
| 5.11 | Local recovery in subagents — let subagents retry/fix errors themselves before escalating | High | Medium | Medium |
| 5.12 | Coverage annotations — mark which paths have been checked so agents don't re-check | Medium | Hard | Low |
| 5.13 | **Scratchpad files** — write intermediate findings to files to free up conversation context | High | Medium | High |
| 5.14 | **Delegating to subagents** reduces context pressure — each subagent has its own isolated context | High | Medium | High |
| 5.15 | Context degradation signs — model starts repeating itself, missing details, or making contradictory statements | High | Medium | Medium |
| 5.16 | **Aggregate metrics can mask poor performance on specific types** — e.g., 95% overall accuracy could hide 50% accuracy on edge cases | Critical | Medium | High |
| 5.17 | **Stratified random sampling** — test across all categories, not just easy ones | High | Medium | Medium |
| 5.18 | Field-level confidence scores — report confidence per output field, not just overall | High | Hard | Medium |
| 5.19 | **Claim → source mapping** — always track which source produced which piece of information | Critical | Medium | High |
| 5.20 | **Handling conflicting data with attribution** — present conflicting sources with labels, don't silently pick one | Critical | Medium | High |
| 5.21 | Include dates for temporal interpretation — "X said Y (2024)" vs "X said Y (2025)" make a difference | High | Easy | Medium |
| 5.22 | Render by content type — format code blocks as code, tables as tables, citations as footnotes, etc. | Medium | Easy | Low |

### Domain 5 — Quick Summary

- **Summarization loses precision** — always extract critical facts separately
- **Lost-in-the-middle effect** is real — design context to put important info at the start or end
- Aggregate metrics lie — stratify your testing
- Claim → source mapping is the solution to multi-source confusion
- Escalation patterns (immediate vs attempt-then-escalate) are frequently tested

---

## Master Study Matrix

| Domain | Weight | # Concepts | Critical | High | Medium | Low | Exam Probability (High) |
|---|---|---|---|---|---|---|---|
| 1 — Agent Architecture | 27% | 24 | 8 | 8 | 8 | 0 | 15 |
| 2 — Tool Design + MCP | 18% | 18 | 11 | 5 | 2 | 0 | 11 |
| 3 — Claude Code Config | 20% | 25 | 11 | 10 | 4 | 0 | 11 |
| 4 — Prompt Engineering | 20% | 21 | 8 | 12 | 1 | 0 | 10 |
| 5 — Context Management | 15% | 22 | 7 | 11 | 4 | 0 | 13 |
| **Total** | **100%** | **110** | **45** | **46** | **19** | **0** | **60** |

---

## Study Order (Recommended)

1. **Domain 1** — heaviest weight, most Critical concepts
2. **Domain 2** — builds on Domain 1 (tools are called in agentic loops)
3. **Domain 4** — independent, medium weight
4. **Domain 3** — practical configuration, good after understanding agents + tools
5. **Domain 5** — lowest weight, save for last

---

## Quick Reference Card (Memory Triggers)

```
Domain 1: Agent loop → stop_reason → tool_use vs end_turn → anti-patterns → 
          coordinator-subagent → Task tool → explicit context → parallel spawning →
          programmatic vs prompt → deterministic vs probabilistic

Domain 2: Descriptions = selection mechanism → isError flag → error categories →
          tool_choice (auto/any/forced) → least privilege → .mcp.json vs ~/.claude.json →
          env var substitution

Domain 3: CLAUDE.md hierarchy (user → project → directory) → @path → .claude/rules/ →
          commands vs skills → SKILL.md frontmatter → planning vs direct execution →
          -p flag → output-format json → Batch API (50% off, 24hr, no multi-turn)

Domain 4: Explicit criteria → few-shot > descriptions → JSON schema = syntax not semantics →
          retry-with-error-feedback → Pydantic → sync vs Batch API →
          independent review instances → no confirmation bias

Domain 5: Summarization loses precision → lost-in-the-middle → extract facts separately →
          immediate vs attempt-then-escalate → structured error context →
          scratchpad files → aggregate metrics lie → claim → source mapping
```
