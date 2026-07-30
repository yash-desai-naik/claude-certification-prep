# Exam Traps — Claude Certified Architect Foundations

> The most common wrong-answer patterns. If you see these on the exam, pause and verify.

---

## Trap 1: "Use sentiment analysis to decide escalation"

**The Misconception:** If a user sounds frustrated or negative, escalate to a human.

**Why It Looks Right:** Many support systems use sentiment as an escalation trigger. It's intuitive that an upset customer needs human help.

**Why It's Wrong:** Sentiment does not equal complexity. A user can be angry about a simple problem that the agent can solve perfectly well. Conversely, a user can be calm while describing a deeply complex issue that needs escalation.

**The Correct Answer Pattern:** Escalation should be based on **task complexity** and **agent capability boundaries**, not emotional tone. Check: does the agent have the tools/knowledge to solve this? Is it outside the defined scope?

**Memory Trick:** "Angry ≠ Stuck. A calm user can stump the model; an angry user may just need a password reset."

---

## Trap 2: "Use self-rated confidence for escalation"

**The Misconception:** Ask the model "How confident are you in this answer?" and escalate if confidence is low.

**Why It Looks Right:** Human-like reasoning — if a person says "I'm not sure," you'd escalate.

**Why It's Wrong:** Models can be **confidently wrong** or **diffidently correct**. A model's self-reported confidence has no reliable correlation with actual accuracy. The model cannot meta-evaluate its own correctness in a trustworthy way.

**The Correct Answer Pattern:** Use **validation hooks** or **tool-based verification** (e.g., run the code, check the database, verify against a source of truth) rather than relying on the model's self-assessment.

**Memory Trick:** "Confidence ≠ Competence. The model doesn't know what it doesn't know — and worse, it thinks it does."

---

## Trap 3: "Parse assistant text for completion"

**The Misconception:** Check if the assistant's response ends with a period or looks finished to determine if it's done.

**Why It Looks Right:** When reading model output, it often seems natural to check if it "sounds done."

**Why It's Wrong:** Model text can end mid-sentence or trail off even when the task is complete. Conversely, it can write a full-looking paragraph and still have more to say. Text-based heuristics are unreliable.

**The Correct Answer Pattern:** Always use `stop_reason` from the API response. `"end_turn"` means the model finished. `"tool_use"` means it's calling a tool. `"max_tokens"` means it was cut off.

**Memory Trick:** "Don't guess if it's done — check `stop_reason`. Text lies; the API field doesn't."

---

## Trap 4: "Arbitrary max iterations as stop condition"

**The Misconception:** Set `max_iterations = 10` and stop the workflow when the count is reached.

**Why It Looks Right:** It's a common pattern in coding — bound loops with a max count. Exceeding it means something is wrong.

**Why It's Wrong:** A fixed iteration count is arbitrary. Some tasks finish in 2 iterations; some genuinely need 15. Using `max_iterations` as the stop condition either cuts off legitimate work or runs unnecessarily long.

**The Correct Answer Pattern:** Use `end_turn` (`stop_reason: "end_turn"`) as the natural stop condition. Use `max_iterations` only as a **safety valve** to catch runaway loops, not as the primary completion signal.

**Memory Trick:** "Max iterations = emergency brake, not finish line. Use `end_turn` for that."

---

## Trap 5: "Few-shot examples always better than tool descriptions"

**The Misconception:** Show the model 5 examples of using a tool, and it will understand the tool better than reading its description.

**Why It Looks Right:** Examples are powerful. Few-shot prompting is a well-known technique for improving model behavior.

**Why It's Wrong:** Tool descriptions are **primary** — the model's tool-use mechanism is built around the description schema. Descriptions are what the model processes first to decide which tool to call. Few-shot examples are supplementary and can even cause confusion if they conflict with the description.

**The Correct Answer Pattern:** Write clear, complete tool descriptions first. Add few-shot examples **only** for edge cases or ambiguous usage patterns that the description doesn't cover.

**Memory Trick:** "Description = instruction manual. Examples = demo videos. You need the manual before the demo."

---

## Trap 6: "JSON Schema guarantees correct values"

**The Misconception:** If you define a JSON Schema with `"type": "number"` and `"minimum": 0`, you'll always get a valid positive number.

