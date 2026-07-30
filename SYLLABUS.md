# Claude Certified Architect Foundations (CCA-F / CCAR-F) — Syllabus

## What This Exam Tests

This exam tests your ability to **build production AI systems with Claude**. It is NOT about theory or memorization. It is about making correct design decisions when building agents, integrating tools, configuring Claude Code, engineering prompts, and managing context.

The exam has **5 domains** and is **scenario-based** (you get realistic situations and must pick the best solution).

---

## Domain 1: Agent Architecture and Orchestration (27% — Biggest Domain)

This is the foundation of everything. Master this = 1/4 of the exam.

### What You Will Learn

| Topic | What It Means |
|-------|---------------|
| **Agent Loop Lifecycle** | The repeating cycle: send request → Claude responds → check stop_reason → if tool_use, run tool and repeat → if end_turn, done |
| **stop_reason values** | `end_turn` = finished (stop), `tool_use` = wants to call a tool (continue), `max_tokens` = hit limit (increase it), `stop_sequence` = custom match |
| **Anti-patterns** | What NOT to do: parsing Claude's text for completion signals, using arbitrary iteration limits as the main stop condition |
| **Hub-and-Spoke Architecture** | One coordinator agent that delegates work to specialized subagents (search, analysis, synthesis). All communication flows through coordinator. |
| **Coordinator Responsibilities** | Breaks task into pieces, assigns to subagents, combines results, handles errors |
| **Subagent Context Isolation** | Subagents do NOT see each other's conversations. You must explicitly pass context. They don't inherit parent history. |
| **Task Tool** | The mechanism for spawning subagents. Coordinator's allowedTools must include "Task" (or "Agent"). |
| **Parallel Spawning** | Multiple Task calls in one response = subagents run in parallel |
| **AgentDefinition Configuration** | How you define an agent: name, description, system prompt, allowed tools |
| **Programmatic Enforcement vs Prompt Guidance** | Code-level guards (hooks) are 100% reliable. Prompt instructions are 90%+ but can fail. Use hooks for money/security/compliance. |
| **PostToolUse Hook** | Runs after tool executes. Used to normalize data formats, trim verbose outputs, transform results. |
| **PreToolUse Hook** | Runs before tool executes. Used to block policy violations before they happen. |
| **Deterministic vs Probabilistic** | Code = always enforced. Prompt = may fail. Choose code when failure has consequences. |
| **Fixed Pipelines (Prompt Chaining)** | Predetermined sequence of steps. Good for predictable workflows like code review. |
| **Dynamic Adaptive Decomposition** | Model decides how to break down the task as it goes. Good for open-ended investigations. |
| **Multi-pass Code Review** | Review each file separately for local issues, then run an integration pass for cross-file issues. |
| **Session Resume (--resume)** | Continue a previous session with full context restored. |
| **fork_session** | Create independent branches from shared context to compare approaches. |
| **New Session vs Resume Decision** | Resume if context is still valid. Start fresh if tool results are stale (files changed). |

---

## Domain 2: Tool Design and MCP Integration (18%)

### What You Will Learn

