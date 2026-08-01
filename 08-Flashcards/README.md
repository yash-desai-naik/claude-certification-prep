# CCA-F Exam Flashcards — 300+ Questions

Answer-first format. Each card tests **understanding**, not recall. Trick section explains why students make mistakes.

---

## Domain 1: Agent Architecture and Orchestration (27%)

### 1.1 — Agent Loop Lifecycle

### #1
**Front:** Your agent calls a search tool and receives results. The API response shows `stop_reason: "tool_use"`. What should the orchestrator do next?
**Back:** Execute the tool (search), append the result to conversation history, and send the next request to the model. `tool_use` means the model expects result feedback. The loop continues.
**Domain:** 1
**Trick:** Students think `tool_use` means "done with tool" and try to end the loop. `tool_use` means the model **wants** to use a tool — you must run it and return results.

### #2
**Front:** After executing a tool and appending the result, your code checks `stop_reason` and finds `"end_turn"`. A human reviewer says the agent's response looks incomplete. Who is right?
**Back:** The code is right. `stop_reason: end_turn` is the only reliable signal the agent is done. The agent may have determined it gave sufficient answer, even if a human disagrees. Never override `end_turn` based on subjective completeness.
**Domain:** 1
**Trick:** Students trust human judgment over formal signals. `end_turn` is authoritative — the model decided it's done.

### #3
**Front:** An agent loop runs for 12 iterations. The developer sets `max_iterations: 5` to prevent runaway costs. After 5 iterations, the loop terminates mid-analysis. What is the design flaw?
**Back:** Arbitrary iteration limits are an anti-pattern — they terminate agents mid-task. Use `stop_reason === "end_turn"` as the termination condition, not a hardcoded iteration ceiling. If cost control is needed, use token budgets instead.
**Domain:** 1
**Trick:** Students defend arbitrary limits as "safety measures." The exam says: let the model decide when it's done. Hard limits break workflows.

### #4
**Front:** Your agent's output parser scans the model's text response for the string "Task complete" before terminating the loop. Another developer says this is wrong. Why?
**Back:** Parsing assistant text for completion is an anti-pattern. The model may say "Task complete" while still needing another tool call, or it may complete the task without saying those exact words. Always rely on `stop_reason === "end_turn"`.
**Domain:** 1
**Trick:** Text parsing seems logical but is unreliable. The model's conversational text and its actual completion state are independent.

### #5
**Front:** In the agent loop lifecycle, after you append a tool result and send the next request, what determines whether the loop continues or stops?
**Back:** The model's next `stop_reason`. If `tool_use` → continue loop (execute tool, return result, repeat). If `end_turn` → stop loop, return final response. The model decides each iteration.
**Domain:** 1
**Trick:** Developers assume their code controls the loop termination. The model controls termination via `stop_reason`.

### #6
**Front:** You're building an agent that processes insurance claims. The first step must be identity verification. A colleague suggests giving the agent all tools and letting it decide the order. Is this correct?
**Back:** No — when a strict workflow is required (e.g., verify identity before processing), use deterministic enforcement (code-level guards) or a fixed pipeline. Model-driven flexibility is for open-ended tasks, not regulated sequential workflows.
**Domain:** 1
**Trick:** Students over-apply model-driven decision making. Fixed pipelines are correct when order is mandatory.

### #7
**Front:** An agent sends a request, and the API returns `stop_reason: "max_tokens"`. The output is cut off mid-sentence. What should the orchestrator do?
**Back:** This means output was truncated. The model did not finish its response. Typically, you should increase `max_tokens` or retry. Do NOT treat this as completion or append partial output as a final answer.
**Domain:** 1
**Trick:** Students treat any response as valid output. `max_tokens` truncation means the model's reasoning is incomplete.

### #8
**Front:** Your agent calls a weather API tool and gets back `{ status: 500, error: "Service unavailable" }`. The `stop_reason` is still `tool_use`. What happens in the loop?
**Back:** The tool result (including the error) is appended to conversation history as a normal tool result. The model sees the failure and decides what to do — retry, use a different tool, or apologize and end turn. The loop continues until `end_turn`.
**Domain:** 1
**Trick:** Students think errors should break the loop. Errors are just tool results — the model decides how to handle them.

### #9
**Front:** A student claims "You can detect agent completion by checking if the model says 'I have finished' or 'Here is my final answer'." Why is this approach flawed?
**Back:** The model may say "I have finished" but still produce a `stop_reason: tool_use` because it plans to call one more tool. Conversely, it may end turn without any closing phrase. Text parsing + stop_reason checks are redundant — only `end_turn` matters.
**Domain:** 1
**Trick:** Natural language markers are unreliable. The model's conversational output and its internal completion signal are not synchronized.

### #10
**Front:** You deploy a customer support agent. During testing, it loops 20+ times on a single query. Your SRE team wants to set `max_iterations: 10`. How should you respond?
**Back:** Instead of an arbitrary iteration limit that may terminate mid-task, implement a token budget or timeout guard. If the agent exceeds reasonable consumption, escalate to a human or log for review. But during normal operation, let `end_turn` be the only termination signal.
**Domain:** 1
**Trick:** SRE-driven safety limits sound responsible but are an anti-pattern when applied as hard iteration caps.

---

### 1.2 — stop_reason Values

### #11
**Front:** During a loop iteration, the API returns `stop_reason: "stop_sequence"`. The developer has never seen this value. How should it be handled?
**Back:** `stop_sequence` is an edge case that occurs when the output hits a custom stop sequence you defined. Treat it similar to `end_turn` — the model stopped because it hit a trigger you set. Verify the output is complete before returning.
**Domain:** 1
**Trick:** Students panic at unfamiliar `stop_reason` values. `stop_sequence` means your own custom stop condition was met.

### #12
**Front:** An exam question shows four `stop_reason` values: `end_turn`, `tool_use`, `max_tokens`, `stop_sequence`. Which two indicate the model has NOT finished its response?
**Back:** `tool_use` (model wants to call a tool and continue) and `max_tokens` (output was truncated). Only `end_turn` and `stop_sequence` (edge case) indicate the model has finished delivering its response.
**Domain:** 1
**Trick:** Students group `tool_use` with completion states. `tool_use` means "pause — I need data to continue."

### #13
**Front:** You send a message to Claude with no tool definitions. What `stop_reason` values can you expect in the response?
**Back:** Without tools, Claude will return `end_turn` (normal completion) or `max_tokens` (truncation). `tool_use` is only possible when tool definitions are provided.
**Domain:** 1
**Trick:** Students forget that `tool_use` requires tool definitions to be present in the request.

---

### 1.3 — Tool Use vs End Turn

### #14
**Front:** Scenario: An agent is writing code. It calls `Read` to examine a file, then `Write` to make changes. After `Write`, `stop_reason` is `end_turn`. Should the orchestrator verify the file was written correctly before returning?
**Back:** No. If `stop_reason` is `end_turn`, the model decided it's done. The orchestrator should return the response. If you want verification, add a tool that performs automated verification after write and let the model decide to use it before ending turn.
**Domain:** 1
**Trick:** Students add post-hoc verification outside the loop. Verification should be a tool the model can call before ending turn.

### #15
**Front:** Your code review agent reviews a PR and returns `stop_reason: tool_use` with a `read_file` tool call. What is happening?
**Back:** The model is not done reviewing — it needs to read another file before it can form its review. Execute `read_file`, return the content, and let the model continue. Only when `stop_reason` switches to `end_turn` is the review complete.
**Domain:** 1
**Trick:** Students think the first `tool_use` after a response is a "final check." It's part of the ongoing analysis.

---

### 1.4 — Agent Loop Anti-Patterns

### #16
**Front:** Your agent loop has this condition: `while (stop_reason !== "end_turn" && iterations < 15)`. A PR reviewer flags this as problematic. Why?
**Back:** The `iterations < 15` guard is an arbitrary max-iterations anti-pattern. If the agent legitimately needs 16 iterations, it will be cut off mid-task. Use `stop_reason === "end_turn"` alone, with a token budget as a safety net, not an iteration cap.
**Domain:** 1
**Trick:** Combined guards feel safer but introduce the same problem: premature termination.

### #17
**Front:** A developer writes: `if (response.text.includes("Goodbye") || stop_reason === "end_turn") { break; }`. What anti-pattern is present?
**Back:** Parsing assistant text for completion. The model may say "Goodbye" but still have `stop_reason: tool_use` pending. Text scanning is unreliable — only check `stop_reason`.
**Domain:** 1
**Trick:** The OR condition makes it seem robust, but text matching is the flaw.

### #18
**Front:** A team builds an agent that calls exactly 3 tools per task, then returns the accumulated result regardless of `stop_reason`. What happens when the agent needs only 1 tool?
**Back:** The agent will be forced to call 2 extra unnecessary tools (if available) or error out. Fixed tool counts ignore the model's actual needs. Some tasks need 1 tool, others need 10. Let the model decide.
**Domain:** 1
**Trick:** Developers think uniform workflows are simpler. The model determines tool count, not the developer.

### #19
**Front:** After a tool returns an error, your orchestrator automatically retries the same tool 3 times before giving up. Why might this be problematic?
**Back:** The orchestrator is preempting the model's decision-making. The model should see the error and decide whether to retry, try a different tool, or change approach. The orchestrator should only execute what the model requests, not implement its own retry logic on top.
**Domain:** 1
**Trick:** Auto-retry feels helpful but violates model-driven design. Let the model make retry decisions.

---

### 1.5 — Hub-and-Spoke Architecture

### #20
**Front:** An exam question describes a system with a central agent that delegates sub-tasks to worker agents. What is this pattern called?
**Back:** Hub-and-spoke (coordinator-subagent) architecture. One coordinator handles decomposition (splitting the task), delegation (assigning to subagents), and aggregation (combining results).
**Domain:** 1
**Trick:** Students confuse this with peer-to-peer or pipeline architectures. Hub-and-spoke has a single coordinator managing multiple workers.

### #21
**Front:** In a hub-and-spoke architecture, a subagent completes its task but the coordinator doesn't know how to integrate the result. Which coordinator responsibility failed?
**Back:** Aggregation — the coordinator's job of combining subagent outputs into a coherent final result. The coordinator must have enough context and instructions to assemble individual contributions.
**Domain:** 1
**Trick:** Students think only delegation matters. Aggregation is equally critical and often more complex.

### #22
**Front:** Your coordinator agent receives a complex task: "Analyze this 500-page codebase for security vulnerabilities." The coordinator must decide how to split the work. What is this responsibility called?
**Back:** Decomposition — breaking a complex task into smaller, manageable sub-tasks that can be delegated to specialized subagents.
**Domain:** 1
**Trick:** Students confuse decomposition (splitting the task) with delegation (assigning the pieces).

### #23
**Front:** Subagent A encounters an unexpected error. Instead of crashing the entire workflow, the coordinator routes around the failure and adjusts remaining sub-tasks. What coordinator responsibility is demonstrated?
**Back:** Error handling — the coordinator's ability to catch, recover from, or work around failures in subagents without aborting the entire workflow.
**Domain:** 1
**Trick:** Students design brittle systems where one subagent failure crashes everything. Error handling means graceful recovery.

### #24
**Front:** A hub-and-spoke system has 10 subagents. Each subagent is tasked with analyzing one file. The coordinator must produce a final report. An engineer suggests making subagents report directly to each other to save a round trip. Why is this wrong?
**Back:** Subagents have isolated context — they don't see each other's conversations. Direct communication between subagents is not supported in hub-and-spoke. All communication flows through the coordinator, which aggregates results.
**Domain:** 1
**Trick:** Peer-to-peer communication between subagents seems efficient but violates the architecture's isolation model.

---

### 1.6 — Coordinator Responsibilities

### #25
**Front:** A coordinator receives analysis results from 5 subagents. The results contain conflicting findings about a pricing calculation. What should the coordinator do?
**Back:** The coordinator should either: (1) present both perspectives with attribution, or (2) request additional analysis from subagents to resolve the conflict. It should NOT silently pick one result or average them without justification.
**Domain:** 1
**Trick:** Students think the coordinator's job is to pick a winner. The coordinator should flag conflicts and seek resolution.

### #26
**Front:** You're designing a coordinator for a triage system. The coordinator must decide whether a query is supported by existing documentation or needs escalation. What pattern should it follow?
**Back:** Use model-driven decision making: give the coordinator tools (search docs, check policies, escalate to human) and let it decide the sequence. Don't hardcode if-else rules for triage paths.
**Domain:** 1
**Trick:** Triage seems like a perfect if-else candidate, but edge cases make hardcoding fragile.

### #27
**Front:** Three subagents return results. Subagent 2's output is clearly wrong (contradicts the source data). What should the coordinator do?
**Back:** The coordinator can re-delegate the sub-task to Subagent 2 with feedback about what went wrong, or delegate to a different subagent for a fresh attempt. It should not silently discard Subagent 2's output without logging the failure.
**Domain:** 1
**Trick:** Silently ignoring bad subagent outputs misses an opportunity to improve the system.

---

### 1.7 — Subagent Context Isolation

### #28
**Front:** Subagent A is analyzing user authentication code. Subagent B is analyzing payment processing code. You want Subagent B to reference a finding from Subagent A's session. How should this work?
**Back:** Subagent B cannot see Subagent A's conversation. The coordinator must explicitly pass Subagent A's findings to Subagent B as part of Subagent B's context/instructions. Subagents never inherit or share context.
**Domain:** 1
**Trick:** "Subagents share parent context" is a common wrong answer. They are fully isolated — you must pass context explicitly.

### #29
**Front:** A developer complains: "I spawned 3 subagents and put important audit instructions in the coordinator's system prompt. The subagents keep violating those rules." What went wrong?
**Back:** Subagents do NOT inherit coordinator context or system prompts. Each subagent needs its own instructions explicitly included in its task definition. Audit rules must be passed to each subagent individually.
**Domain:** 1
**Trick:** Students assume context flows from parent to child. It does not — explicit context passing is required.

### #30
**Front:** An exam question: A subagent completes its task and returns a massive result (10k tokens). The coordinator's context window is 8k tokens. What problem occurs?
**Back:** The coordinator may not be able to fit the full subagent output in context, leading to the lost-in-the-middle effect or truncation. Subagent outputs should be summarized or structured before being returned to the coordinator.
**Domain:** 1
**Trick:** Students focus on getting the result but ignore context budget for the receiving agent.

---

### 1.8 — Task Tool Spawning

### #31
**Front:** What is the Task tool in Claude Code and what is its primary purpose?
**Back:** The Task tool is the primary mechanism for spawning subagents from Claude Code. It creates an isolated subagent with its own context, tools, and instructions to work on a delegated sub-task.
**Domain:** 1
**Trick:** Students confuse Task tool with function calling or regular tool use. It specifically spawns an independent agent.

### #32
**Front:** You need a subagent to search through 200 files for security issues. You use the Task tool without specifying any tool constraints for the subagent. What risk do you face?
**Back:** The subagent has access to all default tools (Read, Write, Edit, Bash, Grep, Glob), which violates least privilege. You should constrain the subagent's tools to only those needed for its task (e.g., just Read and Grep for a search-only task).
**Domain:** 1
**Trick:** Students forget that subagents get full tool access by default. Constraining tools is the developer's responsibility.

### #33
**Front:** A subagent needs to reference a specific known IP address from the coordinator's analysis. How should the coordinator provide this information?
**Back:** The IP address must be explicitly included in the subagent's task instructions or as part of the context passed to the Task tool. The subagent has no access to the coordinator's conversation history.
**Domain:** 1
**Trick:** "Subagent can read coordinator's files" is wrong. Subagents are isolated — all data must be passed explicitly.

---

### 1.9 — Parallel Subagent Spawning

### #34
**Front:** You have 10 independent files to analyze — each analysis has no dependency on the others. Your developer spawns them sequentially with a single Task call per subagent. What optimization is being missed?
**Back:** Multiple Task calls can run in parallel. Since the analyses are independent, you should spawn all 10 subagents concurrently to reduce total execution time from sequential to the time of the longest analysis.
**Domain:** 1
**Trick:** Sequential spawning is simpler but wastes parallelism. Independent tasks should run concurrently.

### #35
**Front:** Three subagents must run in sequence: B depends on A's output, C depends on B's output. Can any of these run in parallel?
**Back:** No. This is a pipeline dependency — A must finish before B starts, B before C. Parallel spawning is only possible for independent sub-tasks.
**Domain:** 1
**Trick:** Students try to parallelize everything. Dependencies determine parallelism, not desire.

### #36
**Front:** You spawn 5 subagents in parallel. Subagent 3 fails with an error. What happens to the other 4?
**Back:** The coordinator can wait for all 5 to complete (including the failed one) and then handle the failure, or it can implement error handling that captures partial results from the successful 4. The parallel tasks do not automatically abort each other.
**Domain:** 1
**Trick:** "One fail = all fail" is not automatic. The coordinator decides failure handling.

---

### 1.10 — AgentDefinition Parameters

### #37
**Front:** You're defining an AgentDefinition for a code review subagent. What parameters should you configure?
**Back:** The agent role/purpose, tool set (constrained to reading/analysis tools only), instructions (review guidelines, criteria), and context (PR diff, relevant files). You may also set temperature and max_tokens per subagent.
**Domain:** 1
**Trick:** Students think AgentDefinition is just "which LLM to use." It includes tools, instructions, context, and constraints.

### #38
**Front:** Two subagents need the same tool set but different instructions. How should you configure them?
**Back:** Define two separate AgentDefinitions with the same tool list but different instructions/prompts. AgentDefinition binds role + tools + instructions together. Sharing definitions would mean sharing instructions.
**Domain:** 1
**Trick:** Students try to reuse one definition to save code. Reuse is fine for identical roles but wrong when instructions differ.

