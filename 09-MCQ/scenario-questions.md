# Scenario-Based Architecture Questions

**Total Questions:** 30  
**Format:** Long-form scenarios requiring architectural reasoning

---

## Scenario #1: Multi-Agent Research System with Conflicting Sources

**Scenario:**

You are designing a multi-agent research system that will investigate complex topics by consulting multiple sources. The system has:

- A **coordinator agent** that decomposes research questions into sub-topics
- **5 research sub-agents** that each investigate one sub-topic
- A **synthesis agent** that combines findings into a final report
- A **fact-checking tool** with a rate limit of 10 calls per minute

During testing, the system encounters a significant problem. For a question about climate change impacts on agriculture, two sub-agents return findings that directly contradict each other. Sub-agent A's sources say global warming is reducing crop yields. Sub-agent B's sources say CO₂ fertilization is increasing crop yields. The synthesis agent's first attempt at resolution is to average the two positions, producing a report that says "crop yields may be slightly affected."

A reviewer identifies that the synthesis lost the nuance of both positions and didn't properly reflect the scientific debate. Additionally, the synthesis failed to cite which sources supported which position.

**Question:** What is the most effective redesign of the synthesis agent's approach?

**Domain:** 1, 5  
**Difficulty:** Hard  

A) Have the synthesis agent always choose the position supported by more sources (majority voting approach).

B) Redesign the synthesis agent to: (1) identify and flag all contradictions explicitly, (2) present each position with full source attribution and confidence levels, (3) explain the reasoning behind each position including methodological differences in sources, (4) note when the evidence doesn't support a single conclusion, and (5) include an "evidence map" showing which claims are supported by which sources.

C) Remove the conflicting sub-agents and only use sources that agree with each other.

D) Have a human reviewer resolve all contradictions before the synthesis agent proceeds.

**Correct Answer:** B  
**Explanation:** The synthesis agent should embrace contradictions rather than suppress them. The correct approach is to map the landscape of evidence — presenting all positions with their supporting sources, noting methodological differences (e.g., different time periods, geographies, methodologies), and transparently indicating where evidence is conflicting. This preserves nuance and provides the user with a complete picture.

**Why Others Wrong:**
- A: Majority voting oversimplifies complex scientific debates where the minority position may be valid.  
- B: Correct. Full evidence mapping with attribution and explanation of contradictions.  
- C: Cherry-picking sources to avoid contradiction produces biased results.  
- D: Too slow and doesn't scale; the system should handle contradictions algorithmically.

**Exam Objective:** Design synthesis agents that handle conflicting evidence with transparency and nuance.

---

## Scenario #2: Customer Support Agent with Multi-Issue Requests

**Scenario:**

You are designing an AI customer support agent for an e-commerce platform. The agent has access to tools for order lookup, refund processing, product information, and account management.

A customer sends the following message:

"My order #12345 arrived yesterday but the blue sweater is missing. Also, I want to return the red shoes because they don't fit, and can you tell me when the black jeans will be back in stock?"

The agent processes this by:
1. Looking up order #12345
2. Seeing the items in the order
3. Processing a return for the red shoes
4. Telling the customer about sweater unavailability
5. Saying it doesn't know about restock dates

The customer responds frustrated: "You only handled half my issues! You didn't actually resolve the missing sweater."

**Question:** What architectural pattern would best handle this multi-issue scenario?

**Domain:** 1  
**Difficulty:** Medium  

A) Implement a confirmation step before any action: "I see you have 3 requests. Shall I handle them one at a time?"

B) Design a **decomposition and tracking pattern**: (1) the coordinator agent first extracts and catalogs ALL distinct issues from the message, (2) assigns each issue an explicit status (pending/in-progress/resolved/needs-escalation), (3) processes each issue with appropriate tools, (4) before concluding, reviews the issue list to ensure no items remain unresolved, and (5) provides a structured summary of what was done for each issue.

C) Have the agent ask the customer to submit each issue separately.

D) Prioritize issues by order of mention and only process the first one thoroughly.

**Correct Answer:** B  
**Explanation:** Multi-issue requests require explicit issue tracking. The agent should first decompose the message into individual issues, track each through to resolution, and verify completeness before concluding. A confirmed checklist pattern ensures nothing is missed and provides transparency about what was done for each issue.

**Why Others Wrong:**
- A: Asking for permission adds friction; the agent should be capable of handling multiple issues.  
- B: Correct. Issue decomposition + tracking + verification.  
- C: Frustrating user experience; the agent should handle complex requests.  
- D: Ignores valid customer requests.

**Exam Objective:** Implement issue decomposition and tracking for multi-request customer interactions.

---

## Scenario #3: CI/CD Pipeline with Batch API Decision

**Scenario:**

Your team is building a CI/CD pipeline that uses an LLM API to perform automated code reviews on every pull request. The pipeline runs approximately 200 times per day. You need to decide between two API strategies:

**Approach A — Synchronous:** For each file in the PR, make a separate API call. This means faster feedback per file (results stream in as each call completes), but the total pipeline time depends on the number of files.

**Approach B — Batch:** Collect all files from the PR, submit them as a single batch API request, and poll for results. This means the pipeline submits the batch and waits for all results to be ready.

The team is debating which approach to use. Your analysis reveals that PRs have a median of 5 files but can have up to 50 files, and the API has a 1-minute timeout per synchronous call.

**Question:** Which approach is more appropriate for this use case, and what additional design considerations should be addressed?

**Domain:** 2, 4  
**Difficulty:** Medium  

A) Approach A (synchronous) is fine because most PRs are small. For large PRs, just increase the timeout.

B) Approach B (batch) is more reliable because it handles any PR size consistently. However, the design should include: (1) per-file custom IDs for granular error handling, (2) a maximum batch size with chunking for very large PRs, (3) a polling mechanism with timeout, and (4) a fallback to synchronous processing for urgent/small PRs where low latency matters.

C) Neither approach works for CI; use a locally-hosted model instead.

D) Use synchronous for code review and batch for all other pipeline tasks.

**Correct Answer:** B  
**Explanation:** Batch processing is more appropriate because it handles the wide range of PR sizes consistently. However, a hybrid design is optimal — batch for normal operation, with recommendations for per-file custom IDs (for identifying which file reviews failed), maximum batch sizes, and fallback strategies. The synchronous approach risks timeouts on large PRs.

**Why Others Wrong:**
- A: 50 files × 1 minute = 50 minutes of CI time, which is unacceptable.  
- B: Correct. Batch with proper error handling and hybrid fallback.  
- C: Local models may not match API quality; the question is about batch vs sync.  
- D: Code review benefits from the same batch reasoning as other tasks.

**Exam Objective:** Choose between batch and synchronous API strategies based on workload characteristics.

---

## Scenario #4: Claude Code Planning vs Direct Execution

**Scenario:**

A developer is working on a complex refactoring task: renaming a core database column from `user_name` to `full_name` and updating all 47 references across the codebase, including SQL queries, ORM models, API endpoints, tests, and documentation.

The developer considers two approaches in Claude Code:

**Approach A — Direct Execution:** Tell Claude Code to make all the changes in a single session, relying on it to find and update all references correctly.

**Approach B — Plan-Then-Execute:** First ask Claude Code to analyze the codebase and produce a detailed migration plan listing every file and reference that needs to change. Review the plan, then execute it.

**Question:** What is the correct architectural approach for this task?

**Domain:** 3, 1  
**Difficulty:** Medium  

A) Approach A — direct execution is faster and Claude Code is smart enough to handle it.

B) Approach B — plan-then-execute. The developer should: (1) have Claude Code search the codebase comprehensively and produce a migration plan, (2) review the plan to catch missed references, (3) refine the search if needed, (4) execute the changes with the refined plan, (5) run tests to verify. This catches the inevitable missed edge cases that a single-pass approach would miss.

C) Do it manually since AI isn't reliable enough.