**Why It Looks Right:** JSON Schema validates structure and types. A number in the output should be a number.

**Why It's Wrong:** JSON Schema and Structured Output guarantee **syntax** (valid JSON, correct types) but NOT **semantics** (correct values). The model can output `"total": 42` when the actual total is 99. Schema cannot ensure the *right* number, only that it *is* a number.

**The Correct Answer Pattern:** Use validation **after** structured output to check semantic correctness — compare against a known source, run business logic checks, or verify ranges specific to the domain.

**Memory Trick:** "Schema checks the shape, not the truth. `"age": 999` is valid JSON and wrong."

---

## Trap 7: "Everything in CLAUDE.md works for all files"

**The Misconception:** Put all instructions in one big CLAUDE.md file at the project root, and the model follows them for every file.

**Why It Looks Right:** CLAUDE.md at the root is the most obvious place. One file to rule them all.

**Why It's Wrong:** Directory-level `.claude/CLAUDE.md` files override the project-root file for files in that directory. Not all instructions apply to all files — tests, configs, and source code have different needs.

**The Correct Answer Pattern:** Use **directory-specific** CLAUDE.md files for specialized rules (e.g., `tests/.claude/CLAUDE.md` for testing conventions). Keep only universal conventions in the root.

**Memory Trick:** "CLAUDE.md cascades like CSS. Specific beats general. Closer to the file wins."

---

## Trap 8: "Put instructions in ~/.claude/CLAUDE.md for team"

**The Misconception:** Put team coding standards in the user-level CLAUDE.md so everyone has them.

**Why It Looks Right:** It's one place, easy to share, and seems like a good default.

**Why It's Wrong:** `~/.claude/CLAUDE.md` is **personal** and not in VCS. It doesn't apply to other team members, isn't tracked, and can't be reviewed. Team conventions belong in the **project-level** `.claude/CLAUDE.md` which is committed to the repo.

**The Correct Answer Pattern:** Team/project instructions → `.claude/CLAUDE.md` (in repo, shared). Personal preferences → `~/.claude/CLAUDE.md` (local only).

**Memory Trick:** "Home dir = your rules. Repo dir = our rules. Don't put shared standards where only you can see them."

---

## Trap 9: "Batch API works for interactive code review"

**The Misconception:** Submit a code review request via Batch API and get results in a few minutes.

**Why It Looks Right:** Batch API is cheaper. If it's fast enough, why not use it for everything?

**Why It's Wrong:** Batch API has **no tool calling** support and **hours-long latency**. Code review requires multi-turn tool-using interactions (read files, check dependencies, run linting). Batch API produces a single text generation per request — no iteration, no tool use.

**The Correct Answer Pattern:** Use Synchronous API for interactive, tool-using workflows like code review. Use Batch API only for **offline processing** that doesn't need tool calls or real-time response.

**Memory Trick:** "Batch = fire and forget, no tools. Code review = tools and iteration. They don't mix."

---

## Trap 10: "Single pass over 14 files works fine"

**The Misconception:** Pass all 14 files in a single prompt. The model will read and process them equally.

**Why It Looks Right:** The model has a 200K context window. 14 files fit easily.

**Why It's Wrong:** The **lost-in-the-middle** effect means the model pays less attention to content in the middle of its context. Processing 14 files in one pass means files 5-10 get significantly less attention than files 1-3 and 13-14. Important details get missed.

**The Correct Answer Pattern:** Process files in **batches of 3-5**, or use a **hierarchical approach**: summarize groups of files, then process based on summaries. Use tools to read strategically, not dump everything at once.

**Memory Trick:** "200K window doesn't mean 200K attention. The middle is a blind spot."

---

## Trap 11: "Same Claude instance should review its own code"

**The Misconception:** After a Claude agent writes code, have the same agent review it for bugs.

**Why It Looks Right:** The agent has full context of what it just wrote. It knows the intent best.

**Why It's Wrong:** **Confirmation bias** — the model is likely to validate its own decisions and miss errors in its own reasoning. A fresh instance or different model approaches the code without preconceptions.

**The Correct Answer Pattern:** Use a **separate review pass** with a new message thread or a different model/temperature. Fresh eyes catch what the builder missed.