---

### 1.11 — Programmatic Enforcement vs Prompt

### #39
**Front:** You need to ensure agents NEVER delete files. Which approach is more reliable: "Please do not delete files" in the system prompt, or a code-level guard that blocks delete operations?
**Back:** Code-level (programmatic) enforcement is deterministic — it always blocks deletes. Prompt guidance is probabilistic — the model may ignore it. Use code enforcement when reliability is critical.
**Domain:** 1
**Trick:** Students over-trust prompts. Prompt guidance is probabilistic; code enforcement is deterministic.

### #40
**Front:** What should you use prompt guidance for, versus programmatic enforcement?
**Back:** Use programmatic enforcement for hard constraints (must/must-not behaviors). Use prompt guidance for preferences and style (e.g., "be concise", "explain your reasoning"). Prompts guide, code enforces.
**Domain:** 1
**Trick:** Students try to enforce everything via prompts or code everything. Each has its place.

### #41
**Front:** Your compliance team mandates that agents can never access PII data. You add a pre-tool-use hook that redacts PII from tool inputs. Is this prompt or programmatic enforcement?
**Back:** This is programmatic enforcement — a code-level hook intercepts and transforms tool calls before execution. It is deterministic and always runs, unlike a prompt instruction that can be ignored.
**Domain:** 1
**Trick:** Hooks blur the line, but hooks are code — they're programmatic enforcement.

---

### 1.12 — PostToolUse Hooks

### #42
**Front:** A tool that searches customer records returns results in a verbose nested JSON format. The model struggles to parse it. What hook can fix this?
**Back:** A PostToolUse hook that normalizes or summarizes the tool output before it's appended to conversation history. This transforms messy output into a clean format the model can effectively use.
**Domain:** 1
**Trick:** Students try to fix this by changing the tool itself. PostToolUse hooks are designed for output normalization.

### #43
**Front:** Your PostToolUse hook detects that a search tool returned 0 results. What should the hook do?
**Back:** The hook can reformat the output to distinguish "no results found" from "query invalid" — for example, adding a flag like `wasSearched: true, resultCount: 0`. This helps the model distinguish access failures from valid empty results.
**Domain:** 1
**Trick:** Students think PostToolUse is just for formatting. It can enrich output with metadata that aids model decision-making.

### #44
**Front:** A PostToolUse hook is proposed that silently fixes errors in tool output before passing it to the model. Is this appropriate?
**Back:** Only if the hook is fixing formatting/normalization issues. Silently altering semantic content can mislead the model. If the tool output is factually wrong, the hook should flag it, not correct it without the model knowing.
**Domain:** 1
**Trick:** "Fix errors silently" seems helpful but violates transparency. The model should see truth.

---

### 1.13 — Pre-Tool-Use Interception Hooks

### #45
**Front:** A pre-tool-use hook inspects tool arguments and detects the model is trying to delete a production file. What should the hook do?
**Back:** The hook should block the call (return an error or skip execution) and provide feedback to the model explaining why the action was blocked and what alternative to use.
**Domain:** 1
**Trick:** Students think the hook should just block silently. The model needs feedback to understand why and choose alternatives.

### #46
**Front:** When should you use a pre-tool-use hook instead of a PostToolUse hook?
**Back:** Pre-tool-use hooks are for validation, modification, or rejection of tool calls BEFORE execution. PostToolUse hooks are for normalizing or enriching results AFTER execution. Use pre-hook for safety guards, post-hook for output formatting.
**Domain:** 1
**Trick:** Mixing up when each hook runs is common. Pre = before execution (guard), Post = after execution (format).

### #47
**Front:** A model tries to call `write_file` with a path outside the allowed directory. A pre-tool-use hook rewrites the path to the allowed directory and proceeds. Is this correct?
**Back:** The hook should reject the call and tell the model the path is invalid, not silently rewrite it. Silently altering inputs breaks the model's understanding of consequences and can lead to further incorrect behavior.
**Domain:** 1
**Trick:** "Fix it silently" feels efficient but breaks the model's cause-effect understanding.

---

### 1.14 — Fixed Pipeline vs Dynamic Decomposition

### #48
**Front:** You're building a system that processes loan applications with 5 legally mandated steps in a fixed order. Should you use a fixed pipeline or dynamic decomposition?
**Back:** Fixed pipeline (prompt chaining) — the order is legally mandated and always the same. Dynamic decomposition would allow the model to reorder steps, which violates compliance requirements.
**Domain:** 1
**Trick:** Students think dynamic is always better. Fixed pipelines are correct when order is predetermined or regulated.

### #49
**Front:** You're building a research assistant that investigates arbitrary user questions. The appropriate approach varies per query. Should you use a fixed pipeline or dynamic decomposition?
**Back:** Dynamic adaptive decomposition — the model decides how to break down each unique research query. A fixed pipeline would be too rigid for diverse, unpredictable questions.
**Domain:** 1
**Trick:** Students think fixed pipelines are simpler for everything. Dynamic decomposition is better for open-ended, varying tasks.

### #50
**Front:** An exam question shows a system that uses model-driven decomposition at runtime to decide sub-tasks. What is this called?
**Back:** Dynamic adaptive decomposition — the LLM determines how to break down the task at runtime based on the specific inputs and context, rather than following a predetermined sequence.
**Domain:** 1
**Trick:** "Prompt chaining" sounds like "dynamic" but is actually fixed. Dynamic means the model decides the breakdown.

---

### 1.15 — Multi-Pass Code Review

### #51
**Front:** You need to review a 30-file PR. A colleague suggests doing it in one pass. Should you?
**Back:** No — for large PRs, use multi-pass review. Break into focused passes: first pass for security, second for logic errors, third for style. Each pass has specific criteria and runs in its own context.
**Domain:** 1
**Trick:** Single-pass over many files overwhelms context. Multi-pass with focused criteria is more thorough.

### #52
**Front:** In multi-pass code review, each pass should be independent. Why?
**Back:** Independent passes prevent confirmation bias — pass 2's findings aren't influenced by pass 1's conclusions. Each pass evaluates with fresh eyes and specific criteria.
**Domain:** 1
**Trick:** Students think sequential passes should share context to "build on" previous findings. Independence prevents bias.

### #53
**Front:** Pass 1 (security) of a multi-pass review finds a SQL injection vulnerability. Pass 2 (logic) doesn't find it because it's not looking for security issues. Is this a failure of pass 2?
**Back:** No — this is by design. Each pass has a focused scope. The final reviewer or coordinator aggregates findings from all passes. This division of labor is more reliable than one agent trying to spot everything.
**Domain:** 1
**Trick:** Students think every pass should catch everything. Multi-pass means divided responsibility.

---

### 1.16 — --resume vs New Session

### #54
**Front:** Your agent crashes after 15 minutes of analysis. You want to continue from where it left off without losing prior work. Which flag do you use?
**Back:** `--resume` — this resumes a previous session from its last state, restoring the conversation history and context so work can continue.
**Domain:** 1
**Trick:** Students think they must restart from scratch. `--resume` picks up where the session left off.

### #55
**Front:** You want to experiment with an alternative approach for the last 3 steps of a session without losing the first 20 steps. Should you resume or fork?
**Back:** Use `fork_session` to create a branch from a previous session point. This preserves the original session and creates a new branch to explore alternative paths.
**Domain:** 1
**Trick:** `--resume` continues the same path. `fork_session` creates a branch for experimentation.

### #56
**Front:** When should you start a NEW session instead of using `--resume`?
**Back:** Start new when: the task context has fundamentally changed, the previous session was deeply confused or stuck, or starting fresh will produce better results than continuing a degraded context. `--resume` is for continuing interrupted work.
**Domain:** 1
**Trick:** Students always resume to "save work." Sometimes fresh context is more valuable than continued context.

---

### 1.17 — fork_session Purpose

### #57
**Front:** An exam question: A Coordinator has run 15 iterations on a complex analysis. It wants to explore two different resolution paths without committing to one. What should it do?
**Back:** Use `fork_session` to create branches for each path. Each branch gets a copy of the current state. The coordinator can explore both and choose the best outcome.
**Domain:** 1
**Trick:** Students think they must choose one path and re-run if it fails. `fork_session` enables safe exploration.

### #58
**Front:** You fork a session to try a different approach. The forked session gives worse results. What happens to the original session?
**Back:** The original session is preserved and unaffected. Forks are branches — they don't modify the original. You can discard the fork and continue with the original.
**Domain:** 1
**Trick:** Students think forking commits changes. Forks are isolated branches; the original remains intact.

---

### 1.18 — Structured Handoff Summary Fields

### #59
**Front:** A subagent completes its work and hands off to the coordinator. What information should the structured handoff summary include?
**Back:** The summary should include: what was done, what was found, any challenges encountered, any decisions made, what context the next agent needs, and any unresolved items requiring attention.
**Domain:** 1
**Trick:** Students provide just the output. Structured handoffs include status, decisions, and context for the next agent.

### #60
**Front:** In a multi-step escalation flow, Agent A completes triage and hands off to Agent B for resolution. Agent A's handoff summary omits the severity classification. What problem occurs?
**Back:** Agent B lacks critical context needed to prioritize and handle the issue appropriately. Structured handoffs must include all fields the downstream agent needs — never assume context will be passed implicitly.
**Domain:** 1
**Trick:** Teams focus on the WHAT (task output) but forget the CONTEXT (severity, decisions, edge cases).

---

### 1.19 — Task Decomposition Risks

### #61
**Front:** A developer builds an agent that decomposes every task, even very simple ones like "sum two numbers." What risk does this introduce?
**Back:** Over-decomposition — creating unnecessary sub-tasks for trivial work. Each subagent spawn costs tokens and time. Use decomposition only when the task genuinely benefits from parallel or specialized handling.
**Domain:** 1
**Trick:** "More subagents = more parallel = faster" is wrong. Decomposition has overhead; use it judiciously.

### #62
**Front:** A coordinator decomposes a task into 50 sub-tasks, each dependent on the previous one. What went wrong in the decomposition?
**Back:** The dependencies defeat the purpose of decomposition. If sub-tasks must run sequentially, there's little benefit to splitting them. Group dependent steps into larger sub-tasks and decompose only at genuine parallelism points.
**Domain:** 1
**Trick:** Students decompose everything without considering dependency chains. Sequential decomposition wastes overhead.

### #63
**Front:** An exam question: The coordinator decomposes a task but the sub-tasks have overlapping scope. Subagent A and Subagent B both end up analyzing the same file. What risk is demonstrated?
**Back:** Inefficient decomposition with overlapping scope wastes tokens and may produce conflicting results. Decomposition should produce mutually exclusive, collectively exhaustive (MECE) sub-tasks.
**Domain:** 1
**Trick:** Overlap seems like "redundancy for safety" but wastes resources and creates conflicts.

---

### 1.20 — Additional Domain 1 Flashcards

### #64
**Front:** A team designs an agent with 15 different tools. The agent frequently calls the wrong tool for simple tasks. Which principle is being violated?
**Back:** Principle of least privilege for tools — too many tools reduces reliability. Each agent should only have the tools it needs. More tools increase the model's confusion in selection.
**Domain:** 1
**Trick:** Students think more tools = more capable. Fewer, focused tools = better accuracy.

### #65
**Front:** You want the model to always call the `search_knowledge_base` tool first before answering any question. How should you enforce this?
**Back:** Use a fixed pipeline step (code-enforced) where the first step always calls `search_knowledge_base`, or use programmatic enforcement. Prompting "always search first" is probabilistic and may be ignored.
**Domain:** 1
**Trick:** Prompting first is unreliable. For mandatory steps, use code enforcement or fixed pipelines.

### #66
**Front:** A coordinator receives outputs from 3 subagents but cannot merge them because the formats are incompatible (one returns JSON, one returns Markdown, one returns raw text). What went wrong?
**Back:** The coordinator failed to specify output format requirements in the subagent instructions. Subagents need explicit output schema requirements to produce compatible results for aggregation.
**Domain:** 1
**Trick:** Students think subagents "just know" the expected format. Output schemas must be specified upfront.

### #67
**Front:** An agent calls a tool, gets a result, but the result doesn't answer the question. The agent calls a second tool, gets more data, and finally answers. How many loop iterations occurred?
**Back:** Three iterations: (1) initial request → `tool_use`, (2) tool result → `tool_use`, (3) tool result → `end_turn`. Each send-response cycle counts as one iteration, regardless of how many tokens or tools are involved.
**Domain:** 1
**Trick:** Students count tools called (2) instead of loop iterations (3). The count starts from the initial message.

### #68
**Front:** An exam question: "What should the orchestrator do when the agent returns stop_reason: tool_use but the tool name is misspelled and doesn't exist in the tool list?"
**Back:** The orchestrator should return an error result to the model indicating the tool was not found, allowing the model to correct itself and call the right tool. Do not attempt to guess the intended tool.
**Domain:** 1
**Trick:** Students try to fuzzy-match the tool name. Strict matching + error feedback is correct.

### #69
**Front:** A student says: "I can reduce token usage by limiting the agent to 3 loop iterations per task." Is this a valid optimization?
**Back:** No — it's an anti-pattern. Arbitrary iteration limits save tokens at the cost of incomplete work. Use token budgets (max allowed spend) instead of iteration caps for cost control.
**Domain:** 1
**Trick:** Token budgets and iteration caps sound similar. One controls cost, the other breaks workflows.

### #70
**Front:** Your coordinator breaks a task into sub-tasks, but each sub-task is so small ("add 2+2") that the overhead of spawning the subagent exceeds the cost of just doing the work directly. What went wrong?
**Back:** The decomposition granularity is too fine. Subagent spawning has overhead (context setup, token costs). Decompose only when sub-tasks are complex enough to justify the overhead of parallel execution.
**Domain:** 1
**Trick:** Students decompose aggressively. The overhead of spawning must be outweighed by the benefit.

### #71
**Front:** A code review agent examines a file and returns `stop_reason: tool_use` with a `search_web` tool call. The orchestrator doesn't have a web search tool available. What should happen?
**Back:** The orchestrator should return a tool error saying the tool is not available. The model can then either choose an alternative approach or end turn with what it knows. Never fake a tool result.
**Domain:** 1
**Trick:** Students may try to give a best-guess answer. Let the model handle unavailable tools.

### #72
**Front:** In a hub-and-spoke system, the coordinator's context fills up with results from 8 subagents. The 9th subagent's result causes a `max_tokens` error. What design change could prevent this?
**Back:** Implement progressive summarization — have the coordinator summarize completed subagent results periodically, or use scratchpad files to store full results externally while keeping summaries in context.
**Domain:** 1
**Trick:** "Add more context window" is not a design solution. Summarization and external storage scale better.

---

## Domain 2: Tool Design and MCP Integration (18%)

### 2.1 — Tool Selection Mechanism

### #73
**Front:** An LLM has 8 tools available. It needs to decide which one to call for a given task. What does the LLM primarily read to make this decision?
**Back:** The tool **description** — the text description of each tool is the primary mechanism the model uses to select which tool to call. Names matter, but descriptions are the main selector.
**Domain:** 2
**Trick:** Students think tool names are the primary selector. Descriptions provide the semantic context for selection.

### #74
**Front:** Three tools all have descriptions starting with "Searches for..." — `search_users`, `search_orders`, `search_products`. The model frequently picks the wrong one. What is the likely cause?
**Back:** Overlapping descriptions. When tool descriptions are too similar, the model can't distinguish between them. Each description should explicitly state when to use that tool and what differentiates it from similar tools.
**Domain:** 2
**Trick:** Students blame the tool names. The descriptions are the culprit — they don't differentiate enough.

---

### 2.2 — Tool Description Best Practices

### #75
**Front:** What should a well-written tool description include?
**Back:** Input format, expected use case, examples of when to call it, boundaries (what it does NOT do), edge cases it handles, and what output format to expect.
**Domain:** 2
**Trick:** Students write one-line descriptions. Effective descriptions are comprehensive — they guide selection and usage.

### #76
**Front:** You have a tool `get_user(user_id: string)` and a tool `get_customer(user_id: string)` that access different databases. A developer gives them identical descriptions. What will happen?
**Back:** The model will frequently call the wrong one because the descriptions don't differentiate the use cases. Each description must clarify which database/context it accesses and when to prefer one over the other.
**Domain:** 2
**Trick:** Identical descriptions for different tools guarantee confusion. Differentiation in descriptions is mandatory.

### #77
**Front:** Your tool description says: "Gets user information." The model uses it for user authentication checks, profile updates, and billing lookups — all incorrectly. What's missing from the description?
**Back:** The description lacks boundaries — it doesn't specify what the tool is NOT for. Add: "Gets user profile information only. Does NOT handle authentication, billing, or account changes. Use `auth_check` for authentication." Boundaries are as important as capabilities.
**Domain:** 2
**Trick:** Students only list what the tool does, not what it doesn't do. Boundaries prevent misuse.

---

### 2.3 — Fixing Overlapping Descriptions

### #78
**Front:** `search_orders` and `search_invoices` both contain "Search for..." in their descriptions. How should you differentiate them?
**Back:** `search_orders`: "Search for customer orders by ID, date range, or status. Use for order fulfillment queries." `search_invoices`: "Search for billing invoices by invoice number, customer ID, or date. Use for payment/billing queries, NOT order status."
**Domain:** 2
**Trick:** Adding "search for" to both is the overlap. Differentiate by business context and use case.