D) Use Approach A but in a separate git branch so changes can be reverted.

**Correct Answer:** B  
**Explanation:** Plan-then-execute is the recommended architecture for complex, multi-file refactoring. The initial search may miss some references (aliased imports, dynamic queries, generated files). Having a plan that the developer reviews catches these gaps. The plan also serves as a checklist for verification. Direct execution risks missed references that aren't discovered until tests fail.

**Why Others Wrong:**
- A: Single-pass direct execution misses references and provides no review opportunity.  
- B: Correct. Plan-then-execute with human review.  
- C: AI-assisted refactoring is reliable with proper process; the issue is approach.  
- D: A separate branch helps but doesn't fix the missed-reference problem.

**Exam Objective:** Apply plan-then-execute architecture for complex refactoring tasks.

---

## Scenario #5: MCP Tool Error Handling Design

**Scenario:**

You are designing an MCP server that provides customer data to an agent. The server wraps three separate internal APIs: CRM (customer info), Billing (payment history), and Support (ticket history).

During testing, the following failure modes are observed:

1. The CRM API sometimes returns `null` for valid customers who have incomplete profiles
2. The Billing API returns HTTP 503 when under load
3. The Support API returns HTML error pages instead of JSON when it fails
4. The combined tool call takes 45 seconds when all three APIs are slow

The agent currently receives whatever the MCP server returns and has to interpret these varied error conditions on its own.

**Question:** What is the most robust tool error handling design?

**Domain:** 2  
**Difficulty:** Medium  

A) Let the raw API responses pass through to the agent — the model is smart enough to handle them.

B) Design the MCP server to: (1) normalize all responses into a consistent structured format with explicit success/error fields, (2) use `isError: true` for all error conditions with structured error codes and messages, (3) implement per-API timeouts (10s each) with graceful degradation (partial results with error indicators), (4) convert HTML errors to structured error responses, and (5) distinguish between "not found" (valid null) and "error" (system failure).

C) Have the server retry automatically 5 times before returning any error.

D) Return a generic "An error occurred" message for all failure types to keep things simple.

**Correct Answer:** B  
**Explanation:** The MCP server should normalize all API responses into a consistent format. This includes: structured error signals (`isError`), per-API timeouts with grace, normalization of different error formats, and clear distinction between valid empty results and system failures. This makes error handling predictable for the consuming agent.

**Why Others Wrong:**
- A: Raw, varied error formats confuse the agent and lead to inconsistent handling.  
- B: Correct. Normalized, structured error handling with graceful degradation.  
- C: Indiscriminate retrying wastes time and doesn't fix the normalization issue.  
- D: Overly lossy; the agent and developer can't diagnose failures.

**Exam Objective:** Design MCP servers with normalized, structured error handling for agent consumption.

---

## Scenario #6: Context Management in Long Conversations

**Scenario:**

You are building a legal document review agent that helps lawyers analyze contracts. A typical session involves:

- Uploading a 50-page contract
- The agent reading and analyzing the full document
- The lawyer asking 40-50 follow-up questions about specific clauses
- The lawyer referencing clauses mentioned 30 turns ago ("Remember what Section 12.3 said about indemnification?")
- The lawyer making annotations that should persist across the session
- The session lasting 2+ hours with continuous back-and-forth

During testing with real lawyers, the agent starts forgetting details from earlier parts of the conversation after about 20 turns. The lawyers get frustrated when the agent can't recall specific clauses they discussed earlier.

**Question:** What is the most effective context management architecture for this use case?

**Domain:** 1, 3  
**Difficulty:** Hard  

A) Use a model with a large context window (200K tokens) to fit the entire conversation and document.

B) Implement a **multi-tier context architecture**: (1) a persistent "case state" block that's always in context — updated after every turn with key findings, clause references, annotations, and decisions, (2) the original contract analysis stored in a structured format (per-clause summaries), (3) a sliding window of recent turns for immediate context, (4) a retrieval mechanism that fetches specific earlier turns or clauses when referenced, and (5) periodic summarization of older conversation turns into the case state.

C) Ask users to rephrase questions to include full context each time.

D) Store everything in a vector database and retrieve relevant chunks for each query.

**Correct Answer:** B  
**Explanation:** Legal document review needs a sophisticated multi-tier architecture. The case state block (compact, always in context) captures key references. The structured clause summaries provide quick reference. A retrieval mechanism fetches details when specific clauses are mentioned. This beats relying on a single large context window because: (a) the model doesn't handle 200K tokens equally, (b) retrieval is more reliable than hoping the model remembers, and (c) the case state provides a compact yet comprehensive summary.

**Why Others Wrong:**
- A: Large context windows still suffer from "lost in the middle" effects and don't guarantee recall.  
- B: Correct. Multi-tier architecture with persistent state, structured summaries, and retrieval.  
- C: Terrible UX; defeats the purpose of an AI assistant.  
- D: Vector retrieval alone misses the structured nature of legal analysis; need both state and retrieval.

**Exam Objective:** Design multi-tier context management for long-running, detail-intensive agent sessions.

---

## Scenario #7: Escalation Calibration

**Scenario:**

You are designing an escalation system for a customer support agent. The agent handles Tier 1 support (basic questions, order status, returns). When the agent can't resolve an issue, it should escalate to a human agent.

Initial design: "If you can't resolve the issue, escalate to a human."

In testing, two problems emerge:

**Problem 1: Over-escalation.** The agent escalates for minor issues it could handle — "customer asked about shipping times" gets escalated because the agent isn't sure about the exact policy.

**Problem 2: Under-escalation.** The agent tries to handle complex account issues it's not equipped for — "customer's account was hacked" results in the agent trying to reset passwords instead of escalating to security.

**Question:** What escalation design would best address both problems?

**Domain:** 5, 1  
**Difficulty:** Medium  

A) Remove human escalation entirely; let the agent handle everything.

B) Implement **explicit escalation criteria with a structured decision framework**: (1) define specific escalation triggers (security issues, account compromise, legal requests, known system bugs, refund amounts over threshold, customer explicitly asks for human), (2) define "handle confidently" categories (order status, basic returns, shipping info, product details), (3) provide few-shot examples of borderline cases with correct escalation decisions, (4) include a confidence check: "If you're below 80% confidence in your ability to resolve, escalate," and (5) require the agent to provide a structured escalation summary (issue, what was tried, what's needed).

C) Require a human supervisor to review every escalation decision.

D) Escalate after any customer shows frustration (sentiment-based escalation).

**Correct Answer:** B  
**Explanation:** Explicit, specific escalation criteria calibrate the agent by defining clear boundaries. The agent should know exactly what it can handle confidently and what must be escalated. Confidence thresholds and structured escalation summaries improve decision quality and handoff efficiency. The few-shot examples help the agent learn the boundary between "resolve" and "escalate."

**Why Others Wrong:**
- A: Removing escalation means the agent handles things it shouldn't, creating risk.  
- B: Correct. Explicit criteria with examples and confidence threshold.  
- C: Doesn't scale; human review of every escalation defeats the purpose of automation.  
- D: Sentiment-based escalation is unreliable and may escalate unnecessarily.

**Exam Objective:** Calibrate agent escalation decisions with explicit criteria and few-shot examples.

---

## Scenario #8: Tool Misrouting Diagnosis

**Scenario:**

You are debugging a sales assistant agent that keeps using the wrong tools. The agent has the following tools:

1. `get_product_info(product_id)` — Returns product details
2. `check_inventory(product_id)` — Returns stock levels
3. `get_customer_info(customer_id)` — Returns customer details  
4. `lookup_discount(customer_id, product_id)` — Returns available discounts
5. `create_order(customer_id, items)` — Creates a new order
6. `get_order_status(order_id)` — Returns order status

