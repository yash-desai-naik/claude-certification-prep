# Mock Exam: Claude Certified Architect Foundations

**60 Questions | Time: 90 minutes | Domain-aligned distribution**

---

## Instructions

- Each question is scenario-based. Choose the **best** answer.
- Some questions may have multiple seemingly correct options — pick the one that is most appropriate according to official Anthropic recommendations.
- 52+ correct = ready for exam.

---

### Question 1
**Scenario:** A fintech startup needs to integrate Claude into their customer support system. They want to handle account inquiries, transaction disputes, and general FAQs. The CTO is concerned about cost predictability and wants to avoid surprise bills during traffic spikes.
**Question:** Which approach best balances cost predictability with responsiveness?
**Domain:** 1
**Difficulty:** Medium

A) Use the Messages API with auto-scaling enabled and no caps
B) Configure max tokens and use Batch API for all customer inquiries
C) Use the Messages API with token budgeting, request batching during off-peak, and real-time processing during business hours
D) Pre-generate all responses using Claude and cache them in a database

**Correct Answer:** C
**Explanation:** Token budgeting gives cost predictability by capping per-request spend. Batching off-peak requests saves costs on non-urgent queries while keeping real-time processing for active hours. Option A lacks controls, option B is too slow for real-time support, and option D is inflexible for varied customer situations.

---

### Question 2
**Scenario:** A developer sends a request to Claude and receives an API response with `stop_reason: "end_turn"`. The assistant message contains valid JSON the application needs to parse.
**Question:** What does `stop_reason: "end_turn"` specifically indicate about this response?
**Domain:** 1
**Difficulty:** Easy

A) The model hit the maximum token limit mid-response
B) The model decided it had sufficiently answered and handed control back to the user
C) The content was blocked by the safety filter
D) The model encountered an internal error

**Correct Answer:** B
**Explanation:** `end_turn` means the model voluntarily stopped because it assessed the conversation turn was complete. This is the normal stop reason for well-formed responses. It differs from `max_tokens` (hit limit) or `stop_sequence` (found a custom stop sequence).

---

### Question 3
**Scenario:** A healthcare SaaS company is building an AI assistant that drafts patient follow-up notes. They need a reliable mechanism to ensure Claude always outputs structured data in a specific JSON format, even if the user prompt is vague.
**Question:** Which configuration approach guarantees structured output?
**Domain:** 1
**Difficulty:** Medium

A) Use system prompt instructions requesting JSON format
B) Set `tool_choice` to `any` and provide a well-defined tool specification for the output schema
C) Set `temperature` to 0 and retry on malformed responses
D) Use a post-processing script to parse unstructured text into JSON

**Correct Answer:** B
**Explanation:** Setting `tool_choice` to `any` forces Claude to use a tool, and the tool's input schema enforces the exact JSON structure. This is more reliable than prompt instructions alone (A), which are heuristics. Temperature at 0 (C) helps but doesn't guarantee structure. Post-processing (D) is fragile.

---

### Question 4
**Scenario:** An edtech platform uses Claude to generate personalized lesson plans. They notice that after 8-10 messages in a conversation, the quality of lesson plans degrades and the model starts repeating information.
**Question:** What is the most likely cause and the best mitigation?
**Domain:** 1
**Difficulty:** Medium

A) Model drift — re-initialize the conversation every 5 messages
B) Context window nearing capacity causing the model to lose earlier instructions — implement conversation summarization or sliding window
C) Rate limiting — reduce request frequency
D) Temperature decay — increase temperature to 0.7

**Correct Answer:** B
**Explanation:** Long conversations fill the context window, pushing earlier system instructions and key context out. Summarization techniques compress older turns while preserving essential information. Sliding window approaches keep only the most recent N messages plus a summary of older ones.

---

### Question 5
**Scenario:** An enterprise team wants to create a reusable Claude-powered document analysis tool. Multiple developers will contribute prompts and tool definitions. They want consistency across all deployments.
**Question:** What is the recommended way to define and share these configurations?
**Domain:** 1
**Difficulty:** Easy

A) Share a Word document with prompt guidelines
B) Define the prompt, tools, and settings in a `CLAUDE.md` file at the project root
C) Hardcode prompts in each application's source code
D) Use environment variables for prompts

**Correct Answer:** B
**Explanation:** `CLAUDE.md` files provide project-level configuration that standardizes prompt behavior, tool access, and settings across all users of a project. This creates a single source of truth. It supports inheritance, so teams can layer project-specific overrides on organization-wide defaults.

---

### Question 6
**Scenario:** A developer is building an application that uses Claude to triage support tickets. They want the model to classify tickets into categories (billing, technical, account) and extract priority level, then optionally escalate if urgency is high.
**Question:** Which tool configuration approach supports this branching logic?
**Domain:** 2
**Difficulty:** Medium

A) Define one tool for classification and another for escalation; let Claude decide which to use
B) Define three separate tools for each category
C) Use a single tool with all parameters required
D) Use parallel tool calls for classification and escalation in one step

**Correct Answer:** A
**Explanation:** Defining separate tools for classification and escalation lets Claude use the classification tool first, then decide based on the priority output whether to call the escalation tool. Tool selection is a model-time decision, enabling conditional branching without application-side logic.

---

### Question 7
**Scenario:** A team wants Claude to be able to look up customer data from their CRM, check inventory from their warehouse system, and send Slack notifications. All three actions have independent data dependencies.
**Question:** When should the application use parallel tool calls?
**Domain:** 2
**Difficulty:** Medium

A) Always — parallel is always faster
B) Only when tools have no data dependencies on each other's outputs
C) Only when tools are marked as read-only
D) Never — tools must always be called sequentially

**Correct Answer:** B
**Explanation:** Parallel tool calls execute concurrently which improves latency, but they're only safe when there are no interdependencies. The CRM lookup and inventory check are independent and can run in parallel, while a notification that depends on both results must wait.

---

### Question 8
**Scenario:** A developer notices that Claude sometimes calls the wrong tool or invokes a tool with incorrect parameters. They want to improve tool selection accuracy.
**Question:** Which approach most effectively improves tool selection accuracy?
**Domain:** 2
**Difficulty:** Hard