**Memory Trick:** "Never let the writer also be the editor. Fresh eyes find bugs that proud parents miss."

---

## Trap 12: "Skills should be in CLAUDE.md"

**The Misconception:** Document skill instructions in CLAUDE.md so the model always knows about them.

**Why It Looks Right:** CLAUDE.md is where instructions go. Skills are instructions.

**Why It's Wrong:** Skills are **loaded on-demand** when the user invokes them via `/skill_name`. Putting skill content in CLAUDE.md wastes context on instructions that aren't always needed. CLAUDE.md is for always-active instructions.

**The Correct Answer Pattern:** Define skills in `.claude/skills/` — they load only when triggered. Keep CLAUDE.md for **always-active** rules and conventions.

**Memory Trick:** "CLAUDE.md = always on. Skills = summon when needed. Don't put the summon-only spell in the always-on spellbook."

---

## Trap 13: "More tools per agent = more capable"

**The Misconception:** Give the agent 20 tools so it can handle any situation.

**Why It Looks Right:** More capabilities should mean a more capable agent.

**Why It's Wrong:** Each additional tool **reduces reliability** in two ways: (1) tool choice becomes harder — the model may pick the wrong tool, and (2) context is diluted — descriptions fight for attention. Beyond ~5-8 tools, reliability degrades noticeably.

**The Correct Answer Pattern:** Give each agent the **minimum set of tools** needed for its specific role. Split agents if you need more capabilities. Quality over quantity.

**Memory Trick:** "10 tools = 10 ways to pick wrong. Give the agent only what it needs, not everything it could use."

---

## Trap 14: "Subagents inherit coordinator context"

**The Misconception:** When a coordinator spawns a worker agent, the worker automatically has all the context from the coordinator.

**Why It Looks Right:** In object-oriented programming, child objects inherit from parent. Agentic workflows feel similar.

**Why It's Wrong:** Each agent instance starts fresh. The coordinator must **explicitly pass** context to the worker — task, constraints, relevant files, format instructions. Nothing is automatic.

**The Correct Answer Pattern:** Always design explicit context-passing between agents. What does the worker need to know? Pass it in the worker's system prompt or first user message.

**Memory Trick:** "Context is not inheritance. You can't telepathically share thoughts between agents. Say it out loud."

---

## Trap 15: "Silently skip errors to avoid interrupting"

**The Misconception:** If an agent encounters an error, silently skip it and continue. Interrupting the user is bad UX.

**Why It Looks Right:** Graceful degradation is a good pattern. Failures shouldn't break the whole experience.

**Why It's Wrong:** Silently skipping errors **masks failures**. The user gets an incomplete result without knowing something went wrong. This erodes trust and makes debugging impossible.

**The Correct Answer Pattern:** Log errors, include context (what failed, why), and either present the partial result with a clear warning or ask the user how to proceed.

**Memory Trick:** "Silent failure = invisible lie. Tell the user what broke, even if it's ugly."

---

## Trap 16: "Abort whole workflow on single failure"

**The Misconception:** If one step fails, the entire workflow is invalid. Abort everything.

**Why It Looks Right:** In traditional software, an unhandled exception aborts the process. Safety first.

**Why It's Wrong:** Agent workflows often have **independent subtasks**. A failure in research of Company A doesn't mean research of Company B is invalid. Aborting everything wastes work and tokens.

**The Correct Answer Pattern:** Continue with **partial results**, clearly marking what succeeded and what failed. Only abort when the failed step is a **hard dependency** for all subsequent steps.

**Memory Trick:** "One rotten apple doesn't spoil the whole batch — unless everything depends on that apple. Check dependencies before aborting."

---

## Trap 17: "General error 'Operation failed' is sufficient"

**The Misconception:** When a tool fails, return a simple error message. The agent just needs to know it failed.

**Why It Looks Right:** Simple means less code, less can go wrong.

**Why It's Wrong:** An agent needs **structured context** to recover from errors. "Operation failed" tells the agent nothing about *why* it failed, *what* was the input, or *how* to retry differently. Without this, the agent will retry with the same input and fail the same way.

**The Correct Answer Pattern:** Return structured errors with: error type (transient/validation/business/permission), input that caused it, stack trace or context, and suggested corrective action.