In testing, when a user says "I'd like to buy a widget," the agent calls `get_product_info` for the widget, then instead of calling `check_inventory` and `lookup_discount`, it calls `get_order_status` (which fails because there's no order yet) and then gets confused.

When a user says "What's the price for a widget with my account discount?", the agent calls `get_product_info` but not `lookup_discount`.

**Question:** What diagnostic process would best identify the root cause, and what is the likely fix?

**Domain:** 2, 4  
**Difficulty:** Medium  

A) The agent needs more powerful hardware — upgrade the compute resources.

B) The diagnostic process should: (1) review tool descriptions for clarity and completeness — are they clearly describing when each tool should be used and what it requires?, (2) check if any tools have overlapping or ambiguous descriptions — `get_order_status` sounds like it could be a general-purpose "get something" tool, (3) verify that each tool's parameter descriptions are clear, (4) examine if the number of tools (6) is approaching the reliability threshold. The likely fix: redesign tool descriptions to be more specific about preconditions and use cases, and consider whether some tools should be grouped or aligned differently.

C) Replace all tools with one `execute_api(endpoint, params)` tool.

D) Train the agent more by showing it 10,000 example conversations.

**Correct Answer:** B  
**Explanation:** Tool misrouting is almost always caused by tool design issues. The diagnostic should start with tool descriptions — are they specific enough? Do any tools sound like they could do similar things? In this case, `get_order_status` may sound to the agent like a general "get info about something" tool. Descriptions should be explicit about preconditions: `get_order_status(order_id)` should say "Only use after an order has been created. Requires an existing order ID."

**Why Others Wrong:**
- A: Hardware doesn't fix tool routing.  
- B: Correct. Systematic tool description review.  
- C: A single generic tool makes routing harder, not easier.  
- D: Overkill; the issue is tool design, not training.

**Exam Objective:** Diagnose tool misrouting through systematic tool description review.

---

## Scenario #9: Subagent Error Propagation

**Scenario:**

You are designing a multi-agent financial analysis system with the following architecture:

- **Coordinator agent** — decomposes the analysis request and manages sub-agents
- **Market Data Agent** — fetches current market data
- **Historical Analysis Agent** — analyzes historical trends
- **Risk Assessment Agent** — evaluates portfolio risk
- **Report Generation Agent** — produces the final report

During a run, the Market Data Agent encounters an API rate limit error after 3 calls. It tries to continue but produces incomplete data. The coordinator doesn't detect this and passes the incomplete data to the Risk Assessment Agent, which produces an inaccurate risk score. The final report contains incorrect risk metrics.

The team discovers the issue only when a human analyst reviews the report and notices the numbers don't match the market data.

**Question:** What error propagation and handling architecture should be implemented?

**Domain:** 1, 2  
**Difficulty:** Hard  

A) Each sub-agent should include a structured status and confidence field with its output, and the coordinator should check these fields before passing data downstream.

B) Implement a **structured error propagation framework**: (1) every sub-agent returns a structured response with `status` (complete/partial/failed), `output` (actual data), `errors` (array of error objects with type, severity, details), `confidence` (per-output-field), and `metadata` (calls made, sources used), (2) the coordinator checks all sub-agent responses before passing data, (3) if a critical sub-agent fails, the coordinator determines whether to retry, use partial results with caveats, or abort, (4) downstream agents check input quality before processing, (5) the final report includes a "data quality" section documenting any issues in source data.

C) Make all sub-agents retry indefinitely until they get complete data.

D) Have the coordinator run all sub-agents twice and compare results.

**Correct Answer:** B  
**Explanation:** Robust error propagation requires structured metadata flowing through every stage. Each sub-agent reports its quality status, and the coordinator (and downstream agents) use this information to make decisions. This is more reliable than expecting the coordinator to detect incomplete data by examining the content, which it cannot reliably do for unfamiliar domains.

**Why Others Wrong:**
- A: A simple status field is a good start but doesn't provide enough granularity for downstream agents to make informed decisions.  
- B: Correct. Full structured error context propagates through the pipeline.  
- C: Indefinite retries waste resources and may never succeed for persistent errors.  
- D: Doubles cost without guaranteeing completeness.

**Exam Objective:** Design structured error propagation with quality metadata flowing through multi-agent pipelines.

---

## Scenario #10: Provenance Preservation in Synthesis

**Scenario:**

You are building a medical research synthesis agent that helps doctors quickly understand the latest findings. The agent:

1. Searches medical literature databases
2. Reads and extracts key findings from 20-30 papers
3. Synthesizes findings into a concise briefing

A doctor using the system reads a synthesized statement: "Treatment X shows a 30% improvement in patient outcomes compared to placebo." The doctor needs to verify this claim — which paper, which population, what time frame, and under what conditions was this measured. The current system just provides the synthesized text without source mappings.

The doctor says: "I can't use this in clinical decision-making if I can't verify where the information comes from."

**Question:** What provenance architecture should be implemented?

**Domain:** 1, 5  
**Difficulty:** Medium  

A) Add a bibliography at the end of the briefing listing all papers used.

B) Implement a **claim-level provenance system**: (1) each extraction step tags every claim with its source paper ID, page number, and methodology context, (2) the synthesis step preserves these tags and maps them to output claims, (3) the final output includes inline citations linking each claim to specific sources, (4) a references section provides full citation details, (5) a "confidence per claim" indicator based on source quality (RCT vs. case study, sample size, statistical significance), (6) the ability for users to click/tap a claim and see the source excerpt.

C) Have a human annotator add citations to each report after generation.

D) Use a larger model that will "remember" where information came from.

**Correct Answer:** B  
**Explanation:** Medical decision-making requires claim-level provenance — not just a list of sources, but explicit mapping from each claim to its supporting evidence. The architecture requires tracking provenance through the extraction and synthesis pipeline. Each claim retains its source identifier and confidence context, enabling verification.

**Why Others Wrong:**
- A: Bibliographies list sources but don't map claims to sources.  
- B: Correct. Claim-level provenance with inline citations and confidence indicators.  
- C: Doesn't scale to large volumes and delays time-sensitive briefings.  
- D: Models cannot reliably recall source assignments without explicit tracking.

**Exam Objective:** Implement claim-level provenance tracking in synthesis pipelines for high-stakes domains.

---

## Scenario #11: Handling Ambiguous MCP Error Responses

**Scenario:**

Your team builds an MCP server that wraps a payment processing API. The API can fail in several ways:

- Insufficient funds (the API returns `{"error": "declined"}`)
- Invalid card (the API returns `{"error": "declined"}` — same message)
- Fraud detection block (the API returns `{"error": "declined"}` — same message)
- Rate limit exceeded (the API returns HTTP 429)
- Temporary system error (the API returns HTTP 502)

The MCP server passes all responses directly to the agent. The agent treats all `{"error": "declined"}` responses the same way — it tells the customer "your card was declined" — even when the actual reason is a temporary system error or rate limit (which should be retried) or a fraud block (which requires customer verification).

**Question:** How should the MCP server be redesigned to provide actionable error information?

**Domain:** 2  
**Difficulty:** Medium  

A) Map each distinct error condition to a structured response with a unique error code, clear message, and recommended action. Use the `isError` field. Include retry guidance where applicable.

B) Tell the agent to "read the error text more carefully to determine the actual cause."

C) Return the same generic "payment failed" response for all errors to keep the design simple.

D) Have the MCP server retry automatically on all errors before returning to the agent.

**Correct Answer:** A  
**Explanation:** The MCP server should translate diverse, ambiguous API error responses into structured, unambiguous error codes. Each error type should have a distinct code (e.g., `INSUFFICIENT_FUNDS`, `FRAUD_BLOCK`, `RATE_LIMITED`, `SYSTEM_ERROR`) with appropriate metadata. The agent can then take the correct action for each type — retry for rate limits, notify for fraud blocks, suggest alternatives for insufficient funds.