A) Reduce the number of tools to one and handle routing in application code
B) Write detailed, unambiguous tool descriptions including when to use each tool and what each parameter means
C) Use very short tool names like "f1", "f2", "f3" to keep prompts small
D) Set tool_choice to "required" so Claude always calls a tool

**Correct Answer:** B
**Explanation:** Tool descriptions are the primary signal Claude uses for tool selection. Well-written descriptions that include usage context, parameter semantics, and examples dramatically improve accuracy. Short names (C) reduce signal. Option A defeats the purpose of letting Claude decide.

---

### Question 9
**Scenario:** A legal document review application sends large contracts to Claude for clause extraction. The contract text exceeds 150,000 tokens. The application currently truncates the input, which causes missing clauses.
**Question:** What is the best way to handle documents that exceed the context window?
**Domain:** 2
**Difficulty:** Hard

A) Split the document into chunks and process each chunk independently, then merge results in application code
B) Reduce the document resolution by removing punctuation and whitespace
C) Only process the first and last sections of the document
D) Set max_tokens to the maximum allowed value

**Correct Answer:** A
**Explanation:** Chunking large documents into context-window-sized pieces and processing them independently is the standard approach. The application then merges or de-duplicates results. This preserves all content. Options B and C lose information. Option D addresses output length, not input limits.

---

### Question 10
**Scenario:** A team deploying Claude in production observes response times of 5-8 seconds for complex analytical queries. Users expect responses under 3 seconds.
**Question:** Which strategy will most directly reduce end-to-end latency?
**Domain:** 2
**Difficulty:** Medium

A) Reduce max_tokens to the minimum needed for each response
B) Move from async to sync processing
C) Cache common queries using a semantic similarity cache
D) Increase temperature to reduce thinking time

**Correct Answer:** C
**Explanation:** Semantic caching stores embeddings of past queries and serves cached responses for similar questions, often reducing latency to under 500ms for cache hits. Reducing max_tokens (A) helps modestly. Option B doesn't change processing time. Option D doesn't meaningfully affect latency.

---

### Question 11
**Scenario:** A developer sends a streaming request to Claude and notices that the stream ends before all content is delivered. The final event shows `stop_reason: "max_tokens"`.
**Question:** What should the developer do to get the complete response?
**Domain:** 2
**Difficulty:** Easy

A) Retry the same request — it was a network error
B) Increase the `max_tokens` parameter in the request
C) Set `stream` to false
D) Use a different model

**Correct Answer:** B
**Explanation:** `stop_reason: "max_tokens"` means the response was cut off because it hit the token limit. The developer should increase `max_tokens` to allow the model to complete. Option A is incorrect because this is not an error, just a limit being reached.

---

### Question 12
**Scenario:** A developer sends a request to the Messages API. The API responds with an HTTP 429 status code.
**Question:** What is the appropriate handling strategy for this response?
**Domain:** 2
**Difficulty:** Easy

A) Immediately resend the same request
B) Implement exponential backoff with jitter and retry
C) Log the error and abandon the request
D) Switch from the Messages API to the Text Completions API

**Correct Answer:** B
**Explanation:** HTTP 429 means rate limit exceeded. The correct response is to wait and retry with exponential backoff — double the wait time between each attempt, with random jitter to avoid thundering herd problems. Option A will likely get another 429. Option C gives up too easily.

---

### Question 13
**Scenario:** A data processing pipeline needs to analyze 100,000 customer reviews for sentiment. The process is not time-sensitive and can take up to 24 hours.
**Question:** Which API should be used for this workload?
**Domain:** 2
**Difficulty:** Easy

A) Messages API with streaming
B) Batch API
C) Messages API with concurrent requests
D) Tool Use API

**Correct Answer:** B
**Explanation:** The Batch API is ideal for large-scale asynchronous workloads where results are not needed immediately. It offers 50% cost reduction compared to standard API calls and handles large volumes efficiently. The 24-hour SLA aligns with the time requirement.

---

### Question 14
**Scenario:** A team submits a batch of 10,000 text classification jobs through the Batch API. After 30 minutes, they check the status and find that the batch is still "in_progress."
**Question:** What does "in_progress" mean for batch processing?
**Domain:** 2
**Difficulty:** Easy

A) The batch has failed and needs to be resubmitted
B) Some results may already be available to retrieve
C) All results will arrive at once when the batch completes
D) The batch was rejected by the validation system

**Correct Answer:** B
**Explanation:** The Batch API processes requests asynchronously and incrementally. Individual results become available as they complete, even while the overall batch status is "in_progress." The team can poll for individual results by request ID.

---

### Question 15
**Scenario:** A compliance team needs to ensure that Claude never generates content that could be interpreted as medical advice when used in a general wellness application.
**Question:** Which content moderation approach is most effective?
**Domain:** 3
**Difficulty:** Medium

A) Rely on the model's built-in refusal training
B) Implement a two-layer guard: system prompt restrictions plus an output classification model that screens for medical advice
C) Use tool_choice: "none" to prevent any action
D) Limit max_tokens to 50 to restrict response length

**Correct Answer:** B
**Explanation:** Defense in depth is most reliable. System prompt restrictions set the behavioral boundary, and an output classifier catches any violations the model might miss. This two-layer approach is standard for sensitive domains. Single-layer approaches (A, C, D) are insufficient.

---

### Question 16
**Scenario:** A developer is creating a customer-facing chatbot and needs Claude to respond only in Spanish. Despite clear instructions in the system prompt, Claude occasionally responds in English.
**Question:** What is the most reliable way to enforce language consistency?
**Domain:** 3
**Difficulty:** Medium

A) Add "RESPOND ONLY IN SPANISH" in all caps to every user message
B) Use a tool definition that requires language output and set tool_choice to "any"
C) Set temperature to 0
D) Fine-tune the model on Spanish-only data

**Correct Answer:** B
**Explanation:** Forcing tool use with a tool that outputs structured data in the required language is more reliable than prompt instructions alone. The tool schema becomes a structural constraint rather than a suggestion. Option A is fragile, C helps but doesn't guarantee, D is overkill.

---