| Topic | What It Means |
|-------|---------------|
| **Tool Descriptions = Selection Mechanism** | The model reads descriptions to decide which tool to call. Bad descriptions = wrong tool choices. |
| **Writing Good Descriptions** | Include: what it does, input formats, example values, edge cases, when to use it vs similar tools |
| **Avoiding Overlapping Descriptions** | If two tools sound the same, model will confuse them. Make each tool's purpose unique. |
| **isError Flag (MCP)** | Marks a tool response as a failure so the agent knows it's an error, not data. |
| **Error Categories** | Transient (timeout) = retry. Validation (bad input) = fix request. Business (policy violation) = explain. Permission (access denied) = escalate. |
| **Retryable vs Non-retryable** | Distinguish errors that can be fixed by retrying vs errors that need a different approach. |
| **Generic vs Structured Errors** | "Operation failed" gives agent no info. Error type + category + details + isRetryable lets agent decide correctly. |
| **Too Many Tools Reduces Reliability** | More tools = more confusion. Limit each agent to 4-5 role-relevant tools. |
| **Principle of Least Privilege** | Give each agent only the tools it needs, nothing more. |
| **tool_choice: auto** | Default. Model decides whether to use a tool or respond in text. |
| **tool_choice: any** | Model MUST call a tool (but picks which one). Use when you need guaranteed structured output. |
| **tool_choice: forced** | Model MUST call a specific tool. Use to enforce execution order. |
| **Replacing General Tools with Constrained Alternatives** | Instead of a generic `fetch_url`, use `load_document` that only accepts document URLs. |
| **MCP Server Configuration** | `.mcp.json` = project scope, version-controlled. `~/.claude.json` = personal, not shared. |
| **Environment Variable Substitution** | `${GITHUB_TOKEN}` in `.mcp.json` keeps secrets out of version control. |
| **Community vs Custom MCP Servers** | Prefer existing servers for standard integrations (Jira, GitHub). Build custom only for unique needs. |
| **MCP Resources** | Read-only data (catalogs, schemas, docs) that agents can browse without making tool calls. |
| **Built-in Tools** | Read (read files), Write (create files), Edit (precise changes), Bash (run commands), Grep (search file contents), Glob (find files by name) |

---

## Domain 3: Claude Code Configuration and Workflows (20%)

### What You Will Learn

| Topic | What It Means |
|-------|---------------|
| **CLAUDE.md Hierarchy (4 Levels)** | (1) Managed Policy — company-wide, IT-deployed. (2) User — `~/.claude/CLAUDE.md`, personal only. (3) Project — `.claude/CLAUDE.md`, team, version-controlled. (4) Local — `CLAUDE.local.md`, personal per project, add to .gitignore. |
| **@path Syntax** | Import external files into CLAUDE.md using `@filename`. Max 4 hops deep. |
| **.claude/rules/ Directory** | Topic-focused files (testing.md, api-conventions.md) instead of one huge CLAUDE.md |
| **Rules with YAML Frontmatter** | Add `paths: ["src/api/**/*"]` to load rules only when working on matching files. Saves context. |
| **Custom Slash Commands** | `.claude/commands/` = team commands, version-controlled. `~/.claude/commands/` = personal. |
| **Skills (.claude/skills/)** | Packaged capabilities with SKILL.md + frontmatter. More powerful than commands. |
| **SKILL.md Frontmatter** | `context: fork` = runs in isolated subagent. `allowed-tools` = restricts tools. `argument-hint` = prompts for parameters. |
| **Personal Skills Override Project Skills** | Same name in `~/.claude/skills/` beats `.claude/skills/`. |
| **Planning Mode** | Explore code, make plan, NO changes. Use for big/complex/architectural tasks. |
| **Direct Execution** | Just do it. Use for simple, well-understood changes. |
| **Explore Subagent** | Isolate verbose discovery output in a subagent. Main session gets only summary. |
| **Iterative Refinement** | Show concrete input/output examples to guide Claude. Write tests first, iterate until they pass. |
| **Interview Pattern** | Claude asks clarifying questions before implementing, especially for unfamiliar domains. |
| **/compact Command** | Compresses conversation history to free context mid-session. Risk: numbers/dates get vague. |
| **/memory Command** | Opens memory files for editing. Persists across sessions. |
| **CI/CD with -p Flag** | `claude -p "prompt"` runs non-interactively. Only way to use Claude Code in automated pipelines. |
| **--output-format json** | Get structured JSON output for programmatic parsing. |
| **--json-schema** | Enforce a specific JSON schema on the output. |
| **Session Context Isolation for Review** | Same session that generated code is biased when reviewing it. Use independent instance. |
| **Batch API** | 50% cheaper. Up to 24 hours. No multi-turn tool calling. Use for overnight/weekly workloads, NOT for blocking checks. |

---

## Domain 4: Prompt Engineering and Structured Output (20%)