**Memory Trick:** "Error messages are agent food. 'Something broke' is starvation. 'Failed on line 42: invalid email (got 123-456-7890)' is a feast."

---

## Trap 18: "Retry always helps extraction errors"

**The Misconception:** If extraction fails, retry a few times. Eventually it will succeed.

**Why It Looks Right:** Transient errors resolve on retry. The same logic should apply to extraction.

**Why It's Wrong:** If the information is **simply not present** in the source text, no amount of retrying will produce it. Retrying a hallucination doesn't fix the hallucination — it just generates a different one.

**The Correct Answer Pattern:** First check: is the information actually in the source? If not, don't retry — return "not found." If it is, improve the extraction prompt or use a different strategy.

**Memory Trick:** "Retrying a broken extraction is like re-reading a blank page. The answer isn't there — no number of reads changes that."

---

## Trap 19: "All fields should be required in schema"

**The Misconception:** Mark every field as `"required": true` to ensure a complete output every time.

**Why It Looks Right:** You want the full output. Required fields guarantee completeness.

**Why It's Wrong:** If the information isn't in the source text, a required field **forces the model to hallucinate**. It must put *something* there, so it invents a value. You get complete-but-wrong output.

**The Correct Answer Pattern:** Make fields **required only** when you know the information is always present in the source. Make everything else optional, and handle null values in downstream code.

**Memory Trick:** "Required + missing info = confident lie. Optional + missing info = honest null."

---

## Trap 20: "Mark fields as required to ensure they're filled"

**The Misconception:** Using `"required": true` guarantees the field will have a value.

**Why It Looks Right:** Same as Trap 19. There's a surface-level logic: required means the model must put something there.

**Why It's Wrong:** This is a special case of Trap 19 with a different exam framing. The exam specifically asks about ensuring fields are filled. The answer is: required fields cause **fabrication** when the data isn't available. You get a value, but it's made up.

**The Correct Answer Pattern:** Never use `required` as a mechanism to "force" data. Use it only when data is guaranteed to exist. Use optional + post-processing for everything else.

**Memory Trick:** "Required doesn't create data from nothing. It creates lies from nothing. Same Trap, different angle."

---

## Trap 21: "Put all rules in one big CLAUDE.md"

**The Misconception:** Create one monolithic CLAUDE.md file with every possible instruction.

**Why It Looks Right:** One file is simpler. Everything in one place.

**Why It's Wrong:** (1) Long CLAUDE.md files suffer from the **lost-in-the-middle** effect — rules in the middle get less attention. (2) Not all rules apply to all files. (3) Harder to maintain.

**The Correct Answer Pattern:** Use `.claude/rules/` directory with **modular rule files** for different concerns (e.g., `testing.md`, `typescript.md`, `security.md`). The model loads relevant rules contextually.

**Memory Trick:** "One big book means you lose the middle chapters. Split rules into thin volumes. The model remembers thin books better."

---

## Trap 22: "User skills override project skills by default"

**The Misconception:** The user-level skills directory takes precedence over project-level skills.

**Why It Looks Right:** User-level CLAUDE.md overrides project-level. Same logic should apply.

**Why It's Wrong:** By default, project-level skills and user-level skills are **merged**. Override only happens when a skill in one location has the **same name** as a skill in the other — in that case, the user-level skill wins.

**The Correct Answer Pattern:** Skills are additive across levels. Only same-named skills cause override (user wins). Consider naming conventions to avoid accidental collision.

**Memory Trick:** "No name clash = both load. Name clash = yours wins. It's merge, not replace — unless you share a name."

---

## Trap 23: "System prompt keyword associations don't affect tool choice"

**The Misconception:** The system prompt talking about "database queries" doesn't make the model more likely to call the `query_database` tool.

**Why It Looks Right:** Tool choice is based on tool descriptions. The system prompt is about behavior, not tool selection.

**Why It's Wrong:** The model **associates keywords** in the system prompt with tool descriptions. If the system prompt says "you frequently query the database," the model becomes biased toward calling `query_database` — even when `search_index` would be better. This is called **steering**.

**The Correct Answer Pattern:** Be neutral in system prompt language about tool selection. If you want a specific tool called, prefer `tool_choice: {type: "any"}` or `tool_choice: {type: "tool", name: "specific_tool"}` over keyword bias in text.