### Question 17
**Scenario:** A developer wants Claude to extract addresses from emails and return them as structured JSON with fields: street, city, state, zip, country. The extraction must be precise.
**Question:** Which schema design approach minimizes extraction errors?
**Domain:** 3
**Difficulty:** Medium

A) Define the tool with a single string parameter "address" and parse JSON from it
B) Define each field as a separate required parameter with descriptions and format constraints in the JSON Schema
C) Use a free-form text prompt and ask for JSON format
D) Set temperature to 0.9 for creative extraction

**Correct Answer:** B
**Explanation:** Defining each field as a separate parameter with clear descriptions and format constraints gives Claude precise guidance per field. Well-defined JSON Schema with type constraints and descriptions produces the most reliable structured output. Option A offloads parsing to application code.

---

### Question 18
**Scenario:** An application needs Claude to handle multi-step data enrichment: first validate an email format, then look up the user in the database, then format a personalized response.
**Question:** What is the recommended architectural pattern for this chain?
**Domain:** 3
**Difficulty:** Hard

A) Single prompt with all instructions — let Claude figure out the sequence
B) Chain of Thought in a single turn with tool calls executed sequentially
C) Multi-turn conversation where each turn handles one step, passing results as context
D) Use the Batch API to process all steps in parallel

**Correct Answer:** B
**Explanation:** Claude can reason through a multi-step task in a single turn using Chain of Thought, making sequential tool calls as needed. It maintains context internally without requiring application-level orchestration. Multi-turn (C) adds unnecessary latency and complexity.

---

### Question 19
**Scenario:** An e-commerce site uses Claude to generate product descriptions. The marketing team notices that descriptions for electronics tend to be much longer than those for clothing, even though the prompts are similar in structure.
**Question:** What is the likely cause of this inconsistency?
**Domain:** 3
**Difficulty:** Medium

A) The model has inherent bias toward electronics categories
B) Token allocation varies based on the complexity of the input product specifications
C) Temperature is set too high
D) Max_tokens is set per category incorrectly

**Correct Answer:** B
**Explanation:** Claude allocates response length naturally based on the information density of the input. Product descriptions with more specifications (electronics) naturally generate longer outputs. If consistency is desired, explicit length guidelines in the prompt or setting `max_tokens` per category type would help.

---

### Question 20
**Scenario:** A developer is building a code generation tool. They need Claude to output valid Python code that the application can execute directly. Occasionally the model includes markdown code blocks, which breaks the execution pipeline.
**Question:** Which prompt engineering technique most reliably prevents markdown formatting in code outputs?
**Domain:** 3
**Difficulty:** Easy

A) Add "NO MARKDOWN" to the user prompt
B) Use a system prompt directing the model to output only raw code without formatting, and use examples in a few-shot format showing raw code output
C) Post-process the output to strip markdown
D) Set response_format to "text"

**Correct Answer:** B
**Explanation:** Few-shot examples showing the exact desired output format are the most reliable technique. The model generalizes from the pattern in the examples. Combined with a clear system prompt instruction, this consistently produces raw code output without markdown.

---

### Question 21
**Scenario:** A developer has defined 15 tools for a customer support agent. Claude frequently confuses tools with similar purposes, such as "refund_order" and "process_refund."
**Question:** What is the most likely root cause and best fix?
**Domain:** 3
**Difficulty:** Medium

A) Too many tools — split into separate agents with fewer tools each
B) Tool names are too similar — use descriptive, semantically distinct names and detailed descriptions
C) Model is not capable enough — use a more advanced model
D) Set tool_choice to "required" to force tool selection

**Correct Answer:** B
**Explanation:** Confusable tool names are a common source of tool selection errors. Names like "refund_order" and "process_refund" are semantically too similar. Distinct names such as "issue_refund_to_customer" and "process_return_request" reduce confusion. Detailed descriptions for each tool provide additional signal.

---

### Question 22
**Scenario:** A team is designing a CLAUDE.md file for their project. They want different settings for their CI/CD pipeline analysis tool versus their code review assistant.
**Question:** How should they structure the CLAUDE.md configuration?
**Domain:** 4
**Difficulty:** Easy

A) One global CLAUDE.md at the root with all settings combined
B) A root CLAUDE.md with shared settings and subdirectory CLAUDE.md files for project-specific overrides
C) Separate configuration files outside the CLAUDE.md system
D) Environment variables that switch between configurations

**Correct Answer:** B
**Explanation:** CLAUDE.md supports a three-level hierarchy: organization-level, project-level (root), and subdirectory-level. Subdirectory configurations inherit from and can override parent settings. This allows shared global defaults with team-specific or tool-specific overrides.

---

### Question 23
**Scenario:** An enterprise is deploying Claude across multiple teams: engineering, marketing, and customer support. Each team has different needs but should follow company-wide safety guidelines.
**Question:** Which CLAUDE.md hierarchy supports this requirement?
**Domain:** 4
**Difficulty:** Medium

A) One CLAUDE.md per repository with all settings duplicated
B) Organization-level CLAUDE.md with global safety rules, overridden by team-level CLAUDE.md files for specific behaviors
C) No hierarchy — use environment variables for all settings
D) Store all configuration in a database and load at runtime

**Correct Answer:** B
**Explanation:** The three-level CLAUDE.md hierarchy maps well to this: organization level enforces global safety rules, project level adds team-specific configuration, and subdirectory level handles tool-specific needs. Inheritance means teams automatically get global policies without duplication.

---

### Question 24
**Scenario:** A developer is building a system where Claude needs to analyze user requests and decide whether to escalate to a human agent or respond directly. This decision depends on sentiment, topic complexity, and user history.
**Question:** Which architecture best supports this triage workflow?
**Domain:** 4
**Difficulty:** Medium

A) A single Claude call that handles everything
B) A hub-and-spoke architecture where a Claude instance acts as router, dispatching to specialized Claude instances for different tasks
C) Hardcoded rules in the application layer for escalation
D) Random assignment between human and AI response

**Correct Answer:** B
**Explanation:** Hub-and-spoke allows the routing Claude (hub) to evaluate the input and dispatch to specialized spokes (billing specialist, technical support, etc.). Each spoke has focused tools and prompts. This is more scalable and maintainable than a monolithic approach.

---