### What You Will Learn

| Topic | What It Means |
|-------|---------------|
| **Explicit Criteria vs Vague Instructions** | "Flag only if code contradicts comment" works. "Check comments" doesn't. Be specific. |
| **False Positive Impact on Trust** | Too many wrong findings = developers ignore everything, even correct ones. |
| **Severity Criteria with Examples** | CRITICAL = runtime failure (null pointer in payment). HIGH = security (SQL injection). MEDIUM = logic bug (wrong sort). LOW = code quality (duplication). |
| **Few-shot > Textual Descriptions** | 2-4 concrete examples beat 10 paragraphs of instructions. Model copies the pattern. |
| **Few-shot for Ambiguous Scenarios** | Show model how to handle edge cases: "If customer says 'broken' → check order first. If customer says 'manager' → escalate immediately." |
| **Few-shot for Output Formatting** | Show exact JSON structure the model should produce. |
| **Few-shot: Acceptable vs Problematic** | Show both good (do not flag) and bad (flag this) examples so model learns the boundary. |
| **JSON Schemas Fix Syntax, NOT Semantics** | Schema guarantees valid JSON but NOT correct values. Totals can still be wrong. |
| **Required vs Optional Fields** | Required fields + missing data = model fabricates values. Use optional/nullable when data may be absent. |
| **Nullable + Enum with "other"/"unclear"** | Let model return null when info missing. Add "other" + detail field so unknown categories aren't forced into wrong buckets. |
| **Retry-with-Error-Feedback** | If validation fails, retry with: original document + failed extraction + specific error. Model can self-correct. |
| **When Retry Won't Help** | If required info is simply absent from source document, no amount of retrying will produce it. |
| **Self-Correction Pattern** | Ask model to extract both stated_total and calculated_total. If they differ, flag conflict. |
| **Pydantic for Validation** | Python library that checks structure (types, required fields) AND semantics (sums match, dates make sense). |
| **Synchronous API for Blocking Checks** | Real-time validation. Use when developers are waiting for results. |
| **Batch API for Non-blocking Workloads** | Overnight reports, weekly audits. 50% cheaper but up to 24hr delay. |
| **custom_id for Failure Handling** | Tag each batch request. When some fail, resubmit only the failed ones by ID. |
| **SLA Planning** | Choose sync vs batch based on deadline. If deadline is 30hrs away and batch can take 24hrs, submit within 6hrs. |
| **Independent Review Avoids Confirmation Bias** | Generation session rationalizes its own decisions. Independent review catches what generation missed. |
| **Multi-pass Review for Large PRs** | Break into focused passes: security → logic → style. Each pass has one job. |

---

## Domain 5: Context Management and Reliability (15%)

### What You Will Learn