**Memory Trick:** "Keywords in system prompt = invisible magnets pulling tool choice. Don't accidentally steer."

---

## Trap 24: "Planning mode is for simple tasks"

**The Misconception:** Planning mode is a lightweight step before execution. Use it for simple, quick tasks.

**Why It Looks Right:** "Plan first" sounds like a simple procedural step.

**Why It's Wrong:** The exam's Planning Mode is specifically designed for **complex architectural decisions**, multi-step workflows, and tasks where the approach isn't immediately obvious. For simple tasks, planning is unnecessary overhead.

**The Correct Answer Pattern:** Use Planning Mode when the task involves: unknown approach, multiple interacting components, architectural decisions, or high risk of wrong approach. Skip planning for obvious, well-understood tasks.

**Memory Trick:** "Simple task → just do it. Complex task → plan first. Don't plan for the obvious; don't skip planning for the hard."

---

## Trap 25: "Context window always works uniformly"

**The Misconception:** Every token in the 200K context window gets equal attention.

**Why It Looks Right:** The context window is advertised as 200K tokens. It should all be accessible.

**Why It's Wrong:** The **lost-in-the-middle** effect is well-documented. The model pays most attention to content at the **beginning** (primacy) and **end** (recency) of its context. Content in the middle gets significantly less attention, leading to missed details.

**The Correct Answer Pattern:** Place the most critical instructions at the **start** (system prompt, key rules) or **end** (most recent user message, latest tool results). Use summarization or hierarchical approaches for large context dumps. Avoid "sandwiching" important content in the middle.

**Memory Trick:** "Context is a hot dog — the meat is in the front and back. The middle is mostly bun and gets ignored."

---

## Trap 26: "Aggregate 97% accuracy means all types are accurate"

**The Misconception:** If the overall accuracy across all extraction types is 97%, the system is 97% accurate for every extraction type.

**Why It Looks Right:** Aggregate metrics are the standard way to report performance.

**Why It's Wrong:** **Stratified sampling** reveals that aggregate metrics can hide massive variance. The system might be 99.9% accurate for emails and 60% accurate for addresses — but because there are 1000x more emails than addresses, the aggregate still looks good.

**The Correct Answer Pattern:** Always measure accuracy **per type/category** when evaluating extraction or classification. Aggregate metrics are misleading for imbalanced categories.

**Memory Trick:** "97% overall hides 60% for rare cases. Aggregate averages lie — stratify to find the truth."

---

## Trap 27: "Start with direct execution, switch to planning when stuck"

**The Misconception:** Begin executing immediately. If you hit a wall, then engage planning mode.

**Why It Looks Right:** "Move fast and iterate." In development, starting is better than over-planning.

**Why It's Wrong:** Once the model starts executing (especially with side effects like file writes or API calls), **fixing a wrong approach costs more** than planning upfront. The damage is done. The exam explicitly identifies this as an anti-pattern.

**The Correct Answer Pattern:** Plan first for complex tasks. Direct execution is for tasks where the approach is obvious and the cost of wrong execution is low.

**Memory Trick:** "Plan before you leap. Building on the wrong foundation is more expensive than taking time to draw the blueprint."

---

## Trap 28: "Pre-merge checks can use Batch API"

**The Misconception:** Use Batch API for pre-merge checks because it's cheaper.

**Why It Looks Right:** Batch API offers 50% discount. Pre-merge checks are non-interactive. Seems like a perfect fit.

**Why It's Wrong:** Pre-merge checks are **blocking** — developers wait for results before merging. Batch API has **hours-long latency**, which kills developer productivity. Additionally, many pre-merge checks (code review, test generation) need **tool calling**, which Batch API doesn't support.

**The Correct Answer Pattern:** Use **Synchronous API** for any blocking workflow where the developer is waiting. Batch API is for offline, non-blocking, non-tool-calling workloads only.

**Memory Trick:** "Waiting for Batch = waiting for paint to dry. Pre-merge needs speed. Batch is for overnight jobs."

---

## Trap 29: "Proceed with most recently stated preference on contradiction"

**The Misconception:** When a user contradicts themselves, assume the most recent statement is the correct intent.

**Why It Looks Right:** In many systems, the latest input takes precedence. "Last write wins" is a common pattern.