**Why Others Wrong:**
- B: The API returns identical responses for different conditions; the agent can't distinguish them.  
- C: Lossy; hides valuable diagnostic information from the agent.  
- D: Retries don't help with declined transactions and add latency.

**Exam Objective:** Design MCP servers that translate ambiguous API errors into structured, actionable responses.

---

## Scenario #12: Agent with Memory of User Preferences

**Scenario:**

You're building a personal shopping assistant agent that helps users find and purchase products. The agent should remember user preferences across sessions:

- Session 1: User says "I prefer organic products"
- Session 2 (next day): User says "Find me a moisturizer"

The agent should apply the organic preference from Session 1. After 10 sessions spanning 3 months, the user should still benefit from accumulated preference knowledge.

The system currently has no cross-session memory. Each session starts fresh. The user has to repeat preferences every time.

**Question:** What memory architecture should be implemented?

**Domain:** 1, 3  
**Difficulty:** Medium  

A) Store the entire conversation history from all sessions and prepend it to every new session.

B) Implement a **structured user profile system**: (1) a dedicated user profile store with categories (product preferences, communication preferences, size/fit info, past purchases, feedback history), (2) an agent tool `update_user_preference(category, key, value)` that the agent calls when it learns something, (3) a tool `get_user_profile(user_id)` that returns the structured profile at the start of each session, (4) a review step at session end where the agent updates any new preferences learned, (5) time-based decay for preference confidence (explicitly stated preferences weighted more than inferred ones).

C) Ask the user to restate their preferences at the start of each session.

D) Store preferences in the agent's system prompt.

**Correct Answer:** B  
**Explanation:** Structured user profiles separate from conversation history are the correct approach. The agent has explicit tools to read and update the profile. This is more reliable than prepending history (which wastes context and suffers from position bias) and more maintainable than system prompt hacks.

**Why Others Wrong:**
- A: Conversation history grows unbounded and suffers from lost-in-the-middle effects.  
- B: Correct. Structured profile with explicit tools.  
- C: Bad UX; makes the agent seem forgetful.  
- D: System prompts can't be dynamically updated per session.

**Exam Objective:** Design structured cross-session memory with explicit profile management tools.

---

## Scenario #13: Agent Hallucination Detection

**Scenario:**

Your team builds an agent that generates technical documentation from codebases. The agent reads source code, analyzes function signatures, and generates documentation.

A user reports that the documentation for a function `calculateRisk()` includes a claim: "This function uses the Monte Carlo simulation algorithm with 10,000 iterations" — but the actual code uses a simple standard deviation calculation. The agent hallucinated the Monte Carlo detail.

You need to implement a system that detects and prevents such hallucination in documentation outputs.

**Question:** What hallucination detection architecture is most effective?

**Domain:** 4, 1  
**Difficulty:** Medium  

A) Add a prompt telling the agent "Don't hallucinate" before documentation generation.

B) Implement a **verification loop**: (1) the documentation agent generates initial docs, (2) a separate verification agent compares each factual claim against the source code, (3) claims that can't be verified are flagged, (4) unverified claims trigger re-generation with explicit citation requirements, (5) if verification still fails, the claim is either removed or marked as "unverified" in the output.

C) Use a larger model that hallucinates less.

D) Have a human review every documentation page before publishing.

**Correct Answer:** B  
**Explanation:** A separate verification agent provides independent fact-checking. The key is independence — the verification agent should not have seen the generation process, should approach the code fresh, and should check each claim. Claims that can't be verified from code are either corrected or flagged. This catches the specific type of hallucination where the agent "fills in" plausible-sounding but incorrect technical details.

**Why Others Wrong:**
- A: Prompt-level instructions are insufficient guardrails.  
- B: Correct. Independent verification loop catches hallucinations.  
- C: Larger models still hallucinate; size reduces but doesn't eliminate it.  
- D: Doesn't scale and defeats the purpose of automation.

**Exam Objective:** Implement independent verification loops for hallucination detection in agent outputs.

---

## Scenario #14: Tool Access Control Design

**Scenario:**

You are designing a customer service agent for a bank. The agent has access to these tools:

- `get_account_balance(account_id)` — View balance
- `transfer_funds(from_account, to_account, amount)` — Transfer money
- `get_transaction_history(account_id, days)` — View transactions
- `report_lost_card(card_id)` — Report card as lost
- `update_contact_info(customer_id, phone, email)` — Update contact details
- `apply_for_overdraft(account_id, limit)` — Apply for overdraft

The bank has different tiers of customer service representatives. Tier 1 agents should only be able to view information and report lost cards. Tier 2 agents should additionally be able to do transfers and update contact info. Only specialized agents should handle overdraft applications.

**Question:** How should tool access control be architected for this multi-tier system?

**Domain:** 2, 5  
**Difficulty:** Medium  

A) Give all agents access to all tools and rely on the prompt to restrict usage.

B) Implement **role-based tool access**: (1) define agent tiers (Tier 1, Tier 2, Specialist), (2) create separate agent configurations per tier, each with only the tools permitted for that tier, (3) implement authentication at the MCP server level that validates the agent's role before executing any tool, (4) log all tool usage for audit, (5) escalate requests that require higher-tier tools to the appropriate agent tier instead of trying to handle them with available tools.

C) Give all agents access but add a confirmation dialog before sensitive operations.

D) Make the agent ask for manager approval before using any tool.

**Correct Answer:** B  
**Explanation:** Role-based tool access ensures security by construction. Each agent tier only has the tools it needs (principle of least privilege). MCP-level authentication prevents even a compromised or misconfigured agent from accessing tools it shouldn't. This is far more secure than relying on prompt-level restrictions which can be overridden.

**Why Others Wrong:**
- A: Prompt-level restrictions are unreliable and can be bypassed.  
- B: Correct. Role-based access at the tool configuration level.  
- C: Confirmation dialogs slow operations and can be mindlessly approved.  
- D: Manager approval for every tool use is impractical.

**Exam Objective:** Implement role-based tool access control at the configuration and MCP server level.

---

## Scenario #15: Handling PII in Agent Conversations

**Scenario:**

Your team builds an HR support agent that handles employee inquiries about benefits, payroll, and policies. During conversations, employees share personal information:

- "My social security number is XXX-XX-XXXX"
- "I was on medical leave from 01/15 to 02/28"
- "My address changed to 123 Main St"
- "I make $85,000 and want to increase my 401k contribution"

The agent needs this information to process requests but shouldn't retain it unnecessarily. The company must comply with privacy regulations.

**Question:** What data handling architecture should be implemented?

**Domain:** 5  
**Difficulty:** Medium  

A) Mask or redact PII from conversation logs after the session ends, retain logs for the minimum period required by policy, and implement data retention automation with purge schedules.

B) Block all PII from being entered into the agent.

C) Store everything permanently for audit purposes.

D) Ask employees not to share PII through the chat interface.

**Correct Answer:** A  
**Explanation:** Proper PII handling includes: (1) PII detection and masking in logs after the session (the agent needs it during the session), (2) automated data retention enforcement (logs purged after policy-defined period), (3) access controls for any stored conversation data. The agent needs PII during the conversation to process requests, but retention should be minimized.

**Why Others Wrong:**
- A: Correct. Session-use with post-hoc masking and retention automation.  
- B: The agent needs PII to process requests; blocking it breaks functionality.  
- C: Permanent retention violates privacy regulations and principles.  
- D: Unrealistic; employees need to share PII for HR processes.

**Exam Objective:** Design PII handling with session use, post-hoc masking, and automated retention.

---

## Scenario #16: Agent Performance Degradation Over Time

**Scenario:**

A team deploys a customer service agent and monitors its performance. For the first month, the agent performs well — 90% customer satisfaction, 80% first-contact resolution. Over the next two months, satisfaction drops to 78% and resolution drops to 65%.

The agent's configuration, tools, and prompts haven't changed. The model version is the same. The team is confused about why performance is degrading.