### Question 25
**Scenario:** A team wants to implement a RAG (Retrieval-Augmented Generation) system where Claude answers questions based on company documentation. They need to handle multi-part questions that require synthesizing information from multiple documents.
**Question:** What is the best approach for handling multi-document synthesis questions?
**Domain:** 4
**Difficulty:** Hard

A) Retrieve one document and ask Claude to answer from that single source
B) Retrieve the top-K most relevant document chunks, include them all in the context, and let Claude synthesize the answer
C) Ask the user to rephrase as separate questions
D) Use the Batch API to query each document separately

**Correct Answer:** B
**Explanation:** Retrieving multiple relevant chunks and providing them all in context allows Claude to synthesize across documents. Claude's reasoning capability enables it to identify connections, resolve contradictions, and combine information from multiple sources. K should be tuned based on context window size.

---

### Question 26
**Scenario:** A team is building a multi-agent system where a coordinator agent delegates tasks to specialist agents. The specialist agents sometimes return incomplete results, and the coordinator doesn't detect this.
**Question:** How should the team structure feedback between agents to ensure quality?
**Domain:** 4
**Difficulty:** Hard

A) Have the coordinator agent always trust specialist outputs
B) Implement a validation step where the coordinator reviews specialist outputs against expected schemas before proceeding
C) Remove the coordinator — let specialists communicate directly
D) Use the same agent for all tasks

**Correct Answer:** B
**Explanation:** A validation step creates a quality gate. The coordinator can validate tool outputs against expected schemas, check for completeness, and request re-processing if needed. This pattern is essential in production multi-agent systems to prevent error propagation.

---

### Question 27
**Scenario:** A developer wants to share context between two different Claude-powered features in the same application: a chatbot and a document analyzer. Both need access to the same user profile data.
**Question:** Which context management pattern supports this sharing?
**Domain:** 4
**Difficulty:** Medium

A) Duplicate the user profile data in both conversations
B) Store shared context in a database and inject it into each conversation's system prompt
C) Use a global variable in the application code
D) Have the chatbot call the document analyzer via an API

**Correct Answer:** B
**Explanation:** Externalizing shared context to a database and injecting it into system prompts ensures both features have consistent, up-to-date information. This avoids duplication (A) and the injection point makes it auditable. The system prompt is the right place for this context.

---

### Question 28
**Scenario:** A team is deploying Claude behind an API gateway. They need to implement authentication so that only authorized services can make requests, and they want per-service usage tracking.
**Question:** Which security architecture should they use?
**Domain:** 4
**Difficulty:** Medium

A) Embed API keys directly in the frontend application code
B) Use an API gateway with per-service API keys, request signing, and usage logging
C) Rely on IP allowlisting only
D) Use the same API key for all services for simplicity

**Correct Answer:** B
**Explanation:** An API gateway with per-service keys provides authentication, authorization, and observability. Request signing prevents tampering. Usage logging enables billing and rate limiting per service. Options A, C, and D all lack necessary security controls.

---

### Question 29
**Scenario:** A developer creates an application that makes multiple parallel tool calls. One tool call returns an error because the downstream API is unavailable. The other tool calls succeeded.
**Question:** How should the application handle partial tool call failures?
**Domain:** 4
**Difficulty:** Hard

A) Abort the entire request and return an error to the user
B) Process the successful tool results and retry the failed tool call, informing Claude of the partial state
C) Ignore the error — partial data is acceptable
D) Retry all tool calls from scratch

**Correct Answer:** B
**Explanation:** Processing partial results and retrying the failed call is the most resilient approach. Claude can work with partial data and respond based on what's available. Informing Claude about the failure allows it to adapt its response or request the user for alternative information.

---

### Question 30
**Scenario:** A developer is designing tool schemas for an inventory management system. One tool takes a product ID, a quantity, and an optional discount code. The discount code should only be accepted if it's a valid, non-expired code.
**Question:** Where should validation logic for the discount code live?
**Domain:** 4
**Difficulty:** Hard

A) In the tool's JSON Schema as a regex pattern
B) In the application code that executes the tool — the schema defines types, the application validates business rules
C) In the system prompt — explain the validation rules to Claude
D) In the tool description — let Claude validate before calling

**Correct Answer:** B
**Explanation:** JSON Schema handles structural validation (types, formats, required fields). Business logic validation (whether a discount code is valid and non-expired) belongs in application code where it can query databases and check expiration dates. This separation of concerns is critical.

---

### Question 31
**Scenario:** A developer discovers that their application is sending very long prompts (50,000+ tokens) with embedded documents for every request. Most of the document content is repeated across requests.
**Question:** Which optimization would most reduce costs and latency?
**Domain:** 4
**Difficulty:** Medium

A) Compress the documents using zip before sending
B) Implement prompt caching so repeated content is cached between requests
C) Reduce temperature to 0 for all requests
D) Use max_tokens set to the minimum

**Correct Answer:** B
**Explanation:** Prompt caching stores the system prompt and repeated context (like documents) on the server side between requests. This dramatically reduces input token costs (cached tokens are cheaper) and latency for repeated contexts. It's the most impactful optimization for this use case.

---

### Question 32
**Scenario:** A developer is designing a system where different Claude instances handle different parts of a workflow: one validates input, one processes the request, and one formats the output.
**Question:** What is the primary consideration when designing this pipeline?
**Domain:** 4
**Difficulty:** Hard

A) Use the largest model for all stages for consistency
B) Design each stage with clear, well-defined input/output schemas so data passes reliably between stages
C) Use streaming between all stages
D) Process all stages in parallel for speed

**Correct Answer:** B
**Explanation:** Well-defined schemas at each stage boundary are critical for pipeline reliability. They enable validation, error handling, and composability. Without clear contracts between stages, errors propagate unpredictably and debugging becomes difficult.

---

### Question 33
**Scenario:** A team is implementing a user feedback loop where Claude's responses are rated by users. They want to use this feedback to improve response quality over time for that specific application.
**Question:** How should the team incorporate user feedback into their Claude-powered application?
**Domain:** 4
**Difficulty:** Medium

