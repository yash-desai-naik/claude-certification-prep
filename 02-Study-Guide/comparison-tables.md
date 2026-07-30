# Comparison Tables — Claude Certified Architect Foundations

> Quick-reference tables for every key pair on the exam. Understand the *distinction*, not just the definition.

---

## 1. Prompt vs System Prompt

| Feature | Prompt | System Prompt | Key Takeaway |
|---|---|---|---|
| **What it is** | The user's input in a message turn | The overarching instruction prepended to the conversation | System prompt sets the stage; prompt is the actor's line. |
| **Scope** | Per-message | Per-conversation (unless updated) | System prompt applies across *all* turns in the conversation. |
| **Priority** | Later instructions usually override earlier ones | Acts as base layer — user prompt can override, but system prompt sets default behavior | If a user prompt contradicts the system prompt, the model may follow the more recent instruction (the user prompt). |
| **Persistence** | Only for that turn | Persists for the entire conversation unless explicitly replaced | System prompt = persistent role definition. |
| **Typical use** | Task input, questions, file content | Role definition, formatting rules, guardrails, constraints | Use system prompt for *what the model is*; use prompt for *what the model does now*. |

---

## 2. Single Agent vs Multi Agent

| Feature | Single Agent | Multi Agent | Key Takeaway |
|---|---|---|---|
| **Architecture** | One model instance handles everything | Multiple model instances, each with distinct roles | Single agent is a monolith; multi-agent is a team. |
| **Complexity** | Low — one prompt, one loop | High — coordination, handoffs, shared state | Complexity grows non-linearly with each added agent. |
| **Use cases** | Simple Q&A, single-file edit, structured extraction | Multi-step workflows, diverse tool ecosystems, parallel research | If one agent can do it, don't add a second. |
| **Debugging** | Easy — single trace | Hard — need to trace message flow between agents | Multi-agent failures are notoriously hard to reproduce. |
| **When to choose** | Task is self-contained within one domain | Task requires multiple domains or sequential specialized steps | Start single-agent. Split only when you hit clear boundaries. |

---

## 3. Tool Use vs MCP (Model Context Protocol)

| Feature | Tool Use | MCP | Key Takeaway |
|---|---|---|---|
| **Purpose** | Define a specific function the model can call | Standardized protocol to connect models to external tools/servers | Tool use is a capability; MCP is a *protocol* for managing capabilities. |
| **Integration** | Declared inline in the API request | External MCP server that registers tools dynamically | Tool use = hardcoded; MCP = pluggable. |
| **Flexibility** | Static at request time — all tools must be defined upfront | Dynamic — tools can be added/removed via server without API changes | MCP is better for evolving tool ecosystems. |
| **When to use** | Fixed, well-known tools (search, calculator, DB lookup) | Third-party integrations, multi-tool servers, shared tool registries | Use Tool Use for app-specific tools; MCP for reusable tool infrastructure. |
| **Maintenance** | Requires code changes to add/change tools | Server-level changes; no client code change needed | MCP decouples tool definition from application code. |

---

## 4. Claude Code vs API

| Feature | Claude Code | API | Key Takeaway |
|---|---|---|---|
| **Interface** | Terminal-based interactive agent | REST/HTTP interface | Claude Code is a product; API is an integration point. |
| **Primary use** | Developer daily workflow, coding, debugging | Application integration, production pipelines | Claude Code = human-in-the-loop; API = programmatic. |
| **CI/CD support** | Not designed for CI/CD — interactive | Fully headless — perfect for CI/CD | API is the only option for automated pipelines. |
| **Cost model** | Subscription (Pro/Team/Enterprise) | Per-token billing | For heavy automated usage, API is more cost-predictable. |
| **When to choose** | Interactive development, exploratory coding | Production integration, batch processing, automation | Use Claude Code as your pair programmer; API for everything else. |
| **File access** | Automatic — reads/writes to workspace | Must provide file content explicitly | Claude Code handles file I/O; API needs you to do it. |

---

## 5. Hooks vs Prompt Instructions

| Feature | Hooks | Prompt Instructions | Key Takeaway |
|---|---|---|---|
| **Nature** | Deterministic code that runs at specific lifecycle events | Probabilistic instructions in the system/user prompt | Hooks are *guaranteed* to run; instructions are *suggested*. |
| **When used** | Pre-processing, post-processing, validation, routing | Shaping model behavior, formatting output, guardrails | Use hooks for operations that *must* happen; instructions for *preferences*. |
| **Guarantee level** | 100% — code always executes | Non-deterministic — model may misinterpret or ignore | If failure is unacceptable → hook. If it's a suggestion → instruction. |
| **Examples** | Filter profanity before model sees it, validate JSON output | "Respond in JSON", "Be concise", "Use technical language" | Hooks enforce; instructions influence. |
| **Exam tip** | Hooks = security boundaries. Instructions = behavior shaping. | | Exam asks: "Which method guarantees X?" → Answer is hooks. |