Analysis reveals that the types of questions customers are asking have changed. Initially, customers asked simple questions (order status, return policy). Over time, customers have learned to ask more complex questions as their trust in the system grows. Also, the product catalog has expanded, introducing new scenarios the agent wasn't initially handling well.

**Question:** What monitoring and response architecture should be in place?

**Domain:** 4, 3  
**Difficulty:** Medium  

A) Roll back to the initial configuration that worked well.

B) Implement **performance drift monitoring**: (1) track metrics not just overall but stratified by category and complexity, (2) analyze conversation topics weekly to detect shifts in user behavior, (3) have a process for adding new edge cases to the test suite as they emerge in production, (4) periodically refresh the evaluation dataset with recent production samples, (5) have a feedback loop where declining performance in specific categories triggers targeted prompt improvements.

C) The agent is learning bad habits; reset it monthly.

D) Customers are becoming more demanding; this is expected and acceptable.

**Correct Answer:** B  
**Explanation:** Agent performance degrades over time due to data drift (changing user behavior) and concept drift (changing product/service landscape). The solution is continuous monitoring with stratified metrics, regular test set refresh, and targeted improvement cycles. This is analogous to model monitoring in traditional ML.

**Why Others Wrong:**
- A: The initial configuration worked for simpler queries; the same config can't handle evolved usage.  
- B: Correct. Continuous monitoring with drift detection and test set refresh.  
- C: Resetting doesn't fix the drift problem; it just restarts the clock.  
- D: Accepting degradation is not an acceptable engineering approach.

**Exam Objective:** Implement performance drift monitoring and adaptive improvement for deployed agents.

---

## Scenario #17: Cost Optimization for Multi-Agent Systems

**Scenario:**

Your team runs a multi-agent research system that costs approximately $50 per query in API fees. The system uses:

- 1 coordinator agent (1 call)
- 5 research sub-agents (5 calls, potentially multiple rounds each)
- 1 synthesis agent (1-2 calls)
- 1 review agent (1 call)

A typical query costs $35-65. Your team needs to reduce costs by 40% without significantly degrading quality.

Analysis of usage shows:
- Most queries don't need all 5 sub-agents — often 2-3 would suffice
- The review agent often approves without changes (80% of the time)
- Sub-agents frequently make multiple tool calls to refine their search
- The synthesis agent often produces verbose output that's then condensed

**Question:** What cost optimization architecture would be most effective?

**Domain:** 1, 3  
**Difficulty:** Medium  

A) Remove the review agent entirely.

B) Implement **tiered processing with adaptive depth**: (1) classification step: categorize the query by complexity — simple queries use fewer sub-agents (2-3), complex queries use all 5, (2) conditional review: skip review for simple queries with high-confidence sources; only review complex or controversial queries, (3) sub-agent depth control: limit sub-agents to a maximum number of refinement rounds, (4) output-first synthesis: have sub-agents return structured summaries (not full reports) to reduce context and synthesis cost, (5) monitor quality per tier and adjust thresholds based on ongoing evaluation.

C) Switch to a cheaper model for all agents.

D) Combine the synthesis and review steps into one agent.

**Correct Answer:** B  
**Explanation:** Tiered processing adapts resource usage to query complexity. Simple queries get fewer sub-agents and skip review. Complex queries get full processing. This is more effective than blanket cuts because it preserves quality where it matters most while heavily optimizing common simple cases. Each optimization should be validated with quality monitoring.

**Why Others Wrong:**
- A: Removing review entirely may degrade quality for complex queries.  
- B: Correct. Adaptive tiered processing with quality monitoring.  
- C: A cheaper model may degrade quality for all queries, not just simple ones.  
- D: Combining steps loses the benefit of independent review.

**Exam Objective:** Design cost-optimized multi-agent architectures with adaptive resource allocation.

---

## Scenario #18: Real-Time Agent with Human Handoff

**Scenario:**

You are building a real-time voice agent for a telemedicine triage system. Patients call in, describe symptoms, and the agent determines urgency and routes accordingly.

The system must:
- Respond to patients in real-time (under 2 seconds per turn)
- Determine if the condition is an emergency (requires immediate human connection)
- Collect patient information (symptoms, duration, medications)
- Schedule appointments for non-urgent cases
- Handle patients who are emotional, confused, or have language barriers

Testing reveals that in 15% of calls, the agent should transfer to a human but doesn't, and in 10% of calls, the agent transfers unnecessarily.

**Question:** What architecture best handles the real-time requirements and human handoff decision?

**Domain:** 1, 5  
**Difficulty:** Hard  

A) Keep the agent simple with a low threshold for human transfer — when in doubt, transfer.

B) Implement a **dual-model architecture**: (1) a fast, lightweight triage model for real-time symptom collection and initial assessment (under 1 second), (2) a slower, more thorough analysis model running in parallel that processes the full conversation context for handoff decisions (1-3 seconds), (3) if the analysis model determines handoff is needed, the agent initiates the transfer, (4) call recording and transcription for the human agent's context, (5) escalation criteria calibrated on real call data with continuous improvement. The fast model handles the real-time interaction; the slower model makes the escalation decision with more context.

C) Use one very fast model and accept the lower accuracy.

D) Have a human listen to every call in real-time and handle transfers manually.

**Correct Answer:** B  
**Explanation:** Dual-model architecture separates the real-time interaction requirement from the complex decision-making requirement. The fast model handles patient interaction within latency requirements. The slower, more capable model analyzes the full context for escalation decisions. This is a common pattern in real-time agent systems where different components have different latency requirements.

**Why Others Wrong:**
- A: Low threshold means many unnecessary transfers, wasting human agent time.  
- B: Correct. Dual-model separates real-time response from complex decision-making.  
- C: Fast models alone may not have sufficient accuracy for medical triage decisions.  
- D: Manual monitoring defeats the purpose of automation.

**Exam Objective:** Design dual-model architectures separating real-time response from complex decision-making.

---

## Scenario #19: Agent Testing Strategy for Regulated Industries

**Scenario:**

Your team is building an agent that helps pharmacists check for drug interactions. The agent:

- Takes a list of medications
- Checks against a drug interaction database
- Flags potential interactions
- Provides severity ratings and recommendations

The system must be validated for regulatory compliance (equivalent to clinical decision support software). The team needs a testing strategy that:

1. Ensures the agent doesn't miss known drug interactions (safety-critical)
2. Ensures the agent doesn't flag false interactions (efficiency)
3. Handles edge cases (unusual drug combinations, dosing variations)
4. Can be audited by regulators

**Question:** What testing architecture meets these requirements?

**Domain:** 4, 5  
**Difficulty:** Hard  

A) Test with 1,000 randomly selected drug combinations from the database.

B) Implement a **comprehensive multi-layer testing strategy**: (1) **golden dataset**: 500+ curated test cases with known correct outputs, covering all interaction types and severity levels, with explicit edge cases, (2) **adversarial testing**: systematically test permutations of inputs designed to trigger failures (unusual doses, misspelled drug names, brand vs generic variations), (3) **regression test suite**: run after every prompt or configuration change, (4) **output schema validation**: enforce structured output with required fields (interaction found, severity, evidence, recommendation), (5) **independent verification**: a separate system (or human reviewer) verifies a random sample of outputs against the drug database, (6) **audit trail**: every query and response logged with timestamps for regulatory review.

C) Only test the most common 50 drug interactions since they cover 90% of cases.

D) Rely on the drug interaction database vendor's testing.

**Correct Answer:** B  
**Explanation:** Regulated medical applications require a multi-layer testing strategy. The golden dataset provides systematic coverage. Adversarial testing catches edge cases. Output schema validation ensures complete responses. Independent verification provides a second opinion. Audit trail supports regulatory review. No single layer is sufficient for safety-critical applications.