A) Automatically retrain the model on high-rated responses
B) Log feedback with request/response pairs, analyze patterns to refine prompts, and use negative feedback to identify prompt engineering improvements
C) Discard negative feedback — only focus on positive ratings
D) Send feedback directly to Anthropic's model training pipeline

**Correct Answer:** B
**Explanation:** User feedback should be logged, analyzed, and used to iteratively improve prompts, tool definitions, and system design. Patterns in negative feedback reveal weaknesses in prompt engineering or tool configuration. Option A isn't how API-based models work. Option D is not supported.

---

### Question 34
**Scenario:** A company is building a Claude-based coding assistant for internal developers. They've noticed that suggestions for the company's proprietary framework are often incorrect and don't follow internal conventions.
**Question:** What is the most effective way to improve suggestion quality for the proprietary framework?
**Domain:** 4
**Difficulty:** Medium

A) Include the full framework documentation in every prompt
B) Provide few-shot examples of correct code patterns for the framework in the system prompt
C) Ask developers to manually correct each incorrect suggestion
D) Use a different model that supports fine-tuning

**Correct Answer:** B
**Explanation:** Few-shot examples showing correct code patterns for the proprietary framework give Claude concrete references to follow. This is more effective than documentation alone (A) because examples show usage patterns directly. The examples should cover common patterns the framework uses.

---

### Question 35
**Scenario:** An internal audit reveals that a Claude-powered application sometimes generates content that could be considered biased against certain demographic groups. The team needs to address this immediately.
**Question:** Which immediate mitigation strategy should the team implement?
**Domain:** 5
**Difficulty:** Medium

A) Shut down the application until a full retraining is complete
B) Add explicit system prompt instructions requiring fair, unbiased treatment of all demographics, plus an output moderation filter
C) Reduce temperature to 0 to eliminate variability
D) Restrict the application to factual data only

**Correct Answer:** B
**Explanation:** Adding explicit fairness guidelines in the system prompt combined with output filtering provides immediate defense against biased outputs. The system prompt sets behavioral expectations, and the moderation layer catches violations. This can be deployed quickly while longer-term improvements are developed.

---

### Question 36
**Scenario:** A developer is building a Claude-powered resume screening tool. They are concerned about the model developing biases based on names, gender indicators, or educational institutions mentioned in resumes.
**Question:** What should the developer prioritize in their system design?
**Domain:** 5
**Difficulty:** Medium

A) Instruct Claude to ignore personal details and evaluate only skills and experience
B) Pre-process resumes to redact names, gender indicators, and institution names before sending to Claude
C) Use a separate model for screening
D) Apply different evaluation criteria for different demographic groups

**Correct Answer:** B
**Explanation:** Pre-processing to remove protected attributes is a standard fairness practice. It prevents the model from having access to information that could lead to biased decisions. This is more reliable than instructing the model to ignore attributes (A) because the model never sees them.

---

### Question 37
**Scenario:** A team stores user conversations with Claude in their database for quality assurance. They are unsure about the privacy implications and compliance requirements.
**Question:** What should the team consider regarding data handling?
**Domain:** 5
**Difficulty:** Easy

A) All data sent to Anthropic is automatically deleted after 30 days
B) Check the data processing agreement with Anthropic, implement user consent, and ensure compliance with applicable privacy regulations (GDPR, CCPA, etc.)
C) Anonymization is not required for internal storage
D) Anthropic owns all data sent through the API

**Correct Answer:** B
**Explanation:** Companies must understand their data processing agreement with Anthropic (API data is not used for training by default), obtain user consent for storing conversations, and comply with applicable privacy regulations. Data handling responsibilities are shared between the customer and Anthropic per the agreement.

---

### Question 38
**Scenario:** A developer accidentally includes a hardcoded database password in a prompt sent to the Claude API. The credential is visible in the API request logs.
**Question:** What should the developer do immediately?
**Domain:** 5
**Difficulty:** Easy

A) Do nothing — Anthropic doesn't store prompts
B) Rotate the compromised credential immediately and implement secret detection to prevent future leaks
C) Delete the API logs from Anthropic's servers
D) Remove the credential from the prompt and continue

**Correct Answer:** B
**Explanation:** The credential should be considered compromised because it was transmitted to an external API. Immediate rotation is necessary. The team should also implement pre-request scanning for secrets using tools like secret scanners or environment variable checkers.

---

### Question 39
**Scenario:** A company building a Claude-powered financial advisory tool must comply with SEC regulations requiring that all financial advice be explainable and auditable.
**Question:** What architectural pattern supports this requirement?
**Domain:** 5
**Difficulty:** Hard

A) Use a black-box approach with no logging
B) Log all inputs, outputs, and tool calls; implement Chain of Thought prompting so the reasoning is visible; and include citations for financial claims
C) Use a highly restricted model with no reasoning capability
D) Only provide factual information without any advice

**Correct Answer:** B
**Explanation:** Comprehensive logging creates audit trails. Chain of Thought makes reasoning visible and auditable. Citations let reviewers verify claims against source data. This combination satisfies regulatory requirements for explainability while still delivering value.

---

### Question 40
**Scenario:** A globally distributed team is using Claude to review code changes. They notice that code quality feedback for developers in certain regions is more critical than feedback for developers in other regions.
**Question:** What is the most likely cause of this inconsistency?
**Domain:** 5
**Difficulty:** Medium

A) Model training data has regional imbalances
B) Prompt phrasing differences across regions — the prompts should be standardized and tested for consistency
C) Developers in some regions write worse code
D) The model performs differently based on IP address

**Correct Answer:** B
**Explanation:** Inconsistent prompting across teams is a common source of behavioral variation. If different regions use different prompt templates or variations in wording, the model's responses will differ. Standardizing prompts, using shared CLAUDE.md configurations, and testing for consistent output quality solves this.

---

### Question 41
**Scenario:** A developer is designing a tool that lets Claude read and write files in the application's workspace. They need to ensure Claude cannot access system files outside the designated workspace.
**Question:** Which security approach should be used?
**Domain:** 5
**Difficulty:** Medium

A) Trust Claude to follow instructions about which files to access
B) Implement path validation and sandboxing in the tool execution layer — the tool code should validate paths against an allowed directory before performing any operation
C) Obfuscate system file paths in the prompt
D) Use a regex in the tool description to restrict paths