### #79
**Front:** An exam scenario: Two tools have descriptions that differ only by one word ("customer" vs "client"). The model calls the wrong tool 40% of the time. What should you fix?
**Back:** The descriptions are too similar. Rewrite each to be distinct — include different use cases, input examples, and boundary conditions. The model needs semantic differentiation, not synonym differences.
**Domain:** 2
**Trick:** "Customer" and "client" are synonyms to the model. One-word differences are insufficient for differentiation.

---

### 2.4 — isError Flag

### #80
**Front:** An MCP tool returns `{ content: "User not found", isError: true }`. The calling agent sees this and tries a different search strategy. What role did `isError` play?
**Back:** `isError: true` told the model this was an error response, not a valid "user not found" result. Without the flag, the model might treat "User not found" as a successful string result and stop looking.
**Domain:** 2
**Trick:** Without `isError`, error messages look like valid results. The flag tells the model this is a failure to respond to.

### #81
**Front:** A tool returns `{ content: "[]", isError: false }` when no results are found. The model interprets this as "no data exists." Later, a human confirms data does exist but the query was wrong. What should the tool have returned?
**Back:** The tool should have returned a more informative response like `{ content: "No results found for filters X, Y. Query executed successfully (0 matches)", isError: false }` or used a separate field to distinguish "empty results" from "query error."
**Domain:** 2
**Trick:** `isError: false` with empty results is ambiguous. Was the query wrong or is data truly empty? Distinguish these cases.

---

### 2.5 — Error Categories

### #82
**Front:** A database query times out because the server is overloaded. What error category does this fall into, and is it retryable?
**Back:** Transient error — retryable. Transient errors are temporary (network issues, rate limits, timeouts) and may succeed on retry.
**Domain:** 2
**Trick:** Students categorize all errors as "fix the input" type. Transient errors need retry, not input changes.

### #83
**Front:** A tool receives an email address in wrong format ("not-an-email") and cannot proceed. What error category?
**Back:** Validation error — the input was invalid. Not retryable (with the same input). The model must fix the input before retrying.
**Domain:** 2
**Trick:** Validation errors look transient but won't resolve by retrying the same input. The model must change its approach.

### #84
**Front:** A tool call fails because the user's subscription has expired. What error category?
**Back:** Business logic error — a policy violation (user not entitled to this action). Not retryable via same approach; escalate or inform user.
**Domain:** 2
**Trick:** Students think this is validation or transient. Business logic errors require policy decisions, not retries.

### #85
**Front:** A tool call to delete a record fails because the API key does not have delete permissions. What error category?
**Back:** Permission error — access denied. Not retryable. The model must either use a different tool, request higher permissions, or escalate.
**Domain:** 2
**Trick:** Permission errors are often marked as "access denied" but students misclassify them as validation errors.

---

### 2.6 — Retryable vs Non-Retryable Errors

### #86
**Front:** You're designing error handling for a weather API tool. Which errors would you mark as retryable and which as non-retryable?
**Back:** Retryable: network timeout (503), rate limit (429), temporary server error (500). Non-retryable: invalid city name (400), API key expired (401), city not found (404).
**Domain:** 2
**Trick:** 500 errors are typically retryable (transient server issue). 400/401/404 are client errors — retry won't help.

### #87
**Front:** An exam question shows a tool that always returns `{ isError: true, error: "Operation failed" }`. Why is this error response poorly designed?
**Back:** It's a generic error with no category, no detail, no retryable flag, and no guidance. The model gets no information about what failed or whether to retry. Structured errors should include category, code, details, and retryable flag.
**Domain:** 2
**Trick:** "Operation failed" seems like a safe generic message. It gives the model zero actionable information.

### #88
**Front:** A student says: "I'll just retry all errors twice. If it still fails, I'll escalate." Why is this strategy flawed?
**Back:** Retrying non-retryable errors (invalid input, permission denied) wastes tokens and latency on guaranteed failures. The error response should include a `retryable: true/false` flag so the model can decide whether retrying is worthwhile.
**Domain:** 2
**Trick:** Blind retry works for transient failures but wastes resources on validation/permission errors.

---

### 2.7 — tool_choice Modes

### #89
**Front:** You want the model to ALWAYS call the `translate_text` tool, with no option to respond directly. What `tool_choice` setting do you use?
**Back:** `tool_choice: forced` with the specific tool name `translate_text`. This forces the model to call exactly that tool — it cannot respond directly or choose a different tool.
**Domain:** 2
**Trick:** `any` forces tool usage but lets the model pick which tool. `forced` + tool name locks to one specific tool.

### #90
**Front:** An exam question: "Which tool_choice mode allows the model to either call a tool or respond directly without a tool call?"
**Back:** `tool_choice: auto` — the model decides whether to use a tool or respond directly. This is the most flexible mode and the default.
**Domain:** 2
**Trick:** Students confuse `auto` with `any`. `auto` = model can skip tools. `any` = model MUST use a tool.

### #91
**Front:** You have a set of analysis tools. You want the model to always use at least one tool before responding. Which tool_choice mode?
**Back:** `tool_choice: any` — the model must call a tool (it can pick which one) but cannot respond directly. Use this when you want to ensure the model consults some data source before answering.
**Domain:** 2
**Trick:** `any` doesn't let the model skip tools. If you want guaranteed tool usage without specifying which tool, use `any`.

---

### 2.8 — Principle of Least Privilege for Tools

### #92
**Front:** A code review agent has access to: Read, Write, Edit, Bash, Grep, Glob, and Delete. The agent should only READ files. Which tools should it have?
**Back:** Only Read, Grep, and Glob. Write, Edit, Bash, and Delete violate least privilege for a read-only task. Give the agent only what it needs.
**Domain:** 2
**Trick:** Students give all tools "for flexibility." Least privilege means constraining tools to the minimum needed.

### #93
**Front:** A student argues: "Giving a subagent more tools is better because it can handle unexpected situations." Is this correct?
**Back:** No — more tools per agent reduces reliability. The model's tool selection accuracy decreases as the number of tools increases. Constrain tools to only what's necessary for the specific task.
**Domain:** 2
**Trick:** "More tools = more capable" is an anti-pattern. Tool selection accuracy drops with more options.

### #94
**Front:** A customer support agent needs to look up orders and process refunds. What is the minimum set of tools it should have?
**Back:** `search_orders` and `process_refund`. It should NOT have tools for user management, product catalog editing, analytics, or system administration. Exactly the tools it needs for its defined scope.
**Domain:** 2
**Trick:** Students add "just in case" tools. Each extra tool increases confusion and security risk.

---

### 2.9 — Replace General Tools with Constrained Alternatives

### #95
**Front:** You have a generic `search` tool that accepts any query string and searches all company data. Why is this problematic?
**Back:** A generic `search` tool gives the model too much freedom — it may search the wrong data source, return irrelevant results, or access data it shouldn't. Replace with constrained alternatives like `search_users`, `search_orders`, `search_docs` with specific schemas.
**Domain:** 2
**Trick:** Generic search seems flexible. Constrained searches (narrow scope, specific schema) improve accuracy and safety.

### #96
**Front:** Replace a generic `delete_record` tool with what constrained alternatives?
**Back:** Replace with `delete_draft_order(order_id)` and `delete_temp_file(file_path)` — each with specific validation (only deletes drafts/temps, not confirmed or production data). The general tool could accidentally delete anything.
**Domain:** 2
**Trick:** Generic delete is dangerous. Constrained deletes with built-in validation prevent catastrophic mistakes.

### #97
**Front:** An exam scenario: A generic `query_database(sql: string)` tool lets users run any SQL. What constrained alternative should replace it?
**Back:** `search_orders_by_customer(customer_id)` and `get_revenue_by_date_range(start, end)` — specific read-only queries with pre-validated SQL. Never give freeform SQL execution to a model.
**Domain:** 2
**Trick:** Freeform SQL execution is a security and reliability risk. Constrained query tools are safer and more predictable.

---

### 2.10 — MCP Server Config Scopes

### #98
**Front:** An exam question: You configure an MCP server in `.mcp.json` at the project root. Where does this config apply?
**Back:** Project scope — it only applies when Claude Code is working within that specific project. Different projects can have different MCP configurations.
**Domain:** 2
**Trick:** Students think project config is global. `.mcp.json` is project-scoped; `~/.claude.json` is user/global scope.

### #99
**Front:** One developer wants a specific MCP server available for ALL their Claude Code sessions, regardless of project. Where should they configure it?
**Back:** In `~/.claude.json` (user config) — this applies globally across all projects and directories for that user.
**Domain:** 2
**Trick:** A common mistake is putting global config in project files. Global = `~/.claude.json`, project = `.mcp.json`.

### #100
**Front:** A team project needs a shared MCP server config that all team members will have. Where should it go?
**Back:** In `.mcp.json` in the project root, committed to version control. This way, all team members get the same configuration when they clone the repo.
**Domain:** 2
**Trick:** Students put team config in `~/.claude.json` which is per-user and not shared. Project `.mcp.json` is shareable via git.

### #101
**Front:** A developer commits their `~/.claude.json` containing MCP server credentials to the shared repository. What's wrong with this?
**Back:** `~/.claude.json` is user-specific and should not be committed. It may contain secrets. Also, it won't be loaded by other team members' Claude Code (they have their own `~/.claude.json`).
**Domain:** 2
**Trick:** User config files are personal. Project config files are shareable. Never share user config.

---

### 2.11 — .mcp.json vs ~/.claude.json

### #102
**Front:** A developer configures the same MCP server in both `.mcp.json` and `~/.claude.json`. When working in the project, which config is used?
**Back:** The project config (`.mcp.json`) takes precedence within the project directory. Both may merge — but if there's a conflict, project-level overrides user-level.
**Domain:** 2
**Trick:** Students think user config always applies. Project config takes precedence within the project scope.

### #103
**Front:** Why should MCP server configurations in `.mcp.json` use environment variable substitution instead of hardcoded values?
**Back:** Hardcoded values (especially API keys, tokens, passwords) would be exposed in version control. Use `$VAR_NAME` syntax, and set the actual values via environment variables at runtime.
**Domain:** 2
**Trick:** Hardcoding is simpler but insecure. Environment variable substitution keeps secrets out of the repository.

---

### 2.12 — Environment Variable Substitution

### #104
**Front:** An MCP config needs an API key for `ANTHROPIC_API_KEY`. How should you reference it in `.mcp.json`?
**Back:** Use `"apiKey": "${ANTHROPIC_API_KEY}"`. The `${VAR}` syntax tells Claude Code to substitute the value from the environment at runtime.
**Domain:** 2
**Trick:** Students either hardcode the key or use the wrong syntax. `${VAR}` is the correct substitution syntax.

### #105
**Front:** A developer uses `$API_KEY` in `.mcp.json` but the value isn't substituted at runtime. What's likely wrong?
**Back:** The correct syntax is `${API_KEY}` (with curly braces), not `$API_KEY`. The brace-less form may not be recognized as a substitution variable.
**Domain:** 2
**Trick:** Shell scripting uses both forms — but MCP requires `${VAR}` syntax.

---

### 2.13 — MCP Resources Purpose

### #106
**Front:** An MCP server exposes documentation files and database schemas as read-only content the model can reference. What MCP feature is this?
**Back:** MCP Resources — they expose read-only data (docs, schemas, configs, reference material) as resources the model can access without calling tools.
**Domain:** 2
**Trick:** Students confuse Resources with Tools. Resources are read-only data; Tools are executable operations.

### #107
**Front:** When should you expose data as an MCP Resource instead of through a Tool?
**Back:** Use Resources for static or semi-static content the model should read (docs, policies, schemas). Use Tools for dynamic operations that need computation, parameters, or side effects (search, write, compute).
**Domain:** 2
**Trick:** Everything looks like a tool to many developers. Resources are cheaper and more appropriate for read-only reference data.

---

### 2.14 — Built-in Tools

### #108
**Front:** When should you use the Grep built-in tool versus the Glob built-in tool?
**Back:** Grep searches file CONTENTS for a pattern (regex inside files). Glob searches file NAMES/STRUCTURE (file path patterns). Use Grep to find code that references a symbol; use Glob to find files matching a name pattern.
**Domain:** 2
**Trick:** Students confuse "search in files" (Grep) with "search for files" (Glob). Grep = content, Glob = paths.

### #109
**Front:** An exam question: "Which built-in tool should you use to find all JavaScript files modified in the last day?" Can this be done with built-in tools?
**Back:** The built-in tools (Read, Write, Edit, Bash, Grep, Glob) don't directly support file modification time queries. You would need Bash to run a system command like `find . -name "*.js" -mtime -1`.
**Domain:** 2
**Trick:** Students try to force Grep or Glob for everything. Bash is there for system-level operations.

### #110
**Front:** A developer tries to use Edit to make a large change spanning 50 lines across 3 functions. Why might this fail?
**Back:** Edit works best for targeted, localized changes. For large multi-section changes, use Write to replace the entire file. Edit expects to find exact text to replace; large changes increase mismatch risk.
**Domain:** 2
**Trick:** Students use Edit for everything because it's "smarter." Write is better for substantial rewrites.

### #111
**Front:** The Edit tool fails to find the exact text to replace. What fallback strategy should the model use?
**Back:** Fall back to Read (to verify current content) then Write (to rewrite the affected section or entire file). Edit is optimized for precision; when it fails, Write is the reliable fallback.
**Domain:** 2
**Trick:** Students retry Edit with slightly different text. The correct fallback is Write.

---

### 2.15 — Community vs Custom MCP Servers

### #112
**Front:** You need to integrate Claude with a popular API like GitHub. Should you build a custom MCP server or use a community one?
**Back:** Start with a community MCP server if one exists and is well-maintained. Community servers are pre-built and shared. Only build a custom server when no community option fits your exact needs or security requirements.
**Domain:** 2
**Trick:** "Build custom for everything" is NIH (Not Invented Here). Community servers save time and are often well-tested.

### #113
**Front:** A community MCP server does 90% of what you need. When should you build a custom one instead?
**Back:** Build a custom server when: (1) the community server has security/trust issues, (2) your use case has unique constraints the community one can't handle, or (3) you need tight integration with proprietary systems.
**Domain:** 2
**Trick:** "90% is not enough, let's build from scratch" is often wrong. Sometimes the 10% gap can be handled with wrappers.

---

### 2.16 — Additional Domain 2 Flashcards