---

## 6. Memory vs Context

| Feature | Memory | Context | Key Takeaway |
|---|---|---|---|
| **Persistence** | Survives across conversations | Only lives within a single conversation | Memory = long-term; Context = short-term. |
| **What's included** | Summarized or structured information from previous sessions | Full message history of the current conversation | Memory is *compressed* past; Context is the *current* conversation. |
| **How managed** | Explicit read/write via system prompt or API | Automatically accumulated as messages are added | Memory must be actively managed; context manages itself (until it overflows). |
| **Size limit** | Managed by application — fits in ~200K token window | Constrained by model's context window | Memory helps stay within context window. |
| **Use case** | User preferences, learned facts, long-term project knowledge | Current task, recent file content, immediate instructions | Memory = knowledge that outlasts the chat. |

---

## 7. JSON Mode vs Structured Output (via tool_use)

| Feature | JSON Mode | Structured Output (tool_use) | Key Takeaway |
|---|---|---|---|
| **Reliability** | Moderate — model may produce invalid JSON | High — tool_use grammar constrains output to valid JSON | Structured output is strictly more reliable. |
| **Syntax guarantee** | No — model can produce malformed JSON | Yes — tool_call arguments *must* be valid JSON | JSON mode still requires fallback parsing. |
| **Semantic guarantee** | No — values can be wrong types or out of range | Partial — types are enforced but semantic errors still possible | Neither guarantees correct *values*, only structure. |
| **Use case** | Simple extraction, logging, human-readable output | API calls, database writes, any code-dependent consumption | For programmatic consumption → always prefer Structured Output. |
| **Exam tip** | Structured Output is always preferred when the output must be consumed by code. | | "Guarantee valid JSON" = Structured Output. "JSON mode" = best-effort. |

---

## 8. Validation vs Retry

| Feature | Validation | Retry | Key Takeaway |
|---|---|---|---|
| **Purpose** | Check if output meets requirements | Re-run the model to get a corrected output | Validation identifies *if* something is wrong; retry fixes *that*. |
| **When used** | After model output is received | When validation fails | Validation is the gate; retry is the recovery mechanism. |
| **Relationship** | Validation triggers retry | Retry depends on validation to know if it succeeded | Retry without validation is blind. Validation without retry is useless. |
| **Cost** | Low (compute only) | High (additional model calls) | Ideally, reduce retry frequency by improving prompts. |
| **Loop control** | Validation alone doesn't loop | Must set max retry limit to avoid infinite loops | Always bound retries to a max count. |

---

## 9. Coordinator vs Worker Agent

| Feature | Coordinator | Worker Agent | Key Takeaway |
|---|---|---|---|
| **Role** | Plans, delegates, and synthesizes | Executes a specific subtask | Coordinator decides WHAT; worker does HOW. |
| **Responsibilities** | Task decomposition, result aggregation, error handling | Focused execution with specialized tools/knowledge | Coordinator manages the big picture; worker stays in its lane. |
| **Communication pattern** | Star pattern — coordinator talks to each worker | Responds to coordinator with results | Workers never talk to each other directly (in common pattern). |
| **Context** | Has full task context and overall goal | Receives only what coordinator passes | Workers are intentionally blind to the full picture — avoids distraction. |
| **When to use** | Complex multi-step tasks with diverse subtasks | Each subtask has a clear, bounded scope | Every multi-agent system needs a coordinator. Not every coordinator needs multiple workers. |

---

## 10. Parallel vs Sequential Execution

| Feature | Parallel | Sequential | Key Takeaway |
|---|---|---|---|
| **When to use** | Independent subtasks with no data dependency | Subtasks depend on output of a previous step | Parallel = independence. Sequential = dependency. |
| **Trade-off** | Faster wall-clock time but higher token cost | Slower but more accurate context preservation | Parallel saves time but loses shared state. |
| **Token implications** | Each branch pays full context cost | Shared context across steps reduces per-step cost | Parallel is token-heavy because each branch has no overlap. |
| **Error handling** | One branch failure may or may not affect others | One failure blocks all downstream steps | Sequential is fragile — one break stops the chain. |
| **Exam tip** | Exam asks: "Should subtasks A and B run in parallel?" → Check if B needs A's output. | | Never parallelize dependent subtasks. |

---

## 11. Stateless vs Stateful