**Correct Answer:** B
**Explanation:** Security must be enforced at the application layer, not in the prompt. Path validation that checks all file operations against an allowed directory, combined with sandboxing (e.g., running in a container), provides real security. Prompt-level instructions (A, D) are heuristics, not enforcement.

---

### Question 42
**Scenario:** A team discovers that their Claude-powered application is generating outputs that could be considered harmful when users phrase queries in specific adversarial ways.
**Question:** What is the most robust defense against adversarial inputs?
**Domain:** 5
**Difficulty:** Hard

A) Block all queries containing certain keywords
B) Implement a multi-layer defense: input validation, system prompt guardrails, output moderation, and continuous monitoring for novel attack patterns
C) Use only pre-approved query templates
D) Reduce model capability by lowering max_tokens

**Correct Answer:** B
**Explanation:** Defense in depth is the standard approach for adversarial robustness. Each layer catches what the previous layer misses. Input validation blocks obvious attacks, prompt guardrails set boundaries, output moderation catches violations, and monitoring detects new attack patterns for continuous improvement.

---

### Question 43
**Scenario:** A developer sends an API request with `max_tokens: 500` and the response comes back with `stop_reason: "stop_sequence"`. The output is 200 tokens and appears complete.
**Question:** What does this indicate?
**Domain:** 1
**Difficulty:** Easy

A) The response was truncated — increase max_tokens
B) The model encountered a predefined stop sequence in its output and stopped there — this is normal behavior
C) There was an error in the request
D) The model refused to answer

**Correct Answer:** B
**Explanation:** `stop_sequence` means the model generated text that matched one of the stop sequences defined in the request. This is a normal completion, not an error. The response is intentionally limited at that point. The 200-token output being complete confirms this.

---

### Question 44
**Scenario:** A team is evaluating whether to use a single large model or multiple specialized smaller models for their customer support platform. Cost and latency are primary concerns.
**Question:** Which approach is most cost-effective for this scenario?
**Domain:** 1
**Difficulty:** Medium

A) Use the largest model for every request to ensure quality
B) Use a routing layer that directs simple queries to a smaller, faster model and complex queries to a larger model
C) Use only small models for all requests
D) Use different models based on the time of day

**Correct Answer:** B
**Explanation:** Model routing optimizes cost and latency by matching query complexity to model capability. Simple FAQ lookups go to cheaper, faster models while complex multi-step troubleshooting goes to larger models. This can reduce costs by 60-80% compared to using large models for everything.

---

### Question 45
**Scenario:** A developer is building a feature that requires Claude to process user-uploaded images and extract text from them, then perform analysis on the extracted text.
**Question:** What is the correct approach for multi-modal processing?
**Domain:** 1
**Difficulty:** Medium

A) Convert images to text client-side and send only the text
B) Pass image data as base64-encoded content blocks in the Messages API alongside analysis instructions
C) Use a separate OCR service and send results to Claude
D) Send images as URLs and text separately

**Correct Answer:** B
**Explanation:** Claude supports multi-modal inputs through content blocks in the Messages API. Images can be passed as base64-encoded data with image/media type, and text analysis instructions can be included in the same request. This avoids the overhead and potential errors of separate processing pipelines.

---

### Question 46
**Scenario:** An application sends requests to Claude with `temperature: 0.7`. The output quality varies significantly between runs with the same input.
**Question:** What is the expected behavior at this temperature setting?
**Domain:** 1
**Difficulty:** Easy

A) The model will always produce identical outputs
B) Higher temperature increases randomness — outputs will naturally vary. For deterministic results, use temperature: 0
C) Temperature only affects response length, not content
D) Temperature must be between 0 and 0.5 for valid requests

**Correct Answer:** B
**Explanation:** Temperature controls output randomness. Higher values (0.7) produce more varied and creative responses, while temperature 0 produces near-deterministic output. The variability at 0.7 is expected and desirable for creative tasks but problematic when consistency is needed.

---

### Question 47
**Scenario:** A developer is using the Messages API and wants to include a history of previous exchanges to provide context. They're unsure about the correct message structure.
**Question:** Which message role structure is correct for multi-turn conversations?
**Domain:** 1
**Difficulty:** Easy

A) [{"role": "user", "content": "..."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]
B) [{"role": "system", "content": "..."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}, {"role": "user", "content": "..."}]
C) [{"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]
D) [{"role": "assistant", "content": "..."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]

**Correct Answer:** C
**Explanation:** Multi-turn conversations must alternate user/assistant role pairs. Each user turn is followed by an assistant turn, then the next user turn. Messages must always start with a user turn (after optional system prompt). Alternating is required by the Messages API specification.

---

### Question 48
**Scenario:** A developer has defined a tool that performs a database update. When Claude calls this tool and the application executes it, the database update fails due to a constraint violation.
**Question:** How should the application communicate this failure back to Claude?
**Domain:** 2
**Difficulty:** Medium

A) Ignore the error and proceed as if it succeeded
B) Return a `tool_result` content block with an error message describing the failure, using `is_error: true` if supported
C) Raise an exception in the application
D) Retry the database operation 5 times before giving up

**Correct Answer:** B
**Explanation:** Returning a descriptive error in `tool_result` content tells Claude exactly what went wrong. If `is_error: true` is supported, it signals the result is an error. Claude can then decide how to respond — whether to fix parameters and retry, or inform the user. This maintains the conversational flow.

---

### Question 49
**Scenario:** A developer wants to send 50,000 records through Claude for data enrichment. Each record takes about 1 second to process sequentially, which would take nearly 14 hours. They need results within 4 hours.
**Question:** Which approach meets the time requirement?
**Domain:** 2
**Difficulty:** Hard

A) Increase max_tokens to speed up individual requests
B) Use the Batch API which processes requests asynchronously with internal parallelism
C) Use concurrent requests with appropriate rate limit handling
D) Reduce the data enrichment per record

**Correct Answer:** B
**Explanation:** The Batch API processes all requests concurrently on Anthropic's infrastructure, so 50,000 records complete in roughly the same wall-clock time as a single request (typically within hours). Concurrent requests (C) would hit rate limits and require complex management. The Batch API handles this natively.

---