**Why Others Wrong:**
- A: Random sampling may miss critical edge cases.  
- B: Correct. Multi-layer testing with golden dataset, adversarial, schema validation, and audit trail.  
- C: Missing 10% of interactions is unacceptable for drug safety.  
- D: The database may be accurate but the agent's interpretation may not be.

**Exam Objective:** Design multi-layer testing strategies for regulated, safety-critical agent systems.

---

## Scenario #20: Agent-Based Data Migration

**Scenario:**

Your team needs to migrate data from an old CRM system to a new one. The old CRM has inconsistent data — phone numbers in various formats, addresses with typos, duplicate records, missing fields. The new CRM requires clean, structured data.

The team decides to use an agent to:
1. Read records from the old CRM
2. Clean and normalize the data
3. Map fields to the new schema
4. Write to the new CRM
5. Generate a migration report

The old CRM has 500,000 records. The migration must complete within a week.

**Question:** What architecture should be used for this data migration agent?

**Domain:** 1, 3  
**Difficulty:** Medium  

A) Process all 500,000 records in one batch with a single agent.

B) Implement a **chunked pipeline with quality gates**: (1) chunk records into batches of 500, (2) for each batch: extract → clean → validate → map → write, (3) after each chunk, validate a sample of migrated records, (4) track per-chunk quality metrics (error rate, fields corrected, records that failed), (5) if a chunk's error rate exceeds a threshold, flag it for human review, (6) provide parallel processing by running multiple chunks simultaneously, (7) generate a cumulative migration report with per-chunk breakdown.

C) Have a human manually clean all data before migration.

D) Migrate raw data and clean it in the new system.

**Correct Answer:** B  
**Explanation:** Large-scale data migration with an agent requires chunked processing with quality gates. Chunks of 500 records keep each batch manageable. Per-chunk validation catches issues early before they affect the entire dataset. Parallel processing meets the timeline. Quality metrics provide accountability and a clear audit trail.

**Why Others Wrong:**
- A: 500,000 records in one batch exceeds context windows and creates a single point of failure.  
- B: Correct. Chunked pipeline with quality gates and parallel processing.  
- C: 500,000 records of manual cleaning isn't feasible in a week.  
- D: Migrating dirty data just moves the problem to the new system.

**Exam Objective:** Design chunked data processing pipelines with quality validation for agent-based migration.

---

## Scenario #21: Agent with Conditional Autonomy

**Scenario:**

You are building an agent that helps with social media content moderation. The agent reviews reported content and takes action:

- Remove content that clearly violates policies
- Escalate ambiguous content to human moderators
- Leave clearly acceptable content alone

The challenge is that policy violations are nuanced. A post containing "kill" could be:
- A threat (remove + escalate)
- A video game reference ("I totally killed it in that match")
- A metaphor ("This heat is killing me")
- A quote from a book

The agent must distinguish these cases correctly. Over-removal harms user expression; under-removal harms community safety.

**Question:** What autonomy architecture balances these concerns?

**Domain:** 5, 1  
**Difficulty:** Hard  

A) The agent should remove all content containing policy keywords and let users appeal.

B) Implement a **confidence-based autonomy model with clear boundaries**: (1) define clear, specific policy violation criteria with examples for each category, (2) for each decision: assess confidence, (3) high-confidence clear violations: auto-remove with appeal process, (4) high-confidence clearly acceptable: auto-approve, (5) medium-confidence or ambiguous: escalate to human with agent's analysis and recommendation, (6) track human agreement rate with agent's escalation recommendations, (7) continuously calibrate: after humans make decisions on escalated cases, use those decisions to refine the agent's understanding.

C) Escalate all content to humans for review.

D) Use keyword-based filtering without an agent.

**Correct Answer:** B  
**Explanation:** Confidence-based tiered autonomy is appropriate when decisions have both cost of error and volume considerations. The agent handles clear cases autonomously. Ambiguous cases — where the cost of wrong action is high — get human review. The calibration loop ensures the agent learns from human decisions.

**Why Others Wrong:**
- A: Overly aggressive; removes acceptable content.  
- B: Correct. Confidence-based tiered autonomy with calibration loop.  
- C: Doesn't scale; humans can't review all content.  
- D: Keyword filtering is too simplistic for nuanced language.

**Exam Objective:** Design confidence-based autonomy tiers for nuanced decision-making agents.

---

## Scenario #22: Agent Output Formatting for Downstream Systems

**Scenario:**

Your team builds an agent that processes customer support tickets and outputs structured data for a ticketing system. The ticketing system requires:

```json
{
  "ticket_id": "TKT-12345",
  "priority": "HIGH" | "MEDIUM" | "LOW",
  "category": "billing" | "technical" | "account" | "general",
  "summary": "string (max 200 chars)",
  "description": "string (max 5000 chars)",
  "customer_email": "valid email format",
  "assigned_team": "billing-team" | "support-team" | "engineering-team"
}
```

The agent sometimes:
- Outputs priority as "Urgent" instead of "HIGH"
- Outputs category as "payment issue" instead of "billing"
- Writes summaries longer than 200 characters
- Uses incorrect email format
- Hallucinates a ticket_id instead of using the system-generated one

**Question:** How should the agent be designed to reliably produce valid outputs for downstream systems?

**Domain:** 2, 4  
**Difficulty:** Medium  

A) Tell the agent "follow the JSON schema exactly" in the prompt.

B) Use **output schema enforcement with validation pipeline**: (1) define the output schema with JSON Schema including `enum` constraints for priority/category/assigned_team, `maxLength` for string fields, and `format: email` for email, (2) use structured output mode (if available) or constrained decoding, (3) add a validation step after the agent generates output — if validation fails, return the error to the agent with specific guidance for correction, (4) include a `FORBIDDEN` instruction: "Do NOT generate a ticket_id; it will be provided by the system", (5) keep a retry counter (max 3) to prevent infinite loops.

C) Have a developer manually fix formatting issues before sending to the ticketing system.

D) Make the ticketing system more flexible in accepting varied input formats.

**Correct Answer:** B  
**Explanation:** Schema enforcement with validation and retry is the most reliable approach. JSON Schema constraints (`enum`, `maxLength`, `format`) constrain the agent's output. A validation pipeline catches and fixes failures. This is more reliable than prompt-only approaches because correction is automated and targeted.

**Why Others Wrong:**
- A: Prompt-only is insufficient; the agent will still make errors.  
- B: Correct. Schema enforcement + validation + targeted retry.  
- C: Doesn't scale and defeats automation.  
- D: Making downstream systems flexible creates maintenance burden and data inconsistency.

**Exam Objective:** Design output validation pipelines with schema enforcement for downstream system integration.

---

## Scenario #23: Agent with Multi-Step Approval Workflow

**Scenario:**

You are designing an agent for a procurement system. When an employee requests to purchase something:

1. The agent checks if the item is in the approved vendor list
2. Checks if the cost is within the employee's authorization limit
3. If over the limit, routes to the employee's manager for approval
4. If over the manager's limit, routes to department head
5. If over the department head's limit, routes to finance
6. Creates a purchase order once all approvals are obtained
7. Sends the PO to the vendor

Different approval chains apply based on department, cost center, and item type. The agent must track the status of each request and handle rejections at any stage.

**Question:** What architecture supports this multi-step, multi-actor approval workflow?

**Domain:** 1, 5  
**Difficulty:** Medium  

A) Build one large prompt that handles all approval steps sequentially.

B) Implement a **state machine architecture**: (1) define states (initialized → vendor_check → cost_check → approval_pending → approved → rejected → po_created), (2) each state has clear entry criteria and possible transitions, (3) persist state in a database so the agent can resume after approvals that may take days, (4) the agent processes each state transition, (5) when waiting for human approval, the agent sends notifications and waits for webhook callbacks, (6) timeouts: if an approval step takes too long, notify the requester and escalate, (7) rejection handling: if rejected at any step, notify the requester with the reason and move to `rejected` state.