| Feature | Stateless | Stateful | Key Takeaway |
|---|---|---|---|
| **API design** | Each request is self-contained; no conversation history | Requests build on prior exchanges; context accumulates | Stateless = each call starts fresh. Stateful = conversation continues. |
| **Persistence** | No memory of previous requests | Conversation history persisted | Stateless is simple; stateful is powerful but complex. |
| **Session management** | None — no session concept | Must manage session lifecycle (create, read, update, expire) | Stateful requires explicit session handling. |
| **When to use** | Single-turn tasks, batch processing, stateless functions | Multi-turn conversations, interactive agents, coding workflows | Stateless for automation; stateful for interaction. |
| **Scalability** | Trivially scalable — no shared state | Sessions need affinity or distributed state store | Stateless scales horizontally with no overhead. |
| **Cost** | Pay per request; no wasted context | May pay to persist and re-read context across turns | Stateless is cheaper for isolated tasks. |

---

## 12. Planning Mode vs Direct Execution

| Feature | Planning Mode | Direct Execution | Key Takeaway |
|---|---|---|---|
| **When to use** | Complex tasks, architectural decisions, multi-step problems | Simple tasks, well-known patterns, single-step operations | If you can describe the solution in one sentence → execute directly. |
| **Side effects** | No visible side effects — plan is just text | Real side effects — files change, APIs called | Planning is free. Execution costs tokens and has real impact. |
| **Complexity** | Adds overhead but reduces execution errors | Fast but risk of wrong approach due to missing big picture | Planning is an investment that pays off for complex tasks. |
| **Exam tip** | The exam emphasizes: *Plan first, then execute* for any non-trivial task. | | Starting with direct execution for a complex task is an anti-pattern. |
| **Debugging** | Easier — reasoning is visible in the plan | Harder — you see only the final (possibly wrong) output | Plans make reasoning inspectable. |

---

## 13. Synchronous API vs Batch API

| Feature | Synchronous API | Batch API | Key Takeaway |
|---|---|---|---|
| **Latency** | Seconds to minutes — real-time | Hours — queued and processed asynchronously | Sync is interactive; batch is deferred. |
| **Cost** | Standard per-token pricing | ~50% discount on token cost | Batch is significantly cheaper. |
| **Tool calling** | Full support — multi-turn tool use | No tool calling — single generation only | Batch cannot do tool-using workflows. |
| **Use cases** | Interactive chat, real-time agents, coding | Data processing, bulk evaluation, document summarization | Use Sync when you need a response now. Use Batch when you need volume at low cost. |
| **Streaming** | Supported | Not supported | Batch only returns complete results. |
| **When to choose** | User is waiting for the response | Throughput matters more than latency; no tool calling needed | If the task needs tool calls → must be Sync. |

---

## 14. `auto` vs `any` vs `forced` tool_choice

| Feature | `auto` | `any` | `forced` (specific tool name) | Key Takeaway |
|---|---|---|---|---|
| **Behavior** | Model decides whether to call a tool or respond with text | Model *must* call at least one tool, but chooses which | Model *must* call the specified tool | Auto = optional. Any = mandatory, flexible. Forced = mandatory, fixed. |
| **Use case** | General conversation with optional tool use | Workflow that needs a tool but multiple are valid | Extraction tasks, specific function calls | Use forced when you need exactly one tool every time. |
| **Guarantee** | No guarantee tool will be called | Guarantees a tool call (any valid tool) | Guarantees the exact tool is called | Forced = strongest guarantee about which tool. |
| **When to choose** | Open-ended tasks where model should use judgment | Multi-tool workflows where model picks best tool | Known-pattern extraction (e.g., "extract this into JSON") | If you know *which* tool, don't use `auto` or `any`. |
| **Exam tip** | If the task says "always extract to JSON" → forced. If "use tools when needed" → auto. | | Wrong answers conflate `any` (must call *a* tool) with `forced` (must call *this* tool). |

---

## 15. User-level vs Project-level vs Directory-level CLAUDE.md

| Feature | User-level | Project-level | Directory-level | Key Takeaway |
|---|---|---|---|---|
| **Location** | `~/.claude/CLAUDE.md` | `.claude/CLAUDE.md` | `subdir/.claude/CLAUDE.md` | Closer to the file = higher priority. |
| **Scope** | All projects across the system | Single project root | Specific subdirectory within a project | User-level = personal preferences. Project-level = team conventions. Directory-level = module-specific rules. |
| **VCS (Git)** | No — sits outside repository | Yes — part of project repo | Yes — part of project repo | Only user-level config is personal; project/directory-level is shared. |
| **Use cases** | Editor preferences, personal conventions, API keys reference | Coding standards, architecture decisions, tool definitions | Module-specific patterns, file-type conventions | Team conventions live in VCS. Personal settings live in home dir. |
| **Priority** | Lowest (broadest scope) | Medium | Highest (most specific) | More specific path = more specific rules. |