### Question 50
**Scenario:** A developer submits a Batch API request and receives a 400 error with the message "Result URL not available yet."
**Question:** What does this error mean?
**Domain:** 2
**Difficulty:** Easy

A) The batch was rejected permanently
B) The results are still being processed — the team should poll the batch status endpoint until it completes
C) The URL format was incorrect
D) Authentication failed for the batch results

**Correct Answer:** B
**Explanation:** Batch results are available only after the batch reaches a terminal state ("completed" or "failed"). The developer should poll the batch status endpoint. Once status changes to "completed," result URLs become available. This is expected behavior, not an error.

---

### Question 51
**Scenario:** A development team wants to deploy a Claude-powered code review assistant. They want the assistant to check for security vulnerabilities, code style violations, and potential performance issues.
**Question:** How should they configure the tool definitions for this use case?
**Domain:** 3
**Difficulty:** Medium

A) One tool called "run_all_checks" that performs all three checks at once
B) Three separate tools: "check_security", "check_style", "check_performance" — each with clear descriptions of what to check
C) One tool per line of code
D) A single tool with a "check_type" parameter

**Correct Answer:** B
**Explanation:** Separate tools for each concern area allow Claude to call the appropriate tool based on what it identifies in the code. Clear descriptions guide accurate tool selection. This modular approach is more maintainable and allows Claude to prioritize based on what it finds.

---

### Question 52
**Scenario:** A developer is creating a JSON Schema for a tool parameter that accepts an array of email addresses. Some emails fail validation because users include names like "John Doe <john@example.com>".
**Question:** What schema design pattern handles this input variation?
**Domain:** 3
**Difficulty:** Hard

A) Accept strings and parse emails on the server side
B) Use a `oneOf` schema that accepts either a simple email string or a formatted email string, with clear descriptions for each variant
C) Require users to fix their input format
D) Use an enum of all possible email formats

**Correct Answer:** B
**Explanation:** The `oneOf` JSON Schema keyword allows accepting multiple valid formats. Each format variant gets its own schema with a description. Claude can then match the actual input to the appropriate variant. This is the recommended pattern for polymorphic inputs.

---

### Question 53
**Scenario:** A developer notices that Claude sometimes makes multiple tool calls in a single response, even when the developer expected only one tool call per turn.
**Question:** Is this behavior expected, and how can it be controlled?
**Domain:** 3
**Difficulty:** Medium

A) Unexpected — this is a bug; report to Anthropic
B) Expected — Claude can make multiple tool calls in one turn when it determines efficiency. Control via prompt instructions (e.g., "Only use one tool at a time")
C) Only happens when tool_choice is "any"
D) Can be prevented by setting max_tokens to 1

**Correct Answer:** B
**Explanation:** Claude can make multiple parallel or sequential tool calls in a single turn when it identifies independent or sequentially dependent operations. This is a feature that improves efficiency. If single-tool behavior is desired, a prompt instruction to use one tool at a time is the appropriate control.

---

### Question 54
**Scenario:** A team is building an application where Claude helps users compose professional emails. The application prompts Claude with writing style preferences in the system prompt. Some users report that the style isn't applied consistently.
**Question:** What is the best approach to enforce consistent style?
**Domain:** 3
**Difficulty:** Medium

A) Repeat style instructions at the end of every user message
B) Include few-shot examples of desired email styles in the system prompt showing before/after transformations
C) Post-process all outputs through a style-checking script
D) Use a higher temperature for more creative style matching

**Correct Answer:** B
**Explanation:** Few-shot examples showing desired input-to-output transformations are the most reliable way to establish and maintain a consistent style. They demonstrate the expected transformation pattern concretely, which is more effective than declarative instructions (A) about style.

---

### Question 55
**Scenario:** A team is designing a CLAUDE.md configuration for a monorepo containing a frontend (React) and backend (Node.js) application. They want Claude to understand which code belongs to which part of the stack.
**Question:** What is the recommended CLAUDE.md structure for this monorepo?
**Domain:** 4
**Difficulty:** Medium

A) Single CLAUDE.md at root with general instructions
B) Root CLAUDE.md for monorepo context, with separate CLAUDE.md files in `/frontend` and `/backend` directories containing stack-specific instructions
C) No CLAUDE.md — use only prompt instructions
D) One large CLAUDE.md in each subdirectory with full instructions

**Correct Answer:** B
**Explanation:** The hierarchy allows the root CLAUDE.md to describe the monorepo structure and shared conventions. Subdirectory CLAUDE.md files in `/frontend` and `/backend` inherit these and add framework-specific instructions. This avoids duplication while keeping each area's configuration precise.

---

### Question 56
**Scenario:** A developer is designing a Claude-powered system that writes and executes SQL queries against a production database. They are concerned about destructive queries (DROP, DELETE, UPDATE).
**Question:** Which architecture best protects the database?
**Domain:** 5
**Difficulty:** Hard

A) Give Claude full SQL access and trust it to be careful
B) Route all queries through a read-only database proxy with a separate tool for write operations that requires human approval
C) Restrict Claude to only SELECT queries
D) Monitor database logs after queries execute

**Correct Answer:** B
**Explanation:** A read-only proxy for queries prevents accidental data modifications. Write operations through a separate tool with human approval adds a safety gate. This defense-in-depth approach protects data integrity. Option A is too permissive, C is too restrictive, and D is reactive rather than preventive.

---

### Question 57
**Scenario:** A company is deploying Claude for internal HR use. They want to ensure the system never makes decisions about hiring, firing, or compensation — it should only assist with information gathering and documentation.
**Question:** How should the system architecture enforce this boundary?
**Domain:** 5
**Difficulty:** Medium

A) Add a system prompt instruction saying "Never make hiring decisions"
B) Design the system with no tools that can execute HR decisions — only tools for retrieving information and drafting documents
C) Use the smallest available model to limit capability
D) Review all outputs manually

**Correct Answer:** B
**Explanation:** The most reliable boundary is architectural: if no tools exist that can execute decisions, Claude cannot make them. Tool design is an architectural constraint that's more robust than prompt instructions. Prompt instructions (A) can be circumvented by adversarial inputs.

---