C) Have the agent ask for all approvals upfront.

D) Process purchases one at a time, waiting for each approval before proceeding.

**Correct Answer:** B  
**Explanation:** A state machine is the correct pattern for multi-step approval workflows with external dependencies (human approvers, varying timelines, rejection at any step). State persistence in a database allows the workflow to span days. Clear state transitions make the process predictable and debuggable.

**Why Others Wrong:**
- A: A single prompt can't handle days-long workflows with external dependencies.  
- B: Correct. State machine with persistence for long-running workflows.  
- C: Asking for all approvals upfront is impractical for hierarchical approval chains.  
- D: Serial processing is correct but needs a state machine to be robust.

**Exam Objective:** Design state machine architectures for multi-step workflow agents with external dependencies.

---

## Scenario #24: Agent Observability and Debugging

**Scenario:**

Your team deploys a customer-facing agent. When something goes wrong — the agent gives wrong information, makes an unauthorized promise, or fails to resolve an issue — the team has no way to understand what happened. They can see the user's messages and the agent's responses, but they can't see:

- What tools were called and in what order
- What the tool results were
- What the agent was "thinking" between steps
- Which prompt or system instruction was active
- What model version was used

When a critical error occurs (the agent promised a refund the company can't honor), the support team needs to understand exactly what led to the mistake.

**Question:** What observability architecture should be implemented?

**Domain:** 3, 4  
**Difficulty:** Medium  

A) This level of detail isn't needed; just review the user-agent conversation.

B) Implement **comprehensive agent observability**: (1) structured logging at every step with timestamp, (2) log every tool call with input parameters and full response, (3) log intermediate reasoning (the agent's chain of thought), (4) log the active system prompt and any configuration parameters, (5) assign a unique trace ID to each session connecting all logs, (6) store logs in a queryable system (e.g., structured logging DB), (7) implement a "session replay" view where an investigator can step through the agent's actions, (8) for critical errors, implement alerting based on specific patterns (promises, commitments, financial amounts).

C) Screen-record every agent session.

D) Have a human monitor every agent interaction in real-time.

**Correct Answer:** B  
**Explanation:** Comprehensive observability with structured logging at every step is essential for debugging and auditing agent systems. Trace IDs connect all events in a session. Tool call logging shows exactly what data the agent received. Reasoning traces show what the agent was considering. Session replay allows post-mortem analysis.

**Why Others Wrong:**
- A: Without tool-level detail, investigators can't determine root cause.  
- B: Correct. Comprehensive structured logging with trace IDs and session replay.  
- C: Screen recordings don't capture tool calls, reasoning, or internal state.  
- D: Doesn't scale and doesn't help with post-hoc analysis.

**Exam Objective:** Design comprehensive observability with structured logging for agent debugging.

---

## Scenario #25: Agent Deployment with Canary Testing

**Scenario:**

Your team has a production customer service agent handling 50,000 conversations per day. You want to deploy a new version with improved prompts and additional tools. However, you're concerned about regression — the new version might handle some cases worse than the current version.

You need a deployment strategy that:
- Minimizes risk
- Provides statistically significant comparison
- Allows quick rollback if issues are detected
- Gradually increases traffic to the new version

**Question:** What deployment architecture should be used?

**Domain:** 3, 4  
**Difficulty:** Medium  

A) Deploy the new version to all users at once and monitor metrics.

B) Implement a **canary deployment with shadow evaluation**: (1) deploy the new version alongside the current version, (2) start with 5% of traffic routed to the new version, (3) compare key metrics (satisfaction, resolution rate, escalation rate, error rate) between old and new on the same traffic, (4) use statistical testing to determine if differences are significant, (5) if the new version performs better or not worse on all key metrics, gradually increase traffic (10% → 25% → 50% → 100%), (6) if any metric degrades significantly, roll back the new version, investigate, and iterate, (7) keep the ability to instant rollback by maintaining the previous version's deployment.

C) A/B test in a staging environment only.

D) Deploy to a single customer and monitor.

**Correct Answer:** B  
**Explanation:** Canary deployment with gradual traffic increase and statistical comparison is the standard approach for production agent updates. Starting at 5% limits blast radius. Real traffic comparison is more reliable than staging because production data distributions are hard to replicate. Statistical testing prevents false conclusions from noise.

**Why Others Wrong:**
- A: Full deployment risks widespread issues with no gradual exposure.  
- B: Correct. Canary deployment with statistical comparison and rollback capability.  
- C: Staging doesn't capture production data distributions and user behavior.  
- D: Single-customer sample is not statistically meaningful.

**Exam Objective:** Design canary deployment strategies for production agent updates.

---

## Scenario #26: Agent with Dynamic Tool Discovery

**Scenario:**

Your team builds an agent platform where third-party developers can create and publish tools for the agent. New tools are added daily. The agent needs to discover and use relevant tools for each user request.

Challenges:
- 500+ available tools and growing
- New tools are added without agent configuration changes
- Tool quality varies — some are well-designed, others have poor descriptions
- Some tools overlap in functionality
- The agent needs to find the right tool without trying all 500

**Question:** What tool discovery architecture scales to hundreds of tools?

**Domain:** 2, 1  
**Difficulty:** Hard  

A) Give the agent all 500 tools and let it choose.

B) Implement a **tool retrieval architecture**: (1) each tool has a structured manifest with name, description, input schema, output schema, category tags, and usage examples, (2) when a user request comes in, embed the request and use vector similarity search to find the top 10-15 most relevant tools, (3) present only these retrieved tools to the agent, (4) if the agent can't find a suitable tool among retrieved ones, it can trigger a search for alternative tools, (5) implement tool quality scoring — tools with better descriptions, higher usage, and positive feedback rank higher in retrieval, (6) periodically re-embed tools if descriptions are updated.

C) Categorize tools into folders and give the agent one folder at a time.

D) Have a human select relevant tools for each request.

**Correct Answer:** B  
**Explanation:** Vector-based tool retrieval is the standard pattern for scaling to hundreds or thousands of tools. Instead of overwhelming the agent with all options, retrieval narrows to the most relevant subset. This preserves tool selection quality while supporting a large, growing catalog. This is the same pattern used by many agent frameworks.

**Why Others Wrong:**
- A: 500 tools exceeds the recommended maximum by two orders of magnitude.  
- B: Correct. Vector retrieval narrows to relevant subset.  
- C: Hard to categorize tools into mutually exclusive folders.  
- D: Doesn't scale and defeats the purpose of automation.

**Exam Objective:** Design vector-based tool retrieval for large-scale, dynamic tool catalogs.

---

## Scenario #27: Agent Handling of Incomplete or Incorrect User Input

**Scenario:**

A tax preparation agent helps users file their taxes. The agent asks questions about income, deductions, and credits.

A user provides the following input throughout the conversation:
- "I made about $75,000 last year" (approximate)
- "I think I paid around $12,000 in mortgage interest" (approximate)
- "My wife's name is Sarah... or is it Sara? I always forget" (uncertain)
- "We donated to a few charities, maybe $2,000 total" (approximate)
- "I live at 123 Oak Street... wait, we moved. I don't remember the new address" (incorrect/uncertain)

The agent needs to handle approximate and uncertain data appropriately. Tax filing requires accurate information.

**Question:** How should the agent handle approximate, uncertain, or incorrect user input?

**Domain:** 5, 1  
**Difficulty:** Medium  

A) Accept the approximate values and file the taxes — close enough is good enough.

B) Implement **uncertainty tracking and clarification patterns**: (1) flag all approximate or uncertain values ("about", "maybe", "I think", "around"), (2) for each uncertain value, ask the user to confirm or provide exact information, (3) if the user can't provide exact values, suggest ways to find the information (check W-2, look at mortgage statement, verify spelling on ID), (4) for critical fields (name, SSN, address), require exact values and don't proceed until they're confirmed, (5) if the user repeatedly can't provide exact values, recommend consulting a tax professional, (6) never file with approximate values — mark the return as "draft" until all values are confirmed.