| Topic | What It Means |
|-------|---------------|
| **Progressive Summarization Risks** | Numbers, dates, exact amounts get lost when summarizing. "15% discount" becomes "some discount." |
| **Lost-in-the-Middle Effect** | Model remembers start and end of long text but misses middle. Put key info at beginning or end. |
| **Extract Facts into Separate Block** | Pull critical data (order IDs, amounts, dates) into a structured "case facts" block outside the summarized history. |
| **Trim Verbose Tool Outputs** | If tool returns 40 fields but only 5 matter, trim the rest. Saves context weight. |
| **Escalation Triggers** | (1) Customer asks for human — escalate immediately. (2) Policy doesn't cover the situation — escalate. (3) Agent is stuck — escalate after reasonable attempts. |
| **Immediate vs Attempt-then-Escalate** | Explicit "get me a manager" = escalate now. Standard problem = try to resolve first, escalate if unsuccessful. |
| **Ask for Additional Identifiers** | When search returns multiple matches, ask customer for more info. Do NOT guess. |
| **Unreliable Escalation Signals** | Sentiment analysis (mood ≠ complexity), self-rated confidence (model can be confidently wrong). |
| **Structured Error Context** | Pass error details (type, what was attempted, partial results, alternatives) to the coordinator so it can decide what to do. |
| **Access Failure vs Valid Empty Result** | Timeout = retry decision. "0 results" = valid finding, no retry needed. Don't confuse them. |
| **Local Recovery in Subagents** | Let subagents retry their own errors before escalating. Saves coordinator from handling every hiccup. |
| **Coverage Annotations** | Mark which parts of a task have good coverage and which have gaps. |
| **Scratchpad Files** | Save key findings to a file during long sessions. When context degrades, reference the file instead of re-discovering. |
| **Delegating to Subagents Frees Context** | Each subagent has its own context. Main agent keeps only summary. |
| **Context Degradation Signs** | Model repeats itself, misses details, makes contradictory statements, answers from "typical patterns" instead of specifics. |
| **Aggregate Metrics Can Lie** | 97% overall accuracy may hide 40% error rate on specific document types. Always segment by type and field. |
| **Stratified Random Sampling** | Test across all categories, not just the easy ones. Ensures you catch problems in minority types. |
| **Field-Level Confidence Scores** | Report confidence per field, not just overall. Route low-confidence fields to human review. |
| **Claim→Source Mapping** | Always track which source produced each claim. Otherwise attribution is lost during summarization. |
| **Handling Conflicting Data** | Don't silently pick one value. Present both with source attribution and let the coordinator decide. |
| **Include Dates for Temporal Context** | "Source A says 10% (2023)" vs "Source B says 15% (2024)" = growth, not contradiction. |
| **Render by Content Type** | Financial data = tables. News = prose. Technical findings = structured lists. Match format to data. |

---

## What Is NOT Covered (Out of Scope)

These topics will **NOT** appear on the exam and are **NOT** in this study material:

| Not Tested | Why |
|------------|-----|
| Fine-tuning models | Not relevant for architect role |
| API authentication / billing | Operational details, not architecture |
| Specific programming language implementation | Beyond tool/schema config |
| Deploying/hosting MCP servers | Infrastructure, not architecture |
| Claude's internal architecture or training | Not needed for builders |
| Constitutional AI / RLHF | Safety training methodology |
| Embedding models / vector databases | Separate topic |
| Computer use / browser automation | Different use case |
| Image analysis / Vision | Different capability |
| Streaming API | Implementation detail |
| Rate limiting / quotas | Operational |
| OAuth / API key rotation | Authentication details |
| Cloud provider specifics (AWS, GCP, Azure) | Infrastructure |
| Performance benchmarks | Not tested |
| Token counting algorithms | Implementation detail |

---

## Study Materials Available

| Material | What It Is |
|----------|-----------|
| **Domain Notes (5 files)** | Full study notes for each domain with definitions, analogies, examples, exam tricks, and diagrams |
| **Flashcards (315)** | Question-answer pairs with domain tags and exam tricks |
| **MCQs (165)** | Practice questions with detailed explanations of why right is right and wrong is wrong |
| **Scenario Questions (30)** | Long-form architecture scenarios requiring reasoning |
| **Mock Exam (60 questions)** | Full timed practice exam matching real format |
| **Comparison Tables (18)** | Side-by-side comparisons of related concepts |
| **Exam Traps (30)** | Common wrong answers and why students pick them |
| **Cheat Sheets (2)** | One-page overall reference + 5 one-page domain cards |
| **Architecture Diagrams (16)** | Visual representations of key concepts |
| **Glossary (72 terms)** | All technical terms defined in plain English |
| **Revision Guide** | Tiered revision: 5 min to night-before |

---

## Study Order (Recommended)

1. **Domain 1** → Heaviest weight (27%), foundational concepts
2. **Domain 2** → Builds on Domain 1 (tools are called by agents)
3. **Domain 4** → Independent, medium weight
4. **Domain 3** → Practical configuration
5. **Domain 5** → Lowest weight, save for last
6. **Cheat Sheets + Exam Traps** → Consolidate
7. **MCQs + Mock Exam** → Test knowledge
8. **Revision** → Final pass