### #114
**Front:** A tool returns `{ content: "Success", isError: false }` but the operation actually failed (data wasn't saved due to a silent DB error). What is the risk?
**Back:** The model thinks the operation succeeded and will proceed based on false assumptions. This is more dangerous than returning an error — it creates silent data corruption. The tool must detect and report real failures.
**Domain:** 2
**Trick:** False success is worse than a clear error. The model can handle errors but not invisible failures.

### #115
**Front:** An exam question shows: Tool A has description "Retrieves user profile", Tool B has description "Retrieves user settings". When asked "get the user's theme preference," the model calls Tool A. Why?
**Back:** The model doesn't know which tool has theme preference data — the descriptions don't specify. Better descriptions: "Retrieves user profile (name, email, avatar)" and "Retrieves user settings (theme, notifications, privacy)."
**Domain:** 2
**Trick:** Vague descriptions leave the model guessing. Include specific data fields to guide accurate selection.

### #116
**Front:** You're using `tool_choice: forced` with `search_product_catalog`. The model gets the result and another `tool_use` is returned. What happens?
**Back:** `forced` applies per-turn. After the tool result is appended, the next turn's `tool_choice` resets. You must set `tool_choice` on every request if you want persistent forcing.
**Domain:** 2
**Trick:** Students think a single `forced` call persists across all turns. Tool choice is set per request.

### #117
**Front:** A student configures an MCP server with `"env": { "API_KEY": "sk-abc123" }` in `.mcp.json` and commits it. What should they have done instead?
**Back:** Used `"env": { "API_KEY": "${MY_API_KEY}" }` and set `MY_API_KEY` in their local environment or `.env` file. Hardcoded secrets in version-controlled files are a security risk.
**Domain:** 2
**Trick:** Environment variable substitution is not optional — it's a security requirement.

### #118
**Front:** An MCP server exposes a `create_order` tool. The tool description says "Creates a new order." Why is this description insufficient?
**Back:** It doesn't specify: what information is required (customer ID, items, payment method), what side effects occur (charge customer? send confirmation?), what validations are performed, or what errors can occur.
**Domain:** 2
**Trick:** "Creates a new order" is obvious to humans but insufficient for LLMs. They need the full picture of inputs, side effects, and constraints.

### #119
**Front:** A model calls `search_users` with `{ query: "john" }`. The tool returns 200 results. The response is too large and wastes context. What tool design improvement helps?
**Back:** Add pagination parameters to the tool (`page`, `page_size`) or require more specific filters. A tool that returns uncontrolled large results wastes context and degrades performance.
**Domain:** 2
**Trick:** Students focus on making tools work, not on making them context-efficient. Pagination and filters are essential.

### #120
**Front:** Your MCP tool needs to handle file uploads. What's the recommended approach for passing file data?
**Back:** Pass file paths or URIs rather than inline file content. MCP supports `uri` type for resources. Large inline data wastes context and may hit size limits.
**Domain:** 2
**Trick:** Students try to base64-encode large files into tool arguments. Use URI references instead.

### #121
**Front:** An exam scenario: Tool A and Tool B both modify the same database table. Tool A is called and succeeds, but the model then calls Tool B with conflicting data. What tool design principle addresses this?
**Back:** Tools should be idempotent or have clear ordering requirements. If tools conflict, their descriptions should warn about ordering constraints and potential conflicts.
**Domain:** 2
**Trick:** Students design tools in isolation. Tool interactions matter — describe dependencies and ordering in tool descriptions.

---

## Domain 3: Claude Code Configuration and Workflows (20%)

### 3.1 — CLAUDE.md Hierarchy Levels

### #122
**Front:** Where should you place project-wide coding standards that every developer's Claude Code session should follow?
**Back:** In `CLAUDE.md` at the project root. This is the project-level configuration in the 3-level hierarchy: user (`~/.claude/CLAUDE.md`) → project (repo root `CLAUDE.md`) → directory (subfolder `CLAUDE.md`).
**Domain:** 3
**Trick:** Students put everything in user config or one big file. Use the hierarchy: user for personal preferences, project for shared rules, directory for folder-specific instructions.

### #123
**Front:** You want personal workflow preferences (editor shortcuts, personal scripts) available across all your projects. Where should these go?
**Back:** In `~/.claude/CLAUDE.md` — the user-level config. This loads for all projects and is specific to your personal Claude Code setup.
**Domain:** 3
**Trick:** User config is not shared and not in the repo. Project config is shared via version control.

### #124
**Front:** A subfolder `/src/api/` has very specific API documentation conventions that differ from the rest of the project. Where should these instructions live?
**Back:** In `/src/api/CLAUDE.md` — directory-level CLAUDE.md. These instructions load only when working in that directory or its subdirectories. This is more specific than project-level rules.
**Domain:** 3
**Trick:** Students put folder-specific rules in the project root CLAUDE.md, making it bloated. Directory-level CLAUDE.md is the right scope.

### #125
**Front:** A developer has 3 CLAUDE.md files: user, project, and directory level. If all three contain instructions about code formatting, which one takes precedence?
**Back:** The directory-level (most specific scope) takes precedence for files in that directory. The hierarchy is: directory > project > user. More specific scope overrides broader scope.
**Domain:** 3
**Trick:** Students think user config (global) wins. Directory-level (most specific) wins in the hierarchy.

---

### 3.2 — @path Syntax

### #126
**Front:** A project CLAUDE.md is getting too long. The developer wants to break coding standards into a separate file. How can they include it?
**Back:** Use `@path` syntax: `@path ./coding-standards.md` inside CLAUDE.md. This imports the content of the referenced file into CLAUDE.md at that point.
**Domain:** 3
**Trick:** Students try to split into multiple CLAUDE.md files. `@path` lets you compose a single CLAUDE.md from external files.

### #127
**Front:** Can `@path` reference files outside the project directory?
**Back:** No — `@path` references are relative to the CLAUDE.md file's location. They cannot reference files outside the project scope for security reasons.
**Domain:** 3
**Trick:** Students try `@path ../other-project/rules.md`. Only files within the project are accessible.

---

### 3.3 — Max Import Nesting Depth

### #128
**Front:** You have `CLAUDE.md` that `@path`s file A, which `@path`s file B, which `@path`s file C. Can this work?
**Back:** There is a limit on import nesting depth (typically 2-3 levels deep). Deep nesting creates dependency chains that are hard to debug. Keep imports flat (1 level deep) where possible.
**Domain:** 3
**Trick:** Students create deep import hierarchies thinking it's good modularity. Flat imports are simpler and avoid hitting depth limits.

---

### 3.4 — .claude/rules/ Directory

### #129
**Front:** Your project needs testing rules that only apply when working in the `tests/` directory. Where should these go?
**Back:** In `.claude/rules/` directory with a file like `testing-rules.md` that has YAML frontmatter specifying `paths: ["tests/"]`. Rules in this directory load conditionally based on context.
**Domain:** 3
**Trick:** Students put all rules in CLAUDE.md. `.claude/rules/` enables conditional loading based on file paths.

### #130
**Front:** What is the YAML frontmatter structure for a rule that applies to all `.tsx` files?
**Back:** ```yaml
---
paths: ["**/*.tsx"]
---
```
This ensures the rule only loads when TypeScript React files are being worked on.
**Domain:** 3
**Trick:** Rules without path frontmatter load always. Path specifications make rules context-aware and save tokens.

### #131
**Front:** A developer puts all project rules into `.claude/rules/` but doesn't use any path specifications. What problem exists?
**Back:** Without path specs, all rules load in every context — wasting tokens and cluttering context. The purpose of `.claude/rules/` is conditional loading. Rules without path specs are equivalent to putting them in CLAUDE.md.
**Domain:** 3
**Trick:** `.claude/rules/` is only beneficial when rules have path constraints. Otherwise, put them in CLAUDE.md.

### #132
**Front:** When do rules from `.claude/rules/` load?
**Back:** Rules load when Claude Code starts and whenever the working directory or file context changes. They are NOT evaluated continuously but are loaded based on the initial and changing context.
**Domain:** 3
**Trick:** Students think rules are dynamically evaluated on every action. They load at context transitions.

---

### 3.5 — Slash Commands Location

### #133
**Front:** A team wants to share a custom slash command across all team members via version control. Where should it be defined?
**Back:** In `.claude/commands/` in the project repository. Project-scoped commands are shared via git and accessible to all team members working on the project.
**Domain:** 3
**Trick:** Students put shared commands in user config. Project `.claude/commands/` makes commands shareable.

### #134
**Front:** You have a personal slash command `deploy` in `~/.claude/commands/` and the project also has a `deploy` command in `.claude/commands/`. Which runs?
**Back:** The user-level command takes precedence. Personal skills/commands override project-level ones with the same name.
**Domain:** 3
**Trick:** Students think project scope wins. User scope overrides project scope for personal customization.

---

### 3.6 — Skills Frontmatter

### #135
**Front:** An exam question: A SKILL.md has frontmatter `context: fork`. What does this mean?
**Back:** `context: fork` means the skill runs in an isolated session (forked from the main session). This prevents the skill's conversation history from polluting the main context.
**Domain:** 3
**Trick:** Without `context: fork`, the skill runs inline. `fork` means isolated context — the skill can't see main session history and vice versa.

### #136
**Front:** A skill should only have access to Read and Grep tools, not Write or Edit. What frontmatter field controls this?
**Back:** `allowed-tools` — this field restricts which tools the skill can use. Example: `allowed-tools: [Read, Grep]`.
**Domain:** 3
**Trick:** Skills inherit all tools by default. `allowed-tools` prunes the tool set to enforce least privilege on skills.

### #137
**Front:** What does the `argument-hint` frontmatter field in a SKILL.md do?
**Back:** `argument-hint` provides usage hints shown in the command palette when invoking the skill. It helps users understand what arguments the skill expects (e.g., `argument-hint: "<file-path> [--verbose]"`).
**Domain:** 3
**Trick:** Students skip this field. Without `argument-hint`, users don't know how to invoke the skill properly.

### #138
**Front:** A user has a personal skill with the same name as a project skill. Which one runs when invoked?
**Back:** The personal (user-level) skill overrides the project-level skill with the same name. This allows users to customize behavior without modifying project files.
**Domain:** 3
**Trick:** Students think project skills take precedence for consistency. User skills override project skills.

---

### 3.7 — Planning Mode vs Direct Execution

### #139
**Front:** You need Claude to investigate a complex bug that spans multiple files but you don't want it to make any changes yet. What mode should you use?
**Back:** Planning mode — this is investigation-only mode. Claude will analyze the code, propose solutions, but NOT make any file changes. Use for complex tasks needing analysis before action.
**Domain:** 3
**Trick:** Students use direct execution with "don't change files" in the prompt. Planning mode enforces no-changes deterministically.

### #140
**Front:** You're fixing a typo in a variable name. The change is straightforward and well-understood. What mode should you use?
**Back:** Direct execution — the change is simple and the path is clear. No planning phase needed. Go straight to making the edit.
**Domain:** 3
**Trick:** Students use planning mode for everything. Planning mode is for complex/ambiguous tasks, not simple fixes.

### #141
**Front:** An exam question: A developer always uses planning mode "to be safe." What's wrong with this approach?
**Back:** Planning mode adds overhead (an extra analysis phase) for no benefit on simple tasks. It also wastes tokens on analysis when the solution is already clear. Use planning mode for complex tasks, direct execution for simple ones.
**Domain:** 3
**Trick:** "Always safe" sounds good but is inefficient. Match the mode to the task complexity.

### #142
**Front:** When should you switch from direct execution to planning mode?
**Back:** Switch when: the task proves more complex than expected, you encounter unexpected blockers, or Claude makes changes that don't align with intent. Don't start in planning mode for simple tasks — switch when complexity emerges.
**Domain:** 3
**Trick:** The anti-pattern is "start with planning mode." The correct pattern is start simple, escalate to planning when needed.

---

### 3.8 — Explore Subagent

### #143
**Front:** You want to research a third-party library's API without polluting your main Claude Code session's context. What should you use?
**Back:** The Explore subagent — it spawns an isolated subagent for research without polluting the main context. The subagent's conversation is separate and doesn't consume the main session's context window.
**Domain:** 3
**Trick:** Students do research inline, filling their main context with irrelevant findings. Explore subagents keep context clean.

### #144
**Front:** An exam scenario: A developer runs a long research session using inline conversation. After 20 minutes, Claude starts repeating itself. The research isn't done. What went wrong?
**Back:** The developer should have used an Explore subagent for the research. The main session's context got saturated with research conversation, causing context degradation. Explore subagents have their own isolated context.
**Domain:** 3
**Trick:** Research is a perfect use case for subagents. Don't burn main context on investigative work.

---

### 3.9 — /compact Command Risk

### #145
**Front:** Your Claude Code session is getting long. You run `/compact` to compress the conversation. What is the risk?
**Back:** `/compact` compresses the conversation history, which can lose precision — numbers, dates, filenames, and specific details may be lost or garbled in the compression process. Critical context may degrade.
**Domain:** 3
**Trick:** `/compact` seems like a free way to save context. It trades precision for space — essential details can get lost.

### #146
**Front:** When is it safe to use `/compact`, and when should you avoid it?
**Back:** Use `/compact` for general conversation cleanup where approximate context is fine. Avoid it when: exact numbers/dates matter, specific file paths are critical, or you're in the middle of a precise operation that depends on exact details.
**Domain:** 3
**Trick:** Students use `/compact` as a routine cleanup tool. It's a trade-off — use only when precision loss is acceptable.

---

### 3.10 — /memory Command

### #147
**Front:** Your Claude Code session needs to remember a configuration path across multiple sessions. What command stores this?
**Back:** `/memory` — stores persistent information that Claude Code can retrieve across sessions. Unlike conversation context (session-only), `/memory` persists.
**Domain:** 3
**Trick:** Students put persistent info in CLAUDE.md or in conversation. `/memory` is purpose-built for cross-session persistence.

### #148
**Front:** What is the key difference between CLAUDE.md and `/memory`?
**Back:** CLAUDE.md is developer-defined static configuration loaded at startup. `/memory` is dynamic information stored at runtime that persists across sessions. CLAUDE.md = static rules; `/memory` = dynamic persistence.
**Domain:** 3
**Trick:** Students treat `/memory` as "another CLAUDE.md." They serve different purposes — one is config, one is runtime state.

---

### 3.11 — -p Flag for CI/CD

### #149
**Front:** You want to run a code review via Claude Code in your CI pipeline without any interactive prompts. What flag do you use?
**Back:** The `-p` (or `--print`) flag — this runs Claude Code in non-interactive mode, suitable for programmatic use in CI/CD. Claude outputs the response directly without opening an interactive session.
**Domain:** 3
**Trick:** Students try to use interactive mode with piping. `-p` is designed for non-interactive programmatic use.

### #150
**Front:** What is the limitation of using `-p` in CI/CD compared to interactive Claude Code?
**Back:** `-p` mode doesn't support multi-turn interactions or tool use the same way interactive mode does. It's best for single-shot requests. Complex multi-step workflows should use Batch API for programmatic use.
**Domain:** 3
**Trick:** Students think `-p` is equivalent to interactive mode. It's a simplified non-interactive mode.

---

### 3.12 — --output-format and --json-schema

### #151
**Front:** You run Claude Code with `-p "Summarize these files" --output-format json`. What changes in the output?
**Back:** The output will be structured JSON instead of raw text. This makes the output programmatically parseable for further automated processing.
**Domain:** 3
**Trick:** Students forget `--output-format json` changes the output format. Without it, output is unstructured text.

### #152
**Front:** You need Claude Code to output a specific JSON structure (e.g., `{ files: string[], vulnerabilities: number }`). How do you specify this?
**Back:** Use `--json-schema` with a JSON schema definition. For example: `--json-schema '{"type":"object","properties":{"files":{"type":"array","items":{"type":"string"}},"vulnerabilities":{"type":"integer"}}}'`
**Domain:** 3
**Trick:** `--output-format json` just gives structured JSON. `--json-schema` constrains the JSON structure to a specific schema.

---

### 3.13 — Independent Review Instances

### #153
**Front:** A PR review finds 10 issues. A second review of the same code, run in the same session, only finds 3 issues. What's likely happening?
**Back:** Confirmation bias — the second review was influenced by the first review's findings. Each review should run in its own independent instance (separate session) to prevent bias.
**Domain:** 3
**Trick:** Running reviews in the same session contaminates results. Independent = separate sessions.

### #154
**Front:** How do you prevent duplicate review comments from multiple Claude Code review passes on the same PR?
**Back:** Each review pass gets independent context (separate sessions) focusing on specific criteria (security, style, logic). The aggregation step deduplicates before posting comments. Without coordination, independent passes may flag the same issues.
**Domain:** 3
**Trick:** Deduplication needs to be explicit at the aggregation step. Independent instances don't know about each other's findings.

---

### 3.14 — Batch API

### #155
**Front:** An exam question: A company wants to reduce Claude Code costs for weekly security scans. What API should they use?
**Back:** Batch API — which offers ~50% cost savings for non-urgent, bulk processing. Results can take up to 24 hours but the cost reduction is significant.
**Domain:** 3
**Trick:** Students think cost savings are the only factor. Batch API also has NO multi-turn tool calling — a critical trade-off.

### #156
**Front:** You're building an interactive code review tool that needs real-time feedback for developers. Can you use Batch API?
**Back:** No — Batch API results can take up to 24 hours. For interactive/blocking use cases, use synchronous API. Batch is for non-urgent, bulk processing.
**Domain:** 3
**Trick:** Batch's 50% savings tempt teams to use it everywhere. SLA requirements (real-time vs overnight) dictate the choice.

### #157
**Front:** A critical limitation of Batch API for code review workflows is that it does NOT support multi-turn. What does this mean in practice?
**Back:** Batch API processes individual requests independently. The model cannot call tools, read files, or iterate on results within a batch request. Each request is a single-turn prompt-response. For code review, you must include all necessary context in the initial prompt.
**Domain:** 3
**Trick:** Batch = single-turn only. No tool calling, no iteration. All context must be in the prompt upfront.

### #158
**Front:** In a Batch API request, what is the `custom_id` field used for?
**Back:** `custom_id` is a unique identifier you assign to each request in the batch. When results come back (potentially out of order), `custom_id` tells you which result corresponds to which request. Essential for failure handling and result mapping.
**Domain:** 3
**Trick:** Students skip custom_id and rely on array position. Batch results can return in any order — custom_id is the only reliable mapping.

---

### 3.15 — Additional Domain 3 Flashcards

### #159
**Front:** A developer creates a SKILL.md with `context: fork` but notices the skill can't access variables set in the main session. Is this a bug?
**Back:** No — this is by design. `context: fork` means the skill runs in an isolated session with no access to the main session's context. This is the expected isolation behavior.
**Domain:** 3
**Trick:** Isolation sometimes surprises developers. `fork` means full separation — no context sharing with the main session.

### #160
**Front:** What is the difference between `.claude/commands/` and `.claude/skills/`?
**Back:** Commands are lightweight slash-triggered actions (like `/deploy`). Skills are full agent capabilities with their own instructions, tools, and optional context isolation. Commands are simpler; skills are more powerful.
**Domain:** 3
**Trick:** Students treat them interchangeably. Commands = simple actions; Skills = agent-like capabilities.

### #161
**Front:** A developer puts a rule in `.claude/rules/` but it never seems to load. The file has no YAML frontmatter. What's wrong?
**Back:** Without YAML frontmatter specifying `paths`, the rule loads in ALL contexts. If it "never loads," check that the file is in the correct location and named correctly. Also, rules load at context transitions, not continuously.
**Domain:** 3
**Trick:** Students expect immediate loading. Rules load at context setup/transition.

### #162
**Front:** An exam question: "What happens when a new directory-level CLAUDE.md is created while a session is already running?"
**Back:** The new CLAUDE.md is NOT loaded into the running session. CLAUDE.md files are loaded at session start. The developer must start a new session for the new rules to take effect.
**Domain:** 3
**Trick:** Students expect hot-reloading of config. CLAUDE.md is loaded at session initialization only.

### #163
**Front:** You run `claude -p "review this commit" --output-format json`. The output is JSON but the structure varies between runs. What's missing?
**Back:** `--json-schema` to constrain the output structure. `--output-format json` ensures JSON output but doesn't enforce a specific schema. For consistent structure, add a schema definition.
**Domain:** 3
**Trick:** JSON output without a schema can vary in structure. Schema guarantees consistent parsing.

### #164
**Front:** A team uses Batch API for urgent critical bug analysis, expecting results in 5 minutes. What's wrong?
**Back:** Batch API has no SLA guarantee for fast responses — results can take up to 24 hours. For time-sensitive operations, use synchronous API. Batch is for non-urgent bulk processing.
**Domain:** 3
**Trick:** Batch = 50% cheaper but potentially 24-hour turnaround. Don't use it when speed matters.

### #165
**Front:** A CI pipeline runs Claude Code in `-p` mode to analyze every commit. The pipeline keeps failing because Claude asks follow-up questions. What's wrong?
**Back:** In `-p` mode, Claude should not ask follow-up questions (it's non-interactive). The prompt may be ambiguous or missing context. Make the prompt self-contained with all necessary information.
**Domain:** 3
**Trick:** Interactive prompts break in `-p` mode. Design prompts for single-turn, complete-information format.

---

## Domain 4: Prompt Engineering and Structured Output (20%)

### 4.1 — Explicit Criteria vs Vague Instructions

### #166
**Front:** A prompt says: "Review this code for quality issues." The model produces inconsistent reviews — sometimes catching style issues, sometimes missing security problems. What's the root cause?
**Back:** Vague instructions ("quality issues") lead to inconsistent interpretation. Replace with explicit criteria: "Check for: (1) SQL injection vulnerabilities, (2) memory leaks, (3) unhandled errors. Report each with file:line."
**Domain:** 4
**Trick:** "Quality" means different things to different models (and same model at different times). Explicit criteria produce reliable, consistent outputs.

### #167
**Front:** An exam scenario: One prompt says "Be thorough." Another says "Check all 5 categories: security, performance, accessibility, error handling, logging. Rate each 1-5 with evidence." Which produces more reliable output?
**Back:** The second prompt with explicit categories and rating scale. "Be thorough" is vague and subjective — the model doesn't know what thorough means in your context. Explicit criteria eliminate ambiguity.
**Domain:** 4
**Trick:** "Be thorough" seems like a reasonable ask. It's too vague — the model can't calibrate to your definition of thorough.

---

### 4.2 — False Positive Impact

### #168
**Front:** A security scanning prompt has very high recall (catches 99% of issues) but also generates false positives on 30% of clean code. What is the practical problem?
**Back:** False positives erode trust. Developers will start ignoring or dismissing ALL findings, including the real ones. It's better to have slightly lower recall with higher precision than to flood users with unreliable alerts.
**Domain:** 4
**Trick:** High recall seems ideal until the false positive cost is counted. Each false positive makes the system less trusted.

### #169
**Front:** Your code review system flags 20 issues. Only 3 are real. Developers have started merging code without waiting for review results. What happened?
**Back:** The high false positive rate has eroded trust. Developers no longer trust the system's outputs. The system needs fewer, more accurate findings (higher precision) even at the cost of missing some real issues.
**Domain:** 4
**Trick:** More findings = more value? No — more accurate findings = more value. False positives destroy credibility.

---

### 4.3 — Few-Shot > Textual Descriptions

### #170
**Front:** You want the model to classify customer messages as "urgent," "normal," or "low priority." You write detailed rules about what each means. The model still misclassifies. What's missing?
**Back:** Text descriptions alone are less effective than few-shot examples. Show 3-5 examples of each priority level with explanations of WHY each was classified that way. Examples outperform rules.
**Domain:** 4
**Trick:** Developers write rules because they think that's how systems work. LLMs learn better from examples than from abstract rules.

### #171
**Front:** Why do few-shot examples outperform textual descriptions for guiding model behavior?
**Back:** Few-shot examples show the exact input-output pattern the model should follow. Text descriptions are abstract and open to interpretation. Examples demonstrate, descriptions explain — demonstration is more effective.
**Domain:** 4
**Trick:** The human bias is toward writing instructions. Model bias is toward pattern-matching examples.

### #172
**Front:** An exam question: "When should you use few-shot examples instead of textual descriptions?"
**Back:** Use few-shot examples for: ambiguous scenarios (where correct output isn't obvious), output formatting (exact structure desired), distinguishing acceptable vs problematic (show both good and bad). For simple, unambiguous rules, textual descriptions suffice.
**Domain:** 4
**Trick:** Few-shot is not always better. Text descriptions are fine for simple, unambiguous rules. Few-shot shines for ambiguity and formatting.

---

### 4.4 — JSON Schema Guarantees

### #173
**Front:** You define a JSON Schema requiring `"severity": { "enum": ["low", "medium", "high"] }`. The model outputs `{ "severity": "low", "confidence": 0.9 }`. What is guaranteed about this output?
**Back:** JSON Schema guarantees SYNTACTIC correctness — the output will be valid JSON with the right fields and types. It does NOT guarantee SEMANTIC correctness — the severity may be wrong for the actual issue.
**Domain:** 4
**Trick:** Students trust schema-validated output as "correct." Schema validates structure, not truth.

### #174
**Front:** Your JSON Schema ensures the output is valid JSON with the expected fields. The model outputs `severity: "high"` for a typo in a comment. What went wrong?
**Back:** Nothing went wrong with the schema — it enforced syntax correctly. The semantic error (misclassifying a typo as high severity) is a prompt/instruction quality issue, not a schema issue. JSON Schema can't enforce correctness of content.
**Domain:** 4
**Trick:** This is the classic exam trap: conflating schema validation with semantic validation.

---

### 4.5 — Required vs Optional Fields

### #175
**Front:** A JSON Schema has both `required: ["issue", "severity", "fix"]` and `optional: ["notes"]`. The model outputs all required fields but omits `notes`. Is this acceptable?
**Back:** Yes — by design. The model correctly filled all required fields and omitted an optional one. If you always want `notes`, make it required. Don't mark fields required to "encourage" filling — require them only when always needed.
**Domain:** 4
**Trick:** Students mark things optional but then complain when they're missing. Optional means the model can skip them.

### #176
**Front:** A student sets all fields as required in a JSON Schema "to make sure the model fills everything." What is the downside?
**Back:** Required fields force the model to provide a value even when it doesn't have one, leading to hallucinated/default values. Use optional fields for information the model may not always have, and allow null for uncertain values.
**Domain:** 4
**Trick:** Required = must provide a value, not "must provide a correct value." The model will make something up if forced.

---

### 4.6 — Nullable Fields

### #177
**Front:** A code review JSON Schema has `"suggested_fix": { "type": "string" }`. The model always provides a fix suggestion — even for issues where there's no clear fix. What's the problem?
**Back:** The field is not nullable, so the model is forced to invent a fix. Make it nullable (`"type": ["string", "null"]`) so the model can honestly say "null" when no clear fix exists.
**Domain:** 4
**Trick:** Non-nullable fields force the model to fabricate. Nullable fields allow honest "I don't know" responses.

---

### 4.7 — Enums with "other" + Detail

### #178
**Front:** A classification schema has `"type": { "enum": ["bug", "feature", "docs", "performance"] }`. The model encounters an issue that doesn't fit any category. What happens?
**Back:** The model will force-fit it into one of the existing categories (probably incorrectly). Add an "other" value to the enum plus a `detail` field for explanation: `"type": { "enum": ["bug", "feature", "docs", "performance", "other"] }` and `"other_detail": { "type": "string" }`.
**Domain:** 4
**Trick:** Rigid enums don't handle edge cases. "Other" + detail field gracefully covers unexpected situations.

---

### 4.8 — "unclear" Enum Value

### #179
**Front:** A sentiment analysis schema has `"sentiment": { "enum": ["positive", "negative"] }`. The model encounters an ambiguous message. How should the enum be improved?
**Back:** Add `"unclear"` to the enum. Without it, the model is forced to classify ambiguous cases as either positive or negative, introducing false signals. `"unclear"` lets the model acknowledge ambiguity.
**Domain:** 4
**Trick:** Two-category enums force binary decisions. "Unclear" handles the real-world gray areas.

---

### 4.9 — Retry-With-Error-Feedback Pattern

### #180
**Front:** A generation task produces output that fails validation (missing required field `summary`). What's the correct recovery pattern?
**Back:** Use retry-with-error-feedback: send the model the original request plus the validation error ("The summary field was missing. Please regenerate with a complete summary."). The model typically fixes the issue when given specific error feedback.
**Domain:** 4
**Trick:** Students either discard the output and start fresh, or silently fix it themselves. Retry with specific feedback is the standard pattern.

### #181
**Front:** You retry with error feedback 3 times and the model keeps outputting the same invalid structure. What should you conclude?
**Back:** Retry is ineffective when the model lacks the information to fix the issue. The required information may simply be absent from the context. No amount of retrying will help if the model doesn't know the answer. Change the prompt or add context.
**Domain:** 4
**Trick:** Students retry indefinitely. If the required info isn't in context, retry cannot succeed.

---

### 4.10 — Self-Correction Pattern

### #182
**Front:** A model generates: `"total": 100, "items": [25, 25, 25, 25], "calculated_total": 100`. This is correct. But how does the self-correction pattern work when it's NOT correct?
**Back:** The self-correction pattern asks the model to output BOTH the stated total AND a recalculated total. Example: ask for `stated_total` and `calculated_total` (sum of items). If they differ, the model detects its own arithmetic error and can correct.
**Domain:** 4
**Trick:** The model won't self-correct unless you explicitly ask it to verify. The `stated_total` vs `calculated_total` pattern forces verification.

### #183
**Front:** A model states: "Total expenses: $500" and lists items summing to $480. Without which pattern would this slip through?
**Back:** Without the self-correction pattern (stated vs calculated), the $20 discrepancy would go undetected. The pattern forces the model to explicitly compute and compare, catching arithmetic errors.
**Domain:** 4
**Trick:** Students trust the model's stated totals. The model can make arithmetic errors just like humans — verification catches them.

---

### 4.11 — Batch vs Sync for Blocking Checks

### #184
**Front:** You need to validate every code change before it hits production. The validation must complete within 30 seconds. Should you use sync or batch API?
**Back:** Synchronous API — it provides real-time validation that waits for a response. Batch API can take up to 24 hours and is unsuitable for blocking pre-deployment checks.
**Domain:** 4
**Trick:** Batch cost savings tempt teams to use it for everything. When latency matters, use synchronous.

### #185
**Front:** A weekly security analysis runs on 10,000 files and has a 48-hour deadline. Should you use sync or batch?
**Back:** Batch API — the deadline is flexible (48 hours), batch handles large volumes at 50% cost savings, and no interactive tool calls are needed for static analysis.
**Domain:** 4
**Trick:** Teams sometimes default to sync for everything. Batch is the right choice for non-urgent bulk processing.

---

### 4.12 — SLA Planning for Batch

### #186
**Front:** You submit 500 requests to Batch API for a compliance audit due tomorrow. What SLA should you plan for?
**Back:** Plan for up to 24-hour completion time. Batch API processes results when capacity is available — there's no guaranteed fast turnaround. If you need results by morning, submit well in advance, ideally 24+ hours before deadline.
**Domain:** 4
**Trick:** "Batch processes fast for small loads" is wrong. Batch speed depends on system load, not request count.

### #187
**Front:** An exam question: "What is the maximum processing time for Batch API requests?"
**Back:** Up to 24 hours. Batch API is designed for non-urgent bulk processing and has no guarantee of fast completion.
**Domain:** 4
**Trick:** Students think batch is "fast enough." Batch = potentially 24-hour wait.

---

### 4.13 — Multi-Pass Review Architecture

### #188
**Front:** A code review system uses a single pass to check security, performance, and style simultaneously. Reviews miss issues in each category. What should change?
**Back:** Use multi-pass review: Pass 1 = security only, Pass 2 = performance only, Pass 3 = style only. Each pass has focused criteria and runs independently. Combined results give comprehensive coverage.
**Domain:** 4
**Trick:** Single-pass reviews split the model's attention. Focused passes catch more per category.

### #189
**Front:** In a multi-pass review, why must each pass be independent (separate context)?
**Back:** Independent passes prevent confirmation bias. Pass 2 should not know Pass 1's findings — it might stop looking for issues Pass 1 already found. Each pass evaluates with fresh eyes.
**Domain:** 4
**Trick:** "Build on previous findings" sounds efficient but introduces bias. Independent passes are more thorough.

---

### 4.14 — Confirmation Bias in Self-Review

### #190
**Front:** A model writes code and then reviews its own changes in the same session. It finds no issues. A human reviewer finds 5 bugs. What psychological principle explains this?
**Back:** Confirmation bias — the model is biased toward confirming its own work as correct. It's testing to validate, not to find problems. Independent review instances (separate context/session) avoid this.
**Domain:** 4
**Trick:** "The model should know its own code best" is wrong. The model's self-review is biased. Fresh eyes (independent instance) find more issues.

### #191
**Front:** How do you architect a system to avoid confirmation bias in code reviews?
**Back:** Use separate, independent review instances — ideally different agents or sessions with no shared context. The reviewer should not see the original author's reasoning. Fresh, independent perspective = more accurate review.
**Domain:** 4
**Trick:** Sharing context between author and reviewer feels efficient but guarantees bias.

---

### 4.15 — Additional Domain 4 Flashcards

### #192
**Front:** A prompt includes "Evaluate severity as low, medium, or high." The model outputs "urgent" for critical issues. What went wrong?
**Back:** The enum wasn't constrained via JSON Schema. With free text, the model may output undefined values. Constrain valid outputs with an enum: `"severity": { "type": "string", "enum": ["low", "medium", "high"] }`.
**Domain:** 4
**Trick:** Free text fields are flexible but unpredictable. Enums constrain outputs to valid values.

### #193
**Front:** Your JSON Schema has `"confidence": { "type": "number" }`. The model outputs `confidence: 0.999`. What problem might arise?
**Back:** The model is expressing unrealistically high confidence. Consider constraining the range with `minimum: 0, maximum: 1` and provide examples of what each range means (0.0-0.3 = low, 0.7-1.0 = high).
**Domain:** 4
**Trick:** Models tend toward overconfidence. Unconstrained numeric fields produce inflated values.

### #194
**Front:** An exam question: "You add 3 few-shot examples showing high-severity issues. Now the model classifies everything as high-severity. What happened?"
**Back:** The few-shot examples are biased — they only show one category. Few-shot examples must proportionally represent all expected output categories. If high-severity is only 5% of real cases, only 5% of examples should be high-severity.
**Domain:** 4
**Trick:** Few-shot examples are powerful — biased examples produce biased outputs. Balance your examples.

### #195
**Front:** You use retry-with-error-feedback 5 times and the output is still wrong. The error says "Missing customer_name field." What might be the root cause?
**Back:** The model may not have the customer's name in its context. No amount of retrying can fill a gap in the source data. Instead of retrying, check that the necessary information is available in the input context.
**Domain:** 4
**Trick:** Infinite retry is a common mistake. If the data isn't there, retry can't create it.

### #196
**Front:** A schema has `"issue_type": { "enum": ["bug", "feature"] }` with no "other" option. For a "documentation" issue, the model outputs `"bug"`. Is this the model's fault or the schema's?
**Back:** The schema's fault — it didn't provide a valid option for "documentation." The model was forced to choose the closest available category. Add "docs" and/or "other" to the enum.
**Domain:** 4
**Trick:** Students blame the model for incorrect classification. If your enum doesn't cover all cases, the model must force-fit.

### #197
**Front:** A student writes: "I added 10 few-shot examples but the model still ignores the output format." What might be wrong with the examples?
**Back:** The examples may not clearly distinguish input and output sections, or may be inconsistent in format. Few-shot examples need clear separation markers (e.g., "Input:" / "Output:") and consistent structure. Also, put the schema BEFORE the examples.
**Domain:** 4
**Trick:** Examples without clear structure markers confuse the model. Show which parts are inputs and which are expected outputs.

### #198
**Front:** An exam question: "What's the trade-off between required fields and output reliability?"
**Back:** Required fields ensure the field is ALWAYS present, but the model may hallucinate values when it doesn't have real data. Optional fields allow the model to omit fields when uncertain. More required fields ≠ more reliability — it can mean more hallucination.
**Domain:** 4
**Trick:** The natural instinct is to require everything. This forces hallucination when data is missing.

### #199
**Front:** Your retry-with-error-feedback includes the validation error verbatim. The second attempt passes validation but the output is still semantically wrong. What happened?
**Back:** Retry-with-error-feedback fixes FORMAT errors (schema violations) but doesn't guarantee semantic correctness. The model fixed the format issue but still didn't have the right information for a correct semantic answer.
**Domain:** 4
**Trick:** Retry fixes syntax/format issues. It doesn't add missing knowledge. Semantic correctness requires the right context.

### #200
**Front:** A security review prompt uses the term "critical" without defining it. One review flags all unvalidated inputs as critical; another flags only SQL injection as critical. What's causing the inconsistency?
**Back:** "Critical" is a vague term. One model or session interpreted it broadly, another narrowly. Define severity levels with explicit criteria: "Critical = direct data breach or RCE risk. High = sensitive data exposure. Medium = best practice violation."
**Domain:** 4
**Trick:** Subjective terms ("critical," "significant," "minor") mean different things to the model across sessions. Define them explicitly.

---

## Domain 5: Context Management and Reliability (15%)

### 5.1 — Progressive Summarization Risks

### #201
**Front:** An agent summarizes a conversation every 10 turns. After 4 summarizations, the final summary is missing a critical client requirement that was mentioned on turn 3. What happened?
**Back:** Progressive summarization progressively loses detail — especially numbers, dates, specific requirements, and precise details. Each summarization compresses, and details drop out. Critical facts should be extracted into a separate block rather than relying on recursive summarization.
**Domain:** 5
**Trick:** "Summarization preserves all information" is wrong. Each compression loses detail, and losses compound across multiple summarizations.

### #202
**Front:** When should you use summarization vs extracting critical facts into a structured block?
**Back:** Use summarization for general context that can tolerate approximation. Extract critical facts (dates, prices, requirements, decisions) into a separate structured block that is preserved verbatim. Never trust summarization for precise details.
**Domain:** 5
**Trick:** Students treat summarization as a universal compression tool. Critical facts need non-destructive preservation.

### #203
**Front:** An exam scenario: After 3 rounds of summarization, an agent starts contradicting itself about a deadline. The original deadline was "March 15, 2025." What likely happened?
**Back:** The summarization lost the exact date, replacing it with "mid-March" or similar approximation during compression. The next summarization further degraded it. Critical dates must be extracted to a separate block before summarization.
**Domain:** 5
**Trick:** "March 15" → "mid-March" → "March" → "spring" in progressive summarization. Precision degrades.

---

### 5.2 — Lost-in-the-Middle Effect

### #204
**Front:** A long conversation has important instructions at the start, medium-priority details in the middle, and the current query at the end. The model handles the query poorly but remembers the start. What explains this?
**Back:** The lost-in-the-middle effect — LLMs perform worse on information in the middle of long contexts. They remember the beginning (primacy) and end (recency) best. Place critical information at the start or end, never in the middle.
**Domain:** 5
**Trick:** Students assume context is uniformly accessible. The middle of context is a blind spot.

### #205
**Front:** You have a 50-page codebase as context. The most critical constraint is on page 25. What should you do?
**Back:** Move the critical constraint to the start or end of the context. Restructure the input so important rules are at the top (in the system prompt or early instructions) or repeated at the end. Don't bury critical information in the middle.
**Domain:** 5
**Trick:** "The model has 200k context — it can read everything" is wrong. Where information appears in context significantly impacts recall.

### #206
**Front:** An exam question: "How do you mitigate the lost-in-the-middle effect when you can't reduce context length?"
**Back:** Restructure context to put critical information first (use system prompt), repeat key facts at the end, and use structured formats (bullet points, headers) to make information scannable. Avoid prose paragraphs in the middle of context.
**Domain:** 5
**Trick:** Structure matters. Prose in the middle is lost faster than structured lists.

---

### 5.3 — Case Facts Block Strategy

### #207
**Front:** A 200-turn customer support conversation needs analysis. The agent keeps forgetting the customer's account tier (mentioned on turn 5). What design pattern prevents this?
**Back:** Extract a case facts block at the beginning that is updated after every turn: `Account: Premium | Issue: Billing | Amount: $299 | Escalation: None`. This block persists and doesn't get lost in conversation history.
**Domain:** 5
**Trick:** Information mentioned once in conversation gets buried. A persistent facts block keeps critical data accessible.

### #208
**Front:** What should a case facts block contain in a customer support scenario?
**Back:** Customer identifiers, account status, issue summary, key decisions made, pending actions, escalation status, and timestamps. Any information another agent might need at a glance.
**Domain:** 5
**Trick:** Students create verbose summaries. Case facts blocks should be concise, structured, and easy to scan.

---

### 5.4 — Trimming Tool Outputs

### #209
**Front:** A search tool returns 5,000 lines of CSV data. The agent needs only the summary statistics. What should the orchestrator do?
**Back:** Trim the tool output before appending it to conversation history — either truncate to the first/last N lines, or extract a summary. Long tool outputs waste context and may push other critical information out.
**Domain:** 5
**Trick:** "Return full results for completeness" wastes context. Trim tool outputs to what the model actually needs.

### #210
**Front:** An exam scenario: An agent calls `list_files` which returns 10,000 filenames. The agent's context fills up. What should the tool or hook do?
**Back:** The tool should have pagination or the PostToolUse hook should truncate/summarize the output. Returning unbounded results is a tool design flaw. The hook can say "10,000 files found. Showing first 100: [list]."
**Domain:** 5
**Trick:** Tools should be context-aware. Never return unbounded results that can fill the context window.

---

### 5.5 — Escalation Triggers

### #211
**Front:** A support agent encounters a query about a company policy it doesn't have instructions for. What escalation trigger applies?
**Back:** Policy gap — no rule exists for this situation. The agent should escalate because it cannot reliably handle the situation without guidance.
**Domain:** 5
**Trick:** Students have the agent guess or make up a policy. Policy gaps are valid escalation triggers.

### #212
**Front:** A user explicitly says "I want to speak to a human manager." Should the agent try to resolve or escalate?
**Back:** Escalate immediately — explicit user request for human is a valid escalation trigger. Do not attempt to convince the user to stay with the automated system.
**Domain:** 5
**Trick:** Agents that argue with users who ask for humans damage trust. Explicit requests for human = immediate escalation.

### #213
**Front:** An agent tries to resolve an issue but fails twice. It's stuck in a loop. What escalation trigger applies?
**Back:** Inability to progress — the agent is stuck and cannot resolve the issue. It should escalate with a summary of what was attempted and what failed.
**Domain:** 5
**Trick:** Students let agents loop forever "trying one more time." Inability to progress is a valid escalation trigger.

---

### 5.6 — Invalid Escalation Triggers

### #214
**Front:** A developer builds an escalation trigger based on sentiment analysis: "If customer seems angry, escalate." Why is this unreliable?
**Back:** Sentiment analysis is unreliable — the model is bad at judging emotional states from text. Anger can be expressed politely, and strong language can be passion, not anger. Use explicit triggers (user asks to escalate) rather than inferred triggers.
**Domain:** 5
**Trick:** Sentiment-based escalation sounds empathetic. It's unreliable and prone to false positives.

### #215
**Front:** An agent is designed to escalate when its self-rated confidence drops below 0.7. Why won't this work reliably?
**Back:** Self-rated confidence is unreliable — models cannot accurately judge their own confidence. They may express high confidence when wrong and low confidence when right. Escalation must be based on explicit triggers, not self-assessment.
**Domain:** 5
**Trick:** "Confidence: 0.95" sounds reassuring. Models are consistently overconfident, making self-rated confidence useless.

### #216
**Front:** An exam question lists escalation triggers. Which are VALID and which are INVALID? (Sentiment analysis, user explicitly asks, inability to progress, self-rated confidence)
**Back:** VALID: user explicitly asks, inability to progress. INVALID: sentiment analysis, self-rated confidence. Valid triggers are objective and observable; invalid triggers are subjective and unreliable.
**Domain:** 5
**Trick:** The exam tests whether you can distinguish objective triggers from subjective ones.

---

### 5.7 — Resolving Ambiguous Matches

### #217
**Front:** A user asks about "order #1234." Two orders with that ID exist (one from 2023, one from 2024). How should the agent handle this?
**Back:** Ask for additional identifiers: "I found two orders with ID #1234 — one from March 2023 (\$50) and one from June 2024 (\$200). Could you specify which one?" Never guess or pick the most recent.
**Domain:** 5
**Trick:** "Pick the most recent" seems helpful. Asking the user resolves ambiguity definitively.

### #218
**Front:** An agent resolves ambiguous entity matches by picking the first result. A user gets incorrect information. What should the agent have done?
**Back:** Asked for additional identifiers to disambiguate. Picking arbitrarily when uncertain leads to wrong answers and erodes trust. When in doubt, ask.
**Domain:** 5
**Trick:** "Always pick first" is efficient but wrong more than 50% of the time.

---

### 5.8 — Structured Error Context Fields

### #219
**Front:** A subagent encounters a database timeout and passes an error to the coordinator. The coordinator receives: "Something went wrong." Why is this insufficient?
**Back:** The error lacks structure — no error type, no category, no retryable flag, no details. A structured error context should include: error type (transient), message (database timeout), recoverable flag (true), and suggestion (retry with backoff).
**Domain:** 5
**Trick:** "Something went wrong" is the worst error message. Structured errors enable informed recovery decisions.

### #220
**Front:** What fields should a structured error context include when passing errors between agents?
**Back:** Error type/category, error message, isRetryable flag, recovery suggestion, affected data/context, and stack trace or relevant details. This gives the receiving agent enough information to decide next steps.
**Domain:** 5
**Trick:** Students include the error message only. The recovery suggestion and retryable flag are what the next agent needs to act.

---

### 5.9 — Distinguishing Timeout vs "0 Results"

### #221
**Front:** A search tool returns `{ results: [], status: "success" }`. Another call returns `{ results: [], status: "timeout" }`. The agent treats both as "no data found." Why is this a problem?
**Back:** A "success" with 0 results means no data exists (valid empty result). A "timeout" with 0 results means the query couldn't complete (access failure). The agent should retry on timeout but accept "no data" on success.
**Domain:** 5
**Trick:** Empty results and failed queries look the same without status metadata. Always distinguish them.

### #222
**Front:** An exam scenario: A database query returns an empty array. 5 minutes later, data IS found with a different query. What does this indicate about the first result?
**Back:** The first query likely had an overly restrictive filter or wrong parameters — it's an access/query failure, not a genuine "no data" result. Distinguishing query-failure-empty (wrong query) from genuine-empty (no data exists) requires context about query parameters.
**Domain:** 5
**Trick:** Empty results, especially from complex queries, may indicate wrong query parameters, not non-existent data.

---

### 5.10 — Coverage Annotations

### #223
**Front:** A security audit agent reviews 200 files. It needs to track which files it has checked so another agent doesn't re-check them. What should it use?
**Back:** Coverage annotations — mark checked files (e.g., add a comment, update a manifest, or log checked paths). This prevents redundant work and ensures comprehensive coverage.
**Domain:** 5
**Trick:** Students rely on memory or session context. Coverage annotations are explicit markers that persist.

### #224
**Front:** After Agent A checks files 1-100 and marks them, Agent B picks up files 101-200. Agent B's coverage annotation system accidentally overwrites Agent A's marks. What went wrong?
**Back:** The coverage annotation system isn't append-only or conflict-aware. Use separate tracking or an append-only log so agents don't overwrite each other's progress.
**Domain:** 5
**Trick:** Shared state between agents needs conflict resolution. Append-only logs prevent overwrites.

---

### 5.11 — Scratchpad Files

### #225
**Front:** An agent is analyzing a large codebase and discovering relationships between modules. It's writing down intermediate findings. The context is filling up. What should it do?
**Back:** Write intermediate findings to scratchpad files (external files) instead of keeping them in conversation context. This frees up context for active reasoning while preserving findings for later retrieval.
**Domain:** 5
**Trick:** Students keep everything in conversation. Scratchpad files are external storage that preserves findings without consuming context.

### #226
**Front:** An exam question: "What's the difference between keeping findings in conversation vs scratchpad files?"
**Back:** Conversation-kept findings consume active context (may trigger lost-in-the-middle). Scratchpad files preserve findings externally, only read when needed. Scratchpad = non-volatile storage; conversation = active working memory.
**Domain:** 5
**Trick:** Students think "I wrote it down, I'll remember." Without scratchpad files, the model can't look it up — it must be in context or on disk.

---

### 5.12 — Confidence Calibration

### #227
**Front:** A model rates its confidence as "0.95" for a fact-checking answer. The answer is wrong. What does this reveal about confidence calibration?
**Back:** The model's confidence is not calibrated to actual accuracy. Models tend to be overconfident, especially on answers that sound plausible. Self-rated confidence is NOT a reliable indicator of correctness.
**Domain:** 5
**Trick:** High confidence ≠ high accuracy. Models cannot reliably self-assess.

### #228
**Front:** A developer uses field-level confidence scores (one per output field) instead of an overall score. Why is this better?
**Back:** Field-level confidence reveals which parts of the answer are reliable and which are uncertain. Overall confidence can mask that some fields are well-supported and others are guesses. Field-level scoring enables partial trust.
**Domain:** 5
**Trick:** Overall confidence = one number hiding multiple uncertainties. Field-level = granular, actionable.

---

### 5.13 — Stratified Random Sampling

### #229
**Front:** A test suite evaluates 1,000 samples from a code review system: 950 are standard bug fixes, 50 are edge cases. Overall accuracy is 98%. But edge case accuracy is only 60%. What went wrong in testing?
**Back:** The test used random sampling without stratification. The 2% error rate on 950 standard cases masked the 40% error rate on edge cases. Use stratified random sampling — test proportionally across all categories, including rare but critical ones.
**Domain:** 5
**Trick:** "98% accuracy" looks great. Aggregate metrics lie — they hide poor performance on small but critical categories.

### #230
**Front:** An exam scenario: A compliance system is 99.5% accurate overall but handles "unusual transaction pattern" cases at 45% accuracy. Why does this matter?
**Back:** The unusual cases are the HIGHEST RISK cases (money laundering, fraud). Poor performance on high-risk categories is dangerous even with high overall accuracy. Test and report accuracy per category, not just overall.
**Domain:** 5
**Trick:** Overall accuracy hides category-specific failures. The most important categories often have the worst accuracy.

---

### 5.14 — Claim → Source Mapping

### #231
**Front:** An agent summarizes a report and states "Revenue grew 20%." A human asks "Where does that number come from?" The agent can't answer. What design pattern was missing?
**Back:** Claim → source mapping — every factual claim should be traceable to its source. The agent should track which source produced each piece of information, so claims can be verified later.
**Domain:** 5
**Trick:** Students focus on getting the right answer. Without source mapping, you can't verify or audit claims.

### #232
**Front:** How should an agent present claims from multiple sources to avoid confusion?
**Back:** Each claim should include an attribution: "Salesforce reported 20% growth in Q3 2024 (source: Salesforce earnings report, p.5)." This allows readers to verify and compare claims across sources.
**Domain:** 5
**Trick:** Unattributed claims are untrustworthy. Source attribution makes claims verifiable.

---

### 5.15 — Handling Conflicting Data

### #233
**Front:** Source A says "GDPR fine: €10M" and Source B (dated one day later) says "GDPR fine: €20M." The agent picks €10M. What's wrong?
**Back:** The agent should present BOTH values with attribution rather than silently picking one. There may be a reason for the discrepancy (appeal, correction, different sources). Presenting conflicting data honestly is better than a false consensus.
**Domain:** 5
**Trick:** "Pick the most common value" is wrong. Present conflicts with attribution and let the user decide.

### #234
**Front:** An exam scenario: Two sources disagree on a date. How should the agent format the response?
**Back:** "Event X occurred on [Date A] (source: Company blog, 2024) or [Date B] (source: SEC filing, 2025). The discrepancy may be due to announcement vs actual effective date." Present both with context about the difference.
**Domain:** 5
**Trick:** Silently favoring one source hides uncertainty. Presenting both with context builds trust.

---

### 5.16 — Include Dates for Context

### #235
**Front:** A user asks: "What is the company's revenue?" The agent finds a document stating "$5B." The agent doesn't check the document's date. Why is this risky?
**Back:** Without dates, temporal context is lost. "$5B" from 2020 is very different from "$5B" from 2025. Always include dates: "Revenue was $5B as of FY2023 (source: Annual Report 2023)."
**Domain:** 5
**Trick:** "Revenue $5B" is useless without dates. Temporal context is essential for interpretation.

### #236
**Front:** An agent says: "Claude 3.5 Sonnet is the most capable model." This was true in 2024 but false in 2025 (Claude 4 exists). What's the fix?
**Back:** Include dates: "As of June 2024, Claude 3.5 Sonnet was the most capable Anthropic model." This makes the statement temporally grounded and accurate even as facts change.
**Domain:** 5
**Trick:** Information ages. Dates make claims temporally bounded and prevent stale information from being misinterpreted.

---

### 5.17 — Context Degradation Signs

### #237
**Front:** After 50 turns, an agent starts repeating the same analysis in a loop, misses details it previously caught, and makes contradictory statements. What is happening?
**Back:** Context degradation — the model's context is saturated or the lost-in-the-middle effect is causing it to lose track of earlier information. Signs include repetition, missed details, and contradictions.
**Domain:** 5
**Trick:** Students think the model is "getting confused." Context degradation is a known limitation of long sessions.

### #238
**Front:** What actions should you take when you observe context degradation?
**Back:** (1) Use `/compact` (with awareness of precision loss), (2) delegate to subagents to distribute context load, (3) write intermediate results to scratchpad files and start fresh, (4) extract critical facts into a persistent block.
**Domain:** 5
**Trick:** "Restart the session" is the nuclear option. Subagents, scratchpads, and the case facts block can recover without full restart.

---

### 5.18 — State Persistence for Crash Recovery

### #239
**Front:** An agent is processing a 100-step workflow. At step 73, the system crashes. All progress is lost. What design pattern would have prevented this?
**Back:** State persistence — the agent should periodically persist its progress (step completed, intermediate results, decisions made) to a file or database. On restart, it can resume from the last persisted state rather than starting over.
**Domain:** 5
**Trick:** Students assume in-memory state is sufficient. Persistent state enables crash recovery.

### #240
**Front:** An exam question: "What should an agent persist for effective crash recovery?"
**Back:** Current step/position, intermediate results, decisions made, context summary, and any pending operations. The persisted state should be sufficient to reconstruct the agent's working context.
**Domain:** 5
**Trick:** "Save every turn" is too much. Save state when it changes meaningfully (after tool calls, after decisions, at milestones).

---

### 5.19 — Additional Domain 5 Flashcards

### #241
**Front:** An agent uses `/compact` four times in a long session. After the fourth compaction, it incorrectly states that a previously agreed budget of "$12,500" was "$12,000." What happened?
**Back:** Each compaction compressed precision. "$12,500" → "~$12.5k" → "~$12k" → "$12,000". The rounding errors compounded across multiple summarizations. Critical numbers should be extracted into a persistent facts block, not left in summarizable conversation.
**Domain:** 5
**Trick:** Small rounding at each step compounds. The first compaction loses $500, the next loses $500 more.

### #242
**Front:** An agent is summarizing a customer conversation. The original said "The deadline is June 30, 2025, at 5 PM EST." The summary says "Deadline: end of June." What information was lost?
**Back:** The exact date was rounded (June 30 → end of June), the time (5 PM) was dropped, and the timezone (EST) was dropped. These details may be critical for compliance or scheduling.
**Domain:** 5
**Trick:** "End of June" sounds equivalent to "June 30." It's not — June 28 is also "end of June." Precision matters.

### #243
**Front:** A multi-source research agent finds Source A says "The API supports 10,000 requests/second" and Source B says "The API supports 5,000 requests/second." The agent's output says "The API supports approximately 7,500 requests/second." What's wrong?
**Back:** The agent averaged conflicting claims without attribution. The correct approach: "Source A claims 10,000 req/s (vendor docs); Source B claims 5,000 req/s (independent benchmark). The variance may be due to test conditions." Present conflict, don't smooth it.
**Domain:** 5
**Trick:** Averaging conflicting data creates a number supported by neither source. Present the conflict.

### #244
**Front:** A legal document analysis agent outputs: "The contract requires 30 days notice for termination." The actual clause says "30 business days." How did this error occur?
**Back:** The summarization dropped the qualifier "business" — a common precision loss. Generic summarization tends to simplify qualifiers. Exact legal language should be quoted, not summarized.
**Domain:** 5
**Trick:** "Days" vs "business days" is a critical legal difference. Summarization loses these qualifiers.

### #245
**Front:** An agent receives context with the history ordered: Oldest → Newest. Critical instructions are in the middle. The agent misses them. What redesign helps?
**Back:** Reorder context to put critical instructions at the START (in system prompt) and current query at the END. Move middling details to external references. The lost-in-the-middle effect means middle content is least reliably processed.
**Domain:** 5
**Trick:** Chronological ordering (oldest first) is natural for humans but worst for LLM context utilization.

### #246
**Front:** An agent researching product pricing finds three different prices for the same product across three sources. It picks the middle value and proceeds. What is the risk?
**Back:** The agent silently resolved a data conflict without attribution. The user has no way to know there was disagreement. Always present conflicts with source attribution — the user needs to understand the uncertainty.
**Domain:** 5
**Trick:** "Picking the middle" seems like unbiased compromise. It's still a unilateral decision that hides uncertainty.

### #247
**Front:** A team's evaluation shows "95% accuracy on legal clause detection." When stratified by clause type, "indemnification" clauses have 60% accuracy. The team cites the 95% figure in their report. What's unethical about this?
**Back:** They're using an aggregate metric that masks poor performance on a critical category (indemnification clauses). Aggregate metrics should be reported alongside per-category breakdowns, especially for high-risk categories.
**Domain:** 5
**Trick:** 95% sounds impressive until you realize the most important category in your domain is at 60%.

### #248
**Front:** An exam question: "How do you test whether your agent system performs well on all categories, not just common ones?"
**Back:** Use stratified random sampling — ensure your test set includes representative samples from ALL categories, including rare/edge cases. Report accuracy per category, not just overall. Don't let high-volume easy categories mask low-volume hard category failures.
**Domain:** 5
**Trick:** Random sampling gives you proportionally more common cases. Stratified sampling ensures every category is tested.

### #249
**Front:** An agent's context window shows signs of degradation. The agent is in the middle of a critical multi-step analysis. What's the immediate best practice?
**Back:** Persist the current state (intermediate findings, decisions made, remaining steps) to a scratchpad file. Then you can start a new session with the persisted state as context, avoiding the degraded context while preserving progress.
**Domain:** 5
**Trick:** "Push through" with degraded context leads to errors. Persist state and start fresh.

### #250
**Front:** A search returns 0 results. The agent reports "No customers found matching that criteria." Later investigation shows the query was wrong. What should have been different?
**Back:** The agent should have verified the query or distinguished "no results" from "query may be wrong." A coverage annotation approach: if a query returns 0 results, note it as "Searched by X, found 0 — possible filter issue" rather than definitive "no customers exist."
**Domain:** 5
**Trick:** Accepting 0 results at face value is naive. Complex queries can fail silently.

### #251
**Front:** An agent building a report from 50 sources includes 49 sources with dates and 1 source without. What should it do with the undated source?
**Back:** Include the undated source but note: "Source [X] claims [Y], but the document date is unknown. The reliability of this claim depends on when it was made." Flagging temporal uncertainty is better than silent inclusion.
**Domain:** 5
**Trick:** Silent inclusion of undated sources gives them false equivalence with dated ones.

### #252
**Front:** A student says: "I solved the lost-in-the-middle problem by putting everything in a 3-line summary." Is this correct?
**Back:** No — aggressive summarization loses precision (see progressive summarization risks). The solution is structural: put critical info at start or end, use structured facts blocks, and move reference content to external sources — NOT compress everything into a tiny summary.
**Domain:** 5
**Trick:** "Shorter = better" is wrong for complex information. Structure matters more than length.

---

### 5.20 — Additional Cross-Domain & Advanced Flashcards

### #253
**Front:** A developer designs an agent that uses both prompt guidance ("don't delete files") AND a PreToolUse hook that blocks delete operations. The model tries to delete a file. What happens?
**Back:** The PreToolUse hook blocks the deletion (programmatic enforcement is deterministic). The prompt guidance was redundant but provides a nice explanation for the block. Programmatic enforcement always wins — it's the safety net.
**Domain:** 1
**Trick:** Students think prompt + code is redundant. Both layers provide defense-in-depth: prompt explains, code enforces.

### #254
**Front:** An exam question tests: What's the difference between planning mode and an Explore subagent?
**Back:** Planning mode investigates WITHOUT making changes in the main session. Explore subagent spawns an isolated subagent for research, keeping the main context clean. Planning mode = no-changes investigation; Explore = offloaded research.
**Domain:** 3
**Trick:** Both involve investigation. Planning mode is inline (no changes), Explore is an isolated subagent.

### #255
**Front:** A developer uses `tool_choice: forced` with `search_knowledge_base`, but after getting results, the model needs to call `generate_report`. What tool_choice should the second turn use?
**Back:** `tool_choice: auto` or `any` — `forced` is per-turn and must be reset. The developer must decide per-request what `tool_choice` to use. After the search returns, the model may need freedom to choose the next tool.
**Domain:** 2
**Trick:** `forced` persists in the developer's mind but not in the API. Set it per request.

### #256
**Front:** An agent uses `/compact` and loses a critical filename ("production-config.yaml" becomes "config file"). Later, it writes to the wrong file. What's the root cause?
**Back:** The `/compact` compression lost specific identifiers (filenames, paths). Exact names become approximations during compression. Critical identifiers should be extracted into a persistent facts block before compacting.
**Domain:** 3
**Trick:** `/compact` is not lossless. It trades precision for context — and filenames are exactly the kind of detail that gets lost.

### #257
**Front:** A system routes customer queries to specialized agents. The customer says "I have a payment issue." The coordinator sends this to the payment agent. But the issue is actually about a payment-related bug, not billing. The payment agent can't help. What went wrong?
**Back:** The decomposition was too shallow — "payment issue" covers both billing inquiries and payment-related bugs. The coordinator should gather more information before routing, or the routing should include context sufficient for the subagent to identify mismatches.
**Domain:** 1
**Trick:** Surface-level routing (keyword → agent) is brittle. Context-aware decomposition is more reliable.

### #258
**Front:** A tool returns `{ isError: true, error: { category: "permission", message: "API key lacks write access", retryable: false }}`. The model immediately asks the user for a different API key. What enabled this good behavior?
**Back:** The structured error response included all necessary fields: category (permission), message (what's missing), and retryable (false). With `retryable: false`, the model knew retrying was pointless and instead took a different action.
**Domain:** 2
**Trick:** Without `retryable: false`, the model might have retried the same failing call multiple times. The flag saves wasted attempts.

### #259
**Front:** A developer designs JSON Schema with required fields for a triage output. In testing, the model fills all required fields with plausible-looking but incorrect data when the real data is unavailable. What happened?
**Back:** Required fields forced the model to hallucinate values. Use optional or nullable fields for information the model may not always have: "if unavailable, set `cause: null`" is better than requiring a fabricated cause.
**Domain:** 4
**Trick:** Required = guaranteed presence, not guaranteed correctness. The model will invent to satisfy the requirement.

### #260
**Front:** Pre-deployment checks must pass before any code reaches production. The audit processes 500 files and must complete within 20 minutes. Which mode (sync/batch) and why?
**Back:** Synchronous API — the check is blocking (gate before production) and time-sensitive (20 minute SLA). Batch API's 50% savings don't matter if results arrive too late. Sync is for blocking, time-sensitive operations.
**Domain:** 4
**Trick:** Cost savings tempt teams toward batch. When the process blocks the pipeline, speed beats cost.

### #261
**Front:** A summary of a long conversation says: "Customer had an issue with order." The original said: "Customer received wrong item in order #ABC-789 on Dec 15, returned it Dec 16, replacement shipped Dec 18, arrived damaged Dec 20." How much information was lost?
**Back:** Nearly all critical information was lost: order number, specific issue (wrong item), dates of every event, and the fact it happened TWICE (wrong item + damaged replacement). The summary is useless for resolution.
**Domain:** 5
**Trick:** "Customer had an issue with order" is technically true but entirely unactionable. Summarization for actionability must preserve specifics.

### #262
**Front:** An agent uses the `--resume` flag to continue a session. The resumed session behaves differently — making different decisions than before. What could explain this?
**Back:** The resumed session may have been compacted or trimmed, losing nuance. Or the model's output is non-deterministic (temperature > 0). `--resume` restores the conversation but not the exact model state that produced previous responses.
**Domain:** 1
**Trick:** Students expect `--resume` to produce identical behavior. Non-determinism and context compression can change outputs.

### #263
**Front:** Two MCP servers expose tools with identical names. What happens when the model calls that tool?
**Back:** Tool names should be unique across the combined tool set. If two servers have a tool named "search," there will be a conflict. Prefix tool names by server (e.g., "docs_search", "web_search") to avoid ambiguity.
**Domain:** 2
**Trick:** Students assume MCP handles name collisions automatically. Name conflicts break tool selection.

### #264
**Front:** An agent summarizes a conversation and states: "The user was frustrated." The original conversation mentioned no frustration — the user was calm throughout. What went wrong?
**Back:** The model hallucinated an emotional state during summarization. Summarization isn't just compression — it's interpretation. The model may infer or invent details that weren't present. Never trust summarization to preserve emotional or subjective content without explicit markers.
**Domain:** 5
**Trick:** Summarization adds interpretation, not just compression. Inferred details may be wrong.

### #265
**Front:** A developer uses the same tool description for `search_local_db` and `search_remote_db`. The model picks `search_local_db` for a query that requires up-to-date data. Why?
**Back:** The identical descriptions gave the model no basis to differentiate. `search_local_db` happened to appear first or was alphabetically preferred. Description must clarify: "Use for cached/local data (may be stale)" vs "Use for real-time authoritative data."
**Domain:** 2
**Trick:** Identical descriptions make tool selection arbitrary. The model can't choose correctly without differentiation cues.

### #266
**Front:** A JSON Schema has `"nullable": true` for `"error_details"`. An error occurs but the model outputs `null` for this field. The logging system crashes because it expects a string. Who is responsible?
**Back:** The system design — it should handle `null` values from nullable fields. If the field is nullable, downstream systems must handle `null`. Don't make a field nullable if you can't handle null.
**Domain:** 4
**Trick:** Downstream systems often assume fields are always present with values. Nullable fields require null handling downstream.

### #267
**Front:** An exam question: "After 3 retries with error feedback, the output format is still wrong. What's the likely cause?"
**Back:** The error feedback may be unclear or the schema too complex. Simplify the schema, provide a clear example, or check that the error message itself is not confusing. Also consider: is the model's context getting confused by the repeated retry history?
**Domain:** 4
**Trick:** "Retry harder" isn't always the answer. The schema, feedback, or context may need improvement.

### #268
**Front:** A developer sets `tool_choice: any` expecting the model to call tools. Instead, the model fabricates data from a known database. What misunderstanding exists?
**Back:** `any` forces tool use — the model MUST call a tool, it cannot respond directly. If the model is fabricating data, the code isn't properly enforcing `any`. Check that `tool_choice: any` is correctly set and that tools are properly defined.
**Domain:** 2
**Trick:** Students think `any` is a "suggestion." `any` is a requirement — the model cannot skip tools entirely.

### #269
**Front:** A multi-pass review system reviews the same code 3 times. The first pass took 5 minutes, the second 2 minutes, the third 4 minutes. What might this variance indicate?
**Back:** Each pass has different criteria — security review (thorough, complex), style review (fast, formulaic), logic review (medium). Variance is expected when passes have different scopes. However, if the same pass varies wildly, context degradation may be occurring.
**Domain:** 1
**Trick:** "Same code, same time" is wrong. Different review passes have different complexity and scope.

### #270
**Front:** A user asks about "the merger deal." The company has been involved in 3 merger deals over 5 years. The agent picks "the most recent one." What's wrong?
**Back:** The agent should ask "Which merger deal? The 2021 acquisition of Company X, the 2023 merger with Company Y, or the 2024 acquisition of Company Z?" Ambiguous references should be resolved by asking for identifiers, not by guessing.
**Domain:** 5
**Trick:** "Most recent" seems logical. Without confirming, the agent may answer about the wrong deal entirely.

### #271
**Front:** A developer configures a skill in SKILL.md without `context: fork`. During execution, the skill's conversation pollutes the main session. What's missing?
**Back:** The `context: fork` frontmatter. Without it, the skill runs inline in the main session. Adding `context: fork` spawns it in an isolated session, preventing context pollution.
**Domain:** 3
**Trick:** Skills default to inline execution. `context: fork` is opt-in isolation.

### #272
**Front:** A tool always returns `isError: true` with `retryable: true`. The model retries 5 times before giving up. What's wrong with the tool's design?
**Back:** If the error is always retryable, retries will always fail — wasting 5x tokens. The tool should correctly classify errors. If the tool is consistently failing, it's not a transient issue — the root cause must be addressed.
**Domain:** 2
**Trick:** `retryable: true` doesn't mean "retry infinitely." If retries always fail, the error classification is wrong.

### #273
**Front:** A group chat support agent sees conflicting information: User A said "I want a refund" and User B said "No refund, just fix it." What should the agent do?
**Back:** The agent should flag the conflict and ask for clarification: "I see conflicting requests — User A wants a refund while User B wants a fix. Can you confirm which action to take?" Never silently pick one or average the requests.
**Domain:** 5
**Trick:** "Fix it" and "refund" are mutually exclusive. Picking one without confirmation is a mistake.

### #274
**Front:** A JSON Schema has: `"type": "string"` for `"error_explanation"`. The model sometimes outputs extremely long explanations (2,000 words). What constraint is missing?
**Back:** Add `"maxLength": 200` to constrain output length. JSON Schema allows length constraints on strings. Without them, the model can generate arbitrarily long text, wasting tokens and potentially exceeding limits.
**Domain:** 4
**Trick:** Schema validation isn't just about types. Length constraints control verbosity and token usage.

### #275
**Front:** An exam question: "A company reviews 1 million transactions monthly for fraud. 99.9% are legitimate. Their model achieves 99% accuracy on fraud detection. On a day with 10,000 transactions, how many false positives occur?"
**Back:** With 99% accuracy on fraud: ~10 actual fraud cases in 10,000 transactions. 99% recall catches ~9.9 of them but also generates ~100 false positives (1% of 9,990 legitimate transactions). The false positives overwhelm the team.
**Domain:** 5
**Trick:** High accuracy on imbalanced datasets is misleading. 99% accuracy with 0.1% fraud rate means 10x more false positives than real frauds.

### #276
**Front:** A PreToolUse hook detects the model is about to call an expensive API. It should block the call and suggest an alternative. What should the hook return?
**Back:** The hook should return an error-like response explaining why the call was blocked and suggesting alternatives. The model needs feedback to understand and choose a different approach. Silent blocks without explanation confuse the model.
**Domain:** 1
**Trick:** Blocking without feedback leaves the model confused about what to do next.

### #277
**Front:** A CLAUDE.md uses `@path` to include 15 files. Claude Code startup becomes slow. What's the likely cause?
**Back:** Each `@path` import requires reading and processing a file. 15 imports add startup overhead. Consolidate related content into fewer files and only import what's needed for the current context.
**Domain:** 3
**Trick:** `@path` is convenient but not free. Too many imports slow down startup.

### #278
**Front:** A student says: "I use Batch API for everything because it's 50% cheaper." When would this strategy cause a production incident?
**Back:** During any time-sensitive operation — deploying a hotfix, responding to an active security incident, or any blocking check that needs immediate results. Batch API's 24-hour SLA is inappropriate for real-time operations.
**Domain:** 4
**Trick:** "50% cheaper" is tempting. The 24-hour SLA makes batch unsuitable for urgent operations.

### #279
**Front:** A subagent raises an error, and the coordinator immediately spawns a replacement subagent with the same task. The replacement also fails. What pattern is missing?
**Back:** The coordinator is not passing error context to the replacement. The replacement makes the same mistakes because it doesn't know what the first subagent tried. Structured error handoff should include "what was attempted, what failed, and why."
**Domain:** 1
**Trick:** Blindly retrying the same task gives the same result. Error context enables different approaches.

### #280
**Front:** A developer writes a tool description: "Searches customer database." Why might the model use this tool to answer "What's the CEO's name?"
**Back:** The description is too vague. The model might think "customer database" means the company's primary database, which could include employee data. The description should specify: "Searches the CUSTOMER (not employee) database for customer records only."
**Domain:** 2
**Trick:** Vague scope descriptions lead to tool misuse. Explicit boundaries ("not for X") prevent misuse.

### #281
**Front:** In a multi-pass review, Pass 1 finds a logic error. Pass 2 (starting independently) also finds the same logic error. When results are combined, the same issue is reported twice. What's missing in the aggregation step?
**Back:** Deduplication is missing. The coordinator/aggregator should merge findings from independent passes and remove duplicates before presenting results.
**Domain:** 1
**Trick:** Independent passes may catch the same issues. Deduplication must happen at aggregation time.

### #282
**Front:** A developer adds `max_iterations: 100` to their agent loop "to be safe." The agent sometimes loops all 100 times for complex queries. What's the real issue?
**Back:** The agent may be stuck in a loop (repeating the same tool call because results don't help). An iteration cap masks the real problem — the agent needs better tools, error handling, or escape conditions. Fix the underlying loop, don't just cap iterations.
**Domain:** 1
**Trick:** Max iterations treat the symptom (long loops) not the cause (poor tool design or feedback loops).

### #283
**Front:** A security audit of an agent reveals it can access the `delete_user` tool even when processing read-only queries. What is violated?
**Back:** Principle of least privilege — the agent should only have tools matching the current operation mode. If in read-only mode, all write/delete tools should be removed from the tool set, not just hidden via instructions.
**Domain:** 2
**Trick:** Prompting "don't delete" is not enough. The tool should not be in the tool list at all.

### #284
**Front:** A model receives context with the history: 50 turns of conversation, then 10 critical rules, then the current query. The model ignores the rules. What's the most likely cause?
**Back:** Lost-in-the-middle effect — the rules are in the middle of the context, after 50 turns of history. Rules should be at the START (in the system prompt area) where they're most reliably processed.
**Domain:** 5
**Trick:** Ordering matters. Rules after conversation history suffer from primacy/recency effects — the conversation takes primacy, the query takes recency.

### #285
**Front:** A developer puts a rule in `.claude/rules/` with `paths: ["**/*.ts"]`. The rule applies to all TypeScript files. A bug fix in a `.ts` file triggers the rule. The bug fix is unrelated to the rule's topic. What's the downsides?
**Back:** The rule loaded unnecessarily, wasting tokens and context. The path glob matched the file but the rule wasn't relevant to the actual change. Consider more specific paths to avoid loading irrelevant rules.
**Domain:** 3
**Trick:** Broad path patterns trigger rules unnecessarily. More specific patterns save tokens.

### #286
**Front:** A tool always returns `{"status": "ok"}` even when the underlying operation fails silently. The model thinks everything is fine. What's the fundamental design flaw?
**Back:** The tool is lying by omission. It should use the `isError` flag or return error details when operations fail. Silent failures are more dangerous than explicit errors — they create false confidence.
**Domain:** 2
**Trick:** "Always succeed" seems user-friendly. Silent failures mask problems and lead to incorrect downstream decisions.

### #287
**Front:** An agent is in a session with degraded context. The user tells it to retry a search it already attempted. The agent doesn't remember trying it before. What happened?
**Back:** Context degradation caused the agent to lose the memory of the previous search attempt. The search result was in the lost-in-the-middle zone. The agent needs either a scratchpad file recording attempted searches or a fresh session with persisted state.
**Domain:** 5
**Trick:** The agent isn't "forgetful" — the context is structurally degraded. The information is in context but in a position the model can't access reliably.

### #288
**Front:** A subagent coordinator sends the same instructions to all subagents but expects different outputs based on file content. The subagents return very similar analyses because the instructions are identical. What should change?
**Back:** The instructions should be customized per subagent based on the file being analyzed. "Analyze this file for security issues" works for any file, but tailoring instructions to the file type (e.g., "This is an auth module — check for session vulnerabilities") produces better results.
**Domain:** 1
**Trick:** Copy-paste instructions are efficient but produce cookie-cutter results. Tailored instructions yield better analysis.

### #289
**Front:** A user submits a batch process with 10,000 requests but no `custom_id` on any request. One request fails. How do you identify which one?
**Back:** You can't reliably — batch results may return in any order, and without `custom_id`, you can't map results back to requests. Always include `custom_id` if you need to handle individual failures.
**Domain:** 3
**Trick:** "Results come in order" is not guaranteed. `custom_id` is the only reliable mapping.

### #290
**Front:** An exam scenario: "Your MCP server fails to start because an environment variable is missing. The error message is: 'API_KEY not set.' The `.mcp.json` uses `${API_KEY}`. What's wrong?"
**Back:** The `${API_KEY}` variable is not set in the environment. The config references the variable correctly, but the actual environment must have `API_KEY` defined. Check that the variable is exported before starting Claude Code.
**Domain:** 2
**Trick:** Env var substitution works if the env var exists. The config is correct; the environment setup is missing.

### #291
**Front:** A team builds a generic `execute_sql` tool that accepts any SQL statement. The model accidentally runs `DROP TABLE users`. What went wrong at the tool design level?
**Back:** The generic tool had no constraints — no readonly enforcement, no table whitelist, no destructive operation guard. Replace with constrained tools like `search_users_safe(query)` that only run parameterized SELECT queries.
**Domain:** 2
**Trick:** "The model shouldn't do that" is not a defense. Tool design must prevent it at the code level.

### #292
**Front:** A developer creates 4 review passes for a PR (security, performance, style, correctness). Each pass runs in a separate session. The aggregation step takes 3 of the 4 results. What's wrong?
**Back:** If one pass failed silently, the aggregation is incomplete. The aggregator must handle ALL passes — wait for all results, handle failures explicitly, and report missing passes. Silent partial aggregation produces incomplete reviews.
**Domain:** 1
**Trick:** Best-effort aggregation skips missing data. Accurate reviews require all passes to complete or failures to be reported.

### #293
**Front:** An exam question: "A tool description says 'Use for searching.' A second tool description says 'Use for searching documents.' The model picks the second for all queries, even non-document ones. Why?"
**Back:** The descriptions are overlapping but the second is more specific. The model interprets "documents" broadly because the first description is too vague. Fix: clearly differentiate scope: "Search ALL company data" vs "Search KNOWLEDGE BASE articles only."
**Domain:** 2
**Trick:** Vague descriptions lose to slightly more specific ones. Clear, non-overlapping boundaries fix this.

### #294
**Front:** A long-running agent has been in the same session for 2 hours. It starts making statements like "Based on the three files reviewed..." but it reviewed 15 files earlier. What's happening?
**Back:** Context degradation — the agent has lost access to earlier parts of the conversation. It's only "seeing" the most recent files. The remedy is to persist state and start a fresh session with critical context retained.
**Domain:** 5
**Trick:** "Three files" vs "15 files" is a clear degradation sign. The agent is working with a fraction of the actual context.

### #295
**Front:** A PostToolUse hook normalizes tool output from JSON to Markdown. The model now formats its responses based on the Markdown it receives. Why might this cause inconsistency?
**Back:** The model's output may be influenced by the format of recent tool results. If the hook changes format to Markdown, the model may start outputting Markdown instead of the expected format. The hook should normalize without leaking format cues.
**Domain:** 1
**Trick:** Tool output format influences response format. PostToolUse hooks should normalize silently.

### #296
**Front:** A developer wants to ensure agents never use WebSearch for internal queries. They add "Do not use WebSearch for internal queries" to the system prompt. What's missing?
**Back:** Programmatic enforcement — a PreToolUse hook that blocks WebSearch when the query matches internal patterns. The prompt is probabilistic and may be ignored. Add code-level enforcement for critical constraints.
**Domain:** 1
**Trick:** "I told it not to" is not reliable enforcement. Code-level guards are required for critical constraints.

### #297
**Front:** A Batch API analysis processes 500 customer feedback items. Results show 20% negative sentiment. A synchronous analysis of the same data (sampled) shows 35% negative sentiment. Which is more reliable?
**Back:** The synchronous sample may be more current (if data changes over time), but the batch processed the full dataset. The discrepancy is likely due to the sample not being representative. Check if the synchronous sample was stratified.
**Domain:** 4
**Trick:** Synchronous = fast, batch = comprehensive. Neither is inherently more accurate — methodology and sampling matter.

### #298
**Front:** An agent is configured with tool_choice: "auto." It calls a tool, gets results, and responds without calling more tools. The user wanted more tools to be called for thoroughness. What should change?
**Back:** Switch to `tool_choice: any` — this forces the model to use at least one tool per turn. With `auto`, the model can decide to stop using tools at any point. `any` ensures continued tool usage.
**Domain:** 2
**Trick:** `auto` means "optional tool use." `any` means "must use a tool." The user wanted mandatory tool engagement.

### #299
**Front:** A coordinator spawns 3 subagents. Subagent B crashes. The coordinator receives Subagent A's result, Subagent B's partial result, and Subagent C's result. What should the coordinator do?
**Back:** The coordinator should: (1) log Subagent B's crash with any available context, (2) decide whether Subagent B's task can be skipped, reassigned, or if the entire workflow needs re-evaluation, (3) proceed with available results or trigger escalation.
**Domain:** 1
**Trick:** "Abort everything" and "ignore the failure" are both wrong. Evaluate the criticality of the failed sub-task and decide accordingly.

### #300
**Front:** A developer defines a JSON Schema with `"additionalProperties": false`. The model's output includes an unexpected field `"confidence_score"`. The schema rejects it. What should the developer do?
**Back:** Either add `confidence_score` to the schema as an optional field, or set `additionalProperties: true` if dynamic fields are expected. `additionalProperties: false` is strict — it prevents the model from adding useful extra information.
**Domain:** 4
**Trick:** `additionalProperties: false` creates rigid outputs. Sometimes extra information from the model is valuable — allow it or add the field to the schema.

### #301
**Front:** An exam question about MCP: "You add a new community MCP server. Now your model calls its tools for every query, even when built-in tools would be better. What's the issue?"
**Back:** The community server's tool descriptions overlap with built-in tools and are biased toward selection. Review the descriptions for conflicts, adjust description specificity, or constrain the tool set to only the needed tools from each server.
**Domain:** 2
**Trick:** Adding an MCP server adds tools to the pool. Overlapping descriptions cause cross-server tool confusion.

### #302
**Front:** A CLAUDE.md at project root says "Use tabs for indentation." A CLAUDE.md in `/src/` says "Use spaces for indentation." When editing a file in `/src/api/`, which rule applies?
**Back:** The directory-level CLAUDE.md in `/src/` applies (most specific scope). The project root rule is overridden for files under `/src/`. If there's a `/src/api/CLAUDE.md`, that takes precedence over `/src/CLAUDE.md`.
**Domain:** 3
**Trick:** Scope hierarchy: directory > project > user. The most specific path wins, not the first one loaded.

### #303
**Front:** A code reviewer finds a security vulnerability but is influenced by seeing "no issues found" from a previous review pass in the same session. It downgrades the severity. What principle is violated?
**Back:** Independent review instance principle — the second review should not see the first review's results. This is confirmation bias. Each review pass must run in its own isolated session with no knowledge of other passes.
**Domain:** 4
**Trick:** Context sharing between review passes introduces bias. The model should not know what previous passes found or didn't find.

### #304
**Front:** A team deploys an agent that uses self-rated confidence to decide whether to escalate. The agent escalates 80% of queries because it rates most as "medium confidence." What's wrong?
**Back:** Self-rated confidence is unreliable. The model lacks calibration — it doesn't know what it doesn't know. Use explicit escalation triggers (policy gaps, inability to progress, user request) instead of confidence thresholds.
**Domain:** 5
**Trick:** "Medium confidence" sounds reasonable for escalation. The model uses "medium" for most cases, making the signal useless.

### #305
**Front:** A student creates a JSON Schema with `"severity": { "enum": ["low", "medium", "high", "critical"] }` and `"unclear"` is NOT included. The model encounters an issue it can't classify and outputs `"high"` by default. What's the fix?
**Back:** Add `"unclear"` to the enum. Without it, the model is forced to choose among the available options even when none are appropriate. "Unclear" allows the model to acknowledge ambiguity.
**Domain:** 4
**Trick:** Models will force-fit when the right answer isn't available. "Unclear" gives an honest out.

### #306
**Front:** A developer says: "I always start with planning mode, then switch to execution." An exam asks: Is this correct? Why or why not?
**Back:** This is the reverse of the recommended pattern. Start with direct execution for simple tasks. Switch to planning mode when complexity emerges. Starting with planning for everything wastes tokens on unnecessary analysis.
**Domain:** 3
**Trick:** "Plan first, execute later" is intuitive for humans. For simple AI tasks, direct execution is faster and more efficient.

### #307
**Front:** An agent processes a customer escalation but has context from 3 previous agents who handled the case. The current agent contradicts an earlier agent's correct finding. What likely happened?
**Back:** The context is too large — the earlier correct finding is in the "middle" and lost to the lost-in-the-middle effect. Extract key findings into a case facts block at the start of the context to ensure they're reliably accessible.
**Domain:** 5
**Trick:** Full conversation history is not fully accessible. Key findings need to be extracted and positioned at the start.

### #308
**Front:** A PreToolUse hook is used to validate that file paths are within the allowed directory. The model tries to access `../../etc/passwd`. The hook detects the path traversal and blocks it. What else should the hook do?
**Back:** Return feedback to the model explaining why the path was blocked and suggesting the correct allowed path or asking for clarification. Silent blocks leave the model confused about what to do next.
**Domain:** 1
**Trick:** Blocking is not enough — the model needs to understand why to correct its behavior.

### #309
**Front:** A developer creates a tool `get_customer_data` that returns full PII (name, SSN, DOB, address, payment history). A junior agent uses this to answer "Tell me about this customer" and exposes all data. What tool design principle was violated?
**Back:** Principle of least privilege and data minimization. The tool returns all data regardless of what's needed. Create granular tools: `get_customer_name(id)`, `get_customer_payment_history(id)`, `get_customer_shipping_address(id)` — each returns only what the model specifically asked for.
**Domain:** 2
**Trick:** "Just return everything" is easy but dangerous. Granular tools enforce data minimization.

### #310
**Front:** A student argues: "I should use 'please' and 'thank you' in system prompts to get better model cooperation." Is this effective?
**Back:** Politeness in prompts may have a marginal effect but doesn't substitute for explicit criteria, few-shot examples, and structured output schemas. Focus on clarity and specificity, not manners.
**Domain:** 4
**Trick:** "Be polite to AI" is a debated strategy. Explicit instructions and examples outperform politeness.

### #311
**Front:** An exam question: "Your agent receives 50 tool results throughout a session. It starts making decisions as if it hasn't seen results from the 12th through 30th tool calls. What's likely happening?"
**Back:** Those tool results are in the "middle" of the context and are being lost due to the lost-in-the-middle effect. Critical tool results should be summarized into a persistent block at the start, or earlier tool results should be trimmed to keep the context focused on active information.
**Domain:** 5
**Trick:** Every tool result you keep pushes older results toward the middle. Periodic state extraction prevents critical results from getting buried.

### #312
**Front:** A team builds a document analysis system. The JSON Schema requires `parties: array` and `obligations: array`. The model returns empty arrays for both in a test case where the document has no parties or obligations (it's a memo, not a contract). Are empty arrays correct?
**Back:** Yes — `[]` is a valid response meaning "no items found." This is better than hallucinating non-existent parties or obligations. The schema allowed empty arrays, which is correct for handling diverse document types.
**Domain:** 4
**Trick:** Empty arrays are correct for "none found." Students think empty means failure — it means honest absence.

### #313
**Front:** A developer uses the Batch API for security pre-merge checks. A PR is submitted at 9 AM and the check results arrive at 10 PM. Someone already merged the PR at 2 PM. What went wrong?
**Back:** Batch API SLA doesn't guarantee fast turnaround. Security checks should use synchronous API because they block the merge pipeline. Batch is for non-blocking, non-urgent workloads.
**Domain:** 4
**Trick:** "Batch is cheaper" blinded the team to the SLA requirement. Blocking checks need synchronous processing.

### #314
**Front:** An agent has been running for 3 hours. The developer runs `/memory` to store a key finding. Later, in a new session, the agent cannot recall the stored information. What could be wrong?
**Back:** `/memory` stores information for the user and project scope. If the new session uses a different working directory or context, the memory may not be accessible. Also check that `/memory` was called correctly and the data was actually persisted.
**Domain:** 3
**Trick:** `/memory` isn't magic — it has scope and persistence constraints. Verify the stored data is accessible from the new session.

### #315
**Front:** An exam question: "Your MCP server exposes `search_products`. The tool always returns 100 results. The model frequently runs out of context. What's the fix at the tool level?"
**Back:** Add pagination: `search_products(query, page, page_size)` with a default `page_size: 10`. Also add a `max_results` parameter. The tool should never return unbounded results that can fill the model's context.
**Domain:** 2
**Trick:** 100 results may seem reasonable for one call but 10 calls = 1,000 results. Pagination gives the model control over how much data it receives.

---

## Quick Reference: Cards by Domain

| Domain | Card Range | Count |
|--------|-----------|-------|
| Domain 1: Agent Architecture | #1–#72, #253, #255, #257, #262, #269, #276, #279, #281–#282, #288, #292, #295–#296, #299, #308 | ~85 |
| Domain 2: Tool Design & MCP | #73–#121, #254–#255, #258, #263, #265, #268, #272, #280, #283, #286, #290–#291, #293, #298, #301, #309, #315 | ~50 |
| Domain 3: Claude Code | #122–#165, #256, #271, #277, #285, #289, #302, #306, #314 | ~35 |
| Domain 4: Prompt Engineering | #166–#200, #259–#260, #266–#267, #274, #278, #297, #300, #303, #305, #310, #312–#313 | ~40 |
| Domain 5: Context & Reliability | #201–#252, #261, #264, #270, #273, #275, #284, #287, #294, #304, #307, #311 | ~55 |
| Cross-domain / Mixed | #253–#315 (interleaved) | ~65 |
| **Total** | **#1–#315** | **315** |

---

*Generated for Claude Certified Architect Foundations (CCA-F) exam preparation. Last updated: July 2026.*