**Why It's Wrong:** User contradictions could be: a change of mind (OK to use latest), a clarification (latest is more precise), or an **error** (latest is a mistake). Assuming latest is always correct can silently overwrite the user's real intent.

**The Correct Answer Pattern:** When you detect a contradiction, **ask for clarification** rather than assuming. "You mentioned X earlier, but now you're saying Y. Which is correct?"

**Memory Trick:** "Last word isn't always truth. When user contradicts, stop and ask — don't silently assume."

---

## Trap 30: "Summarization preserves all critical details"

**The Misconception:** When you summarize a conversation to save context, all critical details (numbers, dates, specific constraints) are preserved.

**Why It Looks Right:** A summary is supposed to capture the essential information. That's the whole point.

**Why It's Wrong:** Model-generated summaries **inevitably lose precision** — especially around numbers, dates, names, and specific thresholds. The model tends to generalize ("a few hours ago" instead of "at 14:32 UTC", "a significant amount" instead of "$142,893").

**The Correct Answer Pattern:** For critical details, use **structured data** (JSON records, database rows) alongside or instead of natural language summaries. Preserve exact values in a format that doesn't compress or approximate.

**Memory Trick:** "Summaries get the plot right but forget the date and dollar amount. For precision, store structured, not summarized."

---

## Quick Trap Reference

| # | Trap Name | One-Liner |
|---|---|---|
| 1 | Sentiment Escalation | "Angry ≠ Complex. Escalate on capability, not emotion." |
| 2 | Self-Rated Confidence | "Confidence ≠ Accuracy. The model doesn't know what it doesn't know." |
| 3 | Parse Text for Completion | "Don't parse text — check `stop_reason`. Text lies." |
| 4 | Max Iterations Stop | "Max iterations = emergency brake, not finish line." |
| 5 | Few-shot > Descriptions | "Description = manual. Examples = demos. Manual first." |
| 6 | JSON Schema Guarantee | "Schema checks shape, not truth. `"age": 999` is valid and wrong." |
| 7 | One CLAUDE.md for all | "CLAUDE.md cascades. Specific dir beats root." |
| 8 | ~/.claude for team | "Home dir = personal. Repo dir = team. Don't mix." |
| 9 | Batch for code review | "Batch = no tools + hours. Code review needs both." |
| 10 | Single pass 14 files | "Middle of context = blind spot. Batch files." |
| 11 | Self code review | "Writer ≠ editor. Fresh instance finds bugs." |
| 12 | Skills in CLAUDE.md | "Skills = on-demand. CLAUDE.md = always on." |
| 13 | More tools = better | "5 tools you use > 20 tools you confuse." |
| 14 | Subagents inherit context | "No psychic inheritance. Pass context explicitly." |
| 15 | Silent skip errors | "Silent failure = invisible lie. Surface errors." |
| 16 | Abort on single failure | "Partial results beat all-or-nothing. Check dependencies." |
| 17 | "Operation failed" | "Agent needs error context. Feed structured errors." |
| 18 | Retry extraction always | "Missing info ≠ transient error. Check source first." |
| 19 | All fields required | "Required + missing = hallucination. Optional = honest null." |
| 20 | Required = filled | "Same as 19. Required forces fabrication when data absent." |
| 21 | One big CLAUDE.md | "Modular rules beat monolithic. Middle gets lost." |
| 22 | User skills override all | "Same name → override. Different name → merge." |
| 23 | Keywords don't steer | "Keywords = invisible magnets for tool choice." |
| 24 | Planning for simple tasks | "Plan for complex. Execute simple. Don't invert." |
| 25 | Uniform context window | "Context = hot dog. Front and back get attention." |
| 26 | Aggregate accuracy | "Stratify per type. 97% overall hides 60% for rare cases." |
| 27 | Execute first, plan later | "Plan before you build. Wrong foundation costs more." |
| 28 | Batch for pre-merge | "Pre-merge = blocking. Batch = hours. Wrong fit." |
| 29 | Latest preference wins | "Contradiction → ask. Don't assume latest is correct." |
| 30 | Summarization preserves | "Summaries lose numbers/dates. Use structured for precision." |

---

*Last updated for the Claude Certified Architect Foundations exam.*