---

## 16. `.claude/commands/` vs `.claude/skills/`

| Feature | `.claude/commands/` | `.claude/skills/` | Key Takeaway |
|---|---|---|---|
| **Format** | Custom JSON files with name, description, command | Custom JSON files with name, description, system prompt | Commands are *code*; skills are *instructions*. |
| **Features** | Execute arbitrary shell commands on trigger | Inject system prompt instructions on demand | Commands do things. Skills teach things. |
| **Which is current** | Legacy — deprecated in favor of skills | Current — preferred approach | If building new, use skills. Commands still work but not recommended. |
| **Trigger** | User types `/command_name` | User types `/skill_name` | Both triggered by slash. |
| **When to use** | Only for backward compatibility | All new agent capabilities | The exam may test that skills replaced commands as the recommended pattern. |

---

## 17. Transient vs Validation vs Business vs Permission Errors

| Feature | Transient Error | Validation Error | Business Error | Permission Error |
|---|---|---|---|---|
| **Retryable?** | Yes — will likely succeed on retry | No — same input will fail again | No — same input will fail again | No — same input will fail again |
| **Action** | Retry with backoff | Fix input | Fix business logic | Fix auth/credentials |
| **Examples** | Network timeout, rate limit, service temporarily unavailable | Malformed JSON, missing required field, wrong type | Insufficient funds, item out of stock, duplicate entry | Invalid API key, expired token, unauthorized role |
| **Who fixes** | System (automatic) | Developer/user | Business logic/domain rules | Administrator/user |
| **Exam tip** | Only transient errors should trigger automatic retry. The other three need human/code intervention. | | Wrong answer: "Retry all errors." Correct: "Retry only transient errors." |

---

## 18. Fixed Pipeline vs Dynamic Decomposition

| Feature | Fixed Pipeline | Dynamic Decomposition | Key Takeaway |
|---|---|---|---|
| **Predictability** | High — same steps every time | Low — steps vary per input | Fixed = known path. Dynamic = adapts to input. |
| **Flexibility** | Low — can't handle edge cases beyond the fixed path | High — adapts to each task | Fixed is brittle; dynamic is resilient. |
| **Use cases** | Known, repeatable workflows (e.g., data ETL pipeline) | Open-ended tasks (e.g., software development, research) | Use fixed when you know all the steps. Use dynamic when you don't. |
| **Error handling** | Simple — each step has known expected output | Complex — error recovery depends on where in decomposition | Dynamic systems need more sophisticated error handling. |
| **Implementation** | Simple — just a sequence of calls | Complex — needs a coordinator agent | Fixed = code. Dynamic = agent orchestration. |
| **Exam tip** | The exam favors dynamic decomposition for agentic workflows that need to adapt. | | "Unknown steps at design time" → Dynamic decomposition is required. |

---

## Quick Memory Table

| # | Pair | One-Liner Mnemonic |
|---|---|---|
| 1 | Prompt vs System Prompt | "Stage (system) vs actor's line (prompt)" |
| 2 | Single vs Multi Agent | "One brain vs a meeting" |
| 3 | Tool Use vs MCP | "Hardcoded vs pluggable" |
| 4 | Claude Code vs API | "Pair programmer vs automation engine" |
| 5 | Hooks vs Instructions | "Guaranteed vs suggested" |
| 6 | Memory vs Context | "Long-term vs short-term" |
| 7 | JSON vs Structured Output | "Best-effort vs guaranteed syntax" |
| 8 | Validation vs Retry | "Gate vs recovery" |
| 9 | Coordinator vs Worker | "Manager vs doer" |
| 10 | Parallel vs Sequential | "Independence vs dependency" |
| 11 | Stateless vs Stateful | "Each time fresh vs ongoing conversation" |
| 12 | Planning vs Direct | "Think first vs act now" |
| 13 | Sync vs Batch | "Real-time vs deferred, cheap" |
| 14 | auto vs any vs forced | "Maybe vs some vs this one" |
| 15 | CLAUDE.md levels | "Personal vs team vs module" |
| 16 | Commands vs Skills | "Legacy vs current" |
| 17 | Error types | "Transient = retry, rest = fix" |
| 18 | Fixed vs Dynamic | "Known path vs adaptive path" |

---

*Last updated for the Claude Certified Architect Foundations exam.*