### Question 58
**Scenario:** A developer is implementing content filtering for a Claude application targeting children. They need to ensure all outputs are age-appropriate.
**Question:** Which combination of techniques provides the strongest safety guarantee?
**Domain:** 5
**Difficulty:** Medium

A) System prompt with age-appropriateness guidelines
B) System prompt + output content filter + periodic human review of sampled outputs
C) Only use predefined response templates
D) Restrict all inputs to yes/no questions

**Correct Answer:** B
**Explanation:** A layered approach combines prompt guidance (proactive), automated content filtering (real-time), and human review (reactive). Each layer catches what others miss. For applications targeting vulnerable populations (children), this multi-layer approach is considered best practice.

---

### Question 59
**Scenario:** An enterprise customer wants to use Claude to analyze proprietary code and confidential business documents. Their legal team is concerned about data being used for model training.
**Question:** What should the enterprise verify before proceeding?
**Domain:** 5
**Difficulty:** Easy

A) Nothing — all API data is automatically excluded from training by default
B) Verify the data processing agreement terms — API data is NOT used for training by default, but the enterprise should confirm and can request additional safeguards
C) All data sent to Anthropic becomes Anthropic's property
D) Use only open-source models to be safe

**Correct Answer:** B
**Explanation:** By default, Anthropic's API does not use customer data for model training. Enterprises should verify their specific agreement terms and can request additional safeguards like data deletion commitments. Understanding the data processing agreement is a standard compliance step.

---

### Question 60
**Scenario:** A team is building a Claude application that will be used globally across multiple time zones and cultures. They want to ensure the application is culturally sensitive and appropriate.
**Question:** What approach best ensures cross-cultural appropriateness?
**Domain:** 5
**Difficulty:** Medium

A) Use a single prompt translated into each language
B) Design culturally-aware prompts with local context, test with diverse user groups, implement feedback loops, and include cultural sensitivity in system instructions
C) Rely on the model's training to handle cultural differences
D) Avoid references to any cultural specifics

**Correct Answer:** B
**Explanation:** Cross-cultural deployment requires proactive effort: localized prompts that account for cultural norms, testing with representative user groups, feedback collection, and explicit instructions about cultural sensitivity. Option A ignores cultural differences beyond language. Option C is passive. Option D produces generic, less useful responses.

---

## Answer Key

| Q# | Answer | Domain | Difficulty |
|----|--------|--------|------------|
| 1  | C      | 1      | Medium     |
| 2  | B      | 1      | Easy       |
| 3  | B      | 1      | Medium     |
| 4  | B      | 1      | Medium     |
| 5  | B      | 1      | Easy       |
| 6  | A      | 2      | Medium     |
| 7  | B      | 2      | Medium     |
| 8  | B      | 2      | Hard       |
| 9  | A      | 2      | Hard       |
| 10 | C      | 2      | Medium     |
| 11 | B      | 2      | Easy       |
| 12 | B      | 2      | Easy       |
| 13 | B      | 2      | Easy       |
| 14 | B      | 2      | Easy       |
| 15 | B      | 3      | Medium     |
| 16 | B      | 3      | Medium     |
| 17 | B      | 3      | Medium     |
| 18 | B      | 3      | Hard       |
| 19 | B      | 3      | Medium     |
| 20 | B      | 3      | Easy       |
| 21 | B      | 3      | Medium     |
| 22 | B      | 4      | Easy       |
| 23 | B      | 4      | Medium     |
| 24 | B      | 4      | Medium     |
| 25 | B      | 4      | Hard       |
| 26 | B      | 4      | Hard       |
| 27 | B      | 4      | Medium     |
| 28 | B      | 4      | Medium     |
| 29 | B      | 4      | Hard       |
| 30 | B      | 4      | Hard       |
| 31 | B      | 4      | Medium     |
| 32 | B      | 4      | Hard       |
| 33 | B      | 4      | Medium     |
| 34 | B      | 4      | Medium     |
| 35 | B      | 5      | Medium     |
| 36 | B      | 5      | Medium     |
| 37 | B      | 5      | Easy       |
| 38 | B      | 5      | Easy       |
| 39 | B      | 5      | Hard       |
| 40 | B      | 5      | Medium     |
| 41 | B      | 5      | Medium     |
| 42 | B      | 5      | Hard       |
| 43 | B      | 1      | Easy       |
| 44 | B      | 1      | Medium     |
| 45 | B      | 1      | Medium     |
| 46 | B      | 1      | Easy       |
| 47 | C      | 1      | Easy       |
| 48 | B      | 2      | Medium     |
| 49 | B      | 2      | Hard       |
| 50 | B      | 2      | Easy       |
| 51 | B      | 3      | Medium     |
| 52 | B      | 3      | Hard       |
| 53 | B      | 3      | Medium     |
| 54 | B      | 3      | Medium     |
| 55 | B      | 4      | Medium     |
| 56 | B      | 5      | Hard       |
| 57 | B      | 5      | Medium     |
| 58 | B      | 5      | Medium     |
| 59 | B      | 5      | Easy       |
| 60 | B      | 5      | Medium     |

### Domain Score Breakdown

| Domain | Questions | Your Score | % | Status |
|--------|-----------|------------|---|--------|
| 1: Anthropic & Claude Fundamentals | 16 (Q1-5, Q43-47) | /16 | % | |
| 2: Core API & SDK Concepts | 11 (Q6-14, Q48-50) | /11 | % | |
| 3: Advanced Capabilities | 12 (Q15-21, Q51-54) | /12 | % | |
| 4: Application Architecture & Patterns | 12 (Q22-34, Q55) | /12 | % | |
| 5: Safety, Security & Responsible AI | 9 (Q35-42, Q56-60) | /9 | % | |
| **Total** | **60** | **/60** | **%** | |

---

## Score Interpretation

| Score Range | Percentage | Status |
|-------------|------------|--------|
| 52-60 | 87-100% | **Ready for exam** — Strong understanding across all domains |
| 43-51 | 72-86% | **Good** — Review weak domains and re-attack difficult questions |
| 36-42 | 60-71% | **Needs more study** — Focus on domains with lowest scores |
| Below 36 | Below 60% | **Needs significant preparation** — Re-study all domains thoroughly |