C) Use the most common value from last year's tax data.

D) File with approximate values and let the IRS correct them.

**Correct Answer:** B  
**Explanation:** Tax filing is a domain where accuracy matters. The agent should flag uncertainty, ask for clarification, suggest verification methods, and never proceed with uncorrected approximations. This is a safety-critical pattern — knowing when to insist on accurate data and when to help the user find it.

**Why Others Wrong:**
- A: Filing taxes with approximate values can lead to audits, penalties, or incorrect refunds.  
- B: Correct. Systematic uncertainty handling with verification guidance.  
- C: Last year's data may not be accurate for the current year.  
- D: Relying on IRS correction is irresponsible and burdens the system.

**Exam Objective:** Design uncertainty tracking and clarification patterns for accuracy-critical agent domains.

---

## Scenario #28: Agent with External Knowledge Integration

**Scenario:**

You are building a legal research agent that helps lawyers find relevant precedents. The agent has access to:
- A vector database of 1 million case law summaries
- A citation graph showing how cases reference each other
- A tool to fetch full case texts
- A tool to check if a case is still good law (not overturned)

A lawyer asks: "Find me cases about vicarious liability in employer-employee relationships where the employee was using their personal vehicle."

The agent does a vector search and finds 30 potentially relevant cases. It needs to:
1. Filter for cases that are still good law
2. Rank by relevance to the specific scenario
3. Identify the most authoritative cases (most cited by other cases)
4. Present a concise summary of the top cases

**Question:** What retrieval and ranking architecture should be used?

**Domain:** 1, 2  
**Difficulty:** Medium  

A) Present all 30 cases to the lawyer and let them decide.

B) Implement a **multi-stage retrieval and ranking pipeline**: (1) **Stage 1 — Broad Retrieval**: vector similarity search returns 30 candidate cases, (2) **Stage 2 — Validity Filter**: check each case's "good law" status, remove overturned or superseded cases, (3) **Stage 3 — Citation Ranking**: rank remaining cases by citation count/authority, (4) **Stage 4 — Relevance Re-ranking**: use the agent's understanding of the specific scenario (personal vehicle, vicarious liability) to re-rank, prioritizing cases with similar fact patterns, (5) **Stage 5 — Synthesis**: the agent produces a structured summary of top 5-7 cases, each with: case name, citation, key holding, relevance to the query, and status (good law), (6) **Stage 6 — Verification**: the agent cites specific passages from each case to support its relevance assessment.

C) Only return the single best matching case.

D) Use keyword search instead of vector search.

**Correct Answer:** B  
**Explanation:** Legal research requires a multi-stage retrieval pipeline. Broad retrieval captures candidates. Validity filtering removes bad law. Authority ranking prioritizes important cases. Relevance re-ranking tailors results to the specific query. Synthesis produces an actionable summary. Each stage adds value that a single-stage approach cannot provide.

**Why Others Wrong:**
- A: 30 cases overwhelm the lawyer without prioritization.  
- B: Correct. Multi-stage pipeline with filtering, ranking, and synthesis.  
- C: Too narrow; may miss highly relevant alternatives.  
- D: Keyword search is less effective than vector search for semantic matching.

**Exam Objective:** Design multi-stage retrieval and ranking pipelines for domain-specific agent research.

---

## Scenario #29: Agent with Session Timeout and Recovery

**Scenario:**

A customer is chatting with a support agent about a complex billing issue. The conversation has been going on for 45 minutes. The customer has provided detailed information:

- Account number and verification
- Screenshots of incorrect charges
- Preferred resolution (refund + correction)
- Authorization for the agent to investigate

Suddenly, the agent's session times out due to an infrastructure issue. The agent loses all context. The customer has to start over from scratch, re-verifying their identity and re-explaining the entire issue.

The customer is extremely frustrated. The company loses a customer.

**Question:** What session resilience architecture prevents this?

**Domain:** 3, 5  
**Difficulty:** Medium  

A) Increase the session timeout to 2 hours.

B) Implement **session persistence and recovery**: (1) persist the complete conversation state to a database after every turn, including all tool calls, results, and intermediate reasoning, (2) when a new session starts for an existing customer, detect the interrupted session, (3) provide the agent with a structured recovery summary: customer identity (already verified), issue description, actions taken so far, pending actions, any commitments made, (4) when the agent resumes, it acknowledges the interruption, summarizes what it knows, and asks if the customer wants to continue, (5) implement graceful degradation: if the full state can't be restored, the agent uses the recovery summary and transparently communicates any gaps, (6) test session recovery as part of regular chaos engineering.

C) Tell customers to save their session ID to resume later.

D) Have the agent repeat everything it knows when the customer reconnects.

**Correct Answer:** B  
**Explanation:** Session persistence with structured recovery summaries is the correct architecture. Persisting state after every turn ensures minimal data loss. The recovery summary provides the new session with essential context without overwhelming it with full history. Acknowledging the interruption and transparently communicating any gaps maintains trust.

**Why Others Wrong:**
- A: Increasing timeout doesn't prevent infrastructure failures; sessions will still be lost.  
- B: Correct. Session persistence with structured recovery.  
- C: Customers shouldn't need to manage session state.  
- D: Repeating everything wastes context and may have gaps.

**Exam Objective:** Design session persistence and recovery for long-running agent conversations.

---

## Scenario #30: Multi-Modal Agent for Document Processing

**Scenario:**

Your team builds an agent that processes insurance claims. Claims come with:
- A filled claim form (PDF, structured fields)
- Photos of damage (JPEG images)
- A police report (PDF, scanned document, not machine-readable)
- Previous claim history (JSON from database)
- Policy documents (PDF, machine-readable)

The agent must:
1. Extract all information from these documents
2. Cross-reference information across documents (e.g., does the damage date match the police report date?)
3. Check the policy for coverage
4. Determine if the claim should be approved, rejected, or escalated
5. Generate a detailed claims assessment report

Challenges: the police report is a scanned PDF (images, not text). The damage photos need visual analysis. The claim form has handwriting in some fields.

**Question:** What multi-modal processing architecture handles this?

**Domain:** 1, 2  
**Difficulty:** Hard  

A) Convert everything to text and use a text-only agent.

B) Implement a **multi-modal pipeline**: (1) **Document classification and routing**: identify each document type and choose the appropriate processing path, (2) **Text extraction**: OCR for scanned PDFs (police report), text extraction for machine-readable PDFs (policy documents), (3) **Image analysis**: use a vision-capable model to analyze damage photos for severity and type, (4) **Structured data extraction**: parse the claim form (with handwriting recognition for handwritten fields), (5) **Cross-referencing agent**: compare extracted data across sources — do dates match? does the damage description match the photos? does the police report align with the claim?, (6) **Policy check agent**: compare claim details against policy coverage, (7) **Decision agent**: synthesize all findings, apply claims guidelines, and determine outcome, (8) **Report generation agent**: produce a structured assessment with all findings and evidence.

C) Have humans transcribe all non-digital documents first.

D) Use separate agents for each document type without cross-referencing.

**Correct Answer:** B  
**Explanation:** Multi-modal claims processing requires a pipeline with specialized processing paths for each document type. OCR handles scanned documents. Vision models analyze photos. Structured extraction handles forms. The critical step is cross-referencing — comparing data across sources to detect inconsistencies. Each specialized component feeds into a synthesis step that produces the final assessment.

**Why Others Wrong:**
- A: Text-only loses visual information from photos and handwriting.  
- B: Correct. Specialized processing paths with cross-referencing.  
- C: Human transcription doesn't scale and introduces delay.  
- D: Without cross-referencing, inconsistencies across documents are missed.

**Exam Objective:** Design multi-modal processing pipelines with specialized paths and cross-referencing for document-heavy agent workflows.
