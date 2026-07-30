# Community Guide Gap Supplement

This file covers SPECIFIC topics from the community guide (paullarionov/claude-certified-architect/guide_en.md)
that are NOT fully covered in the main study materials. Study these to ensure complete exam readiness.

---

## Gap 1: Mid-Conversation System Messages

**Source:** Community Guide Chapter 1.2

In the Messages API, `system` can appear as a role inside the `messages` array (not just at the top-level `system` field).

**Purpose:** Add or change instructions mid-conversation without invalidating the cached prefix from the top-level system field.

**Placement Rules:**
- Must immediately follow a `user` turn (including one with `tool_result` blocks) or an `assistant` turn ending in server tool use
- Must precede an `assistant` turn or end the array
- Cannot sit between a `tool_use` block and its `tool_result` (returns 400 error)
- Later `system` messages take precedence over earlier ones and over the top-level `system` field

**Example:**
```json
{"role": "system", "content": "Now use a more formal tone."}
```

**Exam Relevance:** Medium. Tests understanding of mid-conversation instruction changes and API constraints.

---

## Gap 2: Format Normalization Rules in Prompts

**Source:** Community Guide Chapter 6.1

When using strict JSON schemas for structured output, ADD normalization rules in the prompt to prevent semantic errors:

```
Normalization Rules:
- Dates: always ISO 8601 (YYYY-MM-DD); "yesterday" -> compute absolute date
- Currency: numeric amount + currency code; "five bucks" -> {"amount": 5, "currency": "USD"}
- Percentages: decimal fraction; "half" -> 0.5
- Units: standard SI units; "a couple inches" -> {"value": 5, "unit": "cm"}
```

**Why this matters:** JSON schemas guarantee syntax but NOT semantic correctness. Normalization rules tell the model HOW to convert informal values into the expected format.

**Exam Relevance:** High. Tests understanding that valid JSON ≠ correct values.

---

## Gap 3: Pydantic for Validation

**Source:** Community Guide Chapter 6.5

Pydantic is a Python library for schema-based data validation. Key exam points:

| Capability | What it does | Why it matters |
|---|---|---|
| **Structural validation** | Checks types, required fields, enum constraints | Catches missing/wrong-type fields |
| **Semantic validation** | Custom validators enforce business logic | Sum(items) == total; start_date < end_date |
| **Validate–retry loops** | On failure, build error message + re-prompt Claude | Fixes extractable errors |
| **JSON Schema generation** | Pydantic models can generate JSON Schema for `tool_use` | Single source of truth for schemas |

**Pattern:**
```
1. Claude extracts data into JSON
2. Pydantic validates structure AND semantics
3. If error → retry with: original doc + incorrect extraction + specific error message
4. If info absent from source → retry won't help
```

**Exam Trap:** Students think Pydantic only does type checking. It also handles semantic validation (sums, dates, business rules).

---

## Gap 4: Rendering by Content Type (Provenance)

**Source:** Community Guide Chapter 12.4

Don't force everything into one format. Match the format to the data type:

| Data Type | Best Format | Why |
|-----------|-------------|-----|
| Financial data | Tables | Columns for periods, rows for metrics |
| News/analysis | Prose paragraphs | Narrative flow, context |
| Technical findings | Structured lists | Scanable, comparable |
| Time series | Chronological ordering | Trend visibility |
| Conflicting claims | Side-by-side comparison | Attribution preserved |

**Exam Relevance:** Medium. Tests understanding that provenance includes presentation format.

---

## Gap 5: Incremental Investigation Strategy (Built-in Tools)

**Source:** Community Guide Chapter 13.2

Do NOT read all files at once. Build understanding INCREMENTALLY:

```
Step 1: Grep → find entry points (function definitions, exports)
Step 2: Read → read the found files
Step 3: Grep → find usages (imports, call sites)
Step 4: Read → read consumer files
Step 5: Repeat until complete picture
```

**Anti-pattern:** Reading every file in a directory before understanding the structure.
**Correct pattern:** Trace from entry point → dependencies → consumers.

**Why this matters for exam:** Questions about context management in large codebases test this incremental approach vs loading everything at once.

---

## Gap 6: Fallback Strategy — Read + Write Instead of Edit

**Source:** Community Guide Chapter 13.3

The `Edit` tool works by matching a UNIQUE text snippet in a file and replacing it. When Edit FAILS (non-unique match):

**Correct fallback:**
```
1. Read → load the full file content
2. Modify the content programmatically
3. Write → write the updated version
```

**Why it matters:** Edit is more precise but can fail on duplicate text. Read+Write is the reliable fallback.

---

## Gap 7: Disable Parallel Tool Use

**Source:** Official Docs (https://platform.claude.com/docs/en/build-with-claude/tool-use)

The `tool_choice` parameter can include `disable_parallel_tool_use: true`:

```python
tool_choice={"type": "auto", "disable_parallel_tool_use": True}
```

**What it does:** Limits Claude to ONE tool call per turn (instead of calling multiple tools in parallel).

**When to use:**
- Tool calls have dependencies (need result of Tool A before calling Tool B)
- Sequential processing required
- Rate-limited APIs

**Default behavior:** Without this flag, Claude can call multiple tools in parallel in a single turn.

**Exam Relevance:** Medium. Tests understanding of parallel vs sequential tool execution control.

---

## Gap 8: Strict Tool Use

**Source:** Official Docs (https://platform.claude.com/docs/en/build-with-claude/tool-use#strict-tool-use)

Add `strict: true` to tool definitions to guarantee Claude's tool calls match your schema exactly:

```python
tools=[{
    "name": "get_weather",
    "description": "...",
    "input_schema": {...},
    "strict": true
}]
```

**What it does:** Ensures tool call arguments follow the JSON schema precisely — no missing args, no wrong types.

**When to use:** Any custom tool where schema compliance is critical.
**Trade-off:** May reject valid calls that use slightly different but semantically equivalent inputs.

---

## Gap 9: Extended Thinking

**Source:** Official Docs (https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

Extended thinking lets Claude "think before it speaks" — generating internal reasoning before producing the visible response.

**Key exam points:**
- Uses additional tokens for internal reasoning
- Controlled via `thinking` parameter in API requests
- NOT the same as `max_tokens` — thinking tokens are separate
- Available on Opus and Sonnet models
- Can be combined with tool use

**Exam Relevance:** Low-Medium. Listed as official resource but community guide doesn't emphasize it heavily.

---

## Gap 10: Scenario 8 — Agentic AI Tools

**Source:** Community Guide notes this scenario has MISSING content.

Based on official Anthropic documentation, "Agentic AI Tools" likely covers:

**Server-side tools (Anthropic-executed):**
- `web_search` — search web for up-to-date information
- `web_fetch` — retrieve full content of web pages/PDFs
- `code_execution` — run Python and bash in sandboxed container
- `advisor` — let faster model consult higher-intelligence model mid-generation
- `tool_search` — discover and load tools on demand (for large tool sets)

**Anthropic-schema client tools (your code executes):**
- `memory` — store/retrieve info across conversations
- `bash` — run shell commands
- `text_editor` — view and modify text files
- `computer_use` — screenshots, mouse/keyboard control

**Key exam takeaway:** Server tools run on Anthropic infrastructure — you don't write handler code. Client tools require your application to execute the call and return results.

**Exam Relevance:** Medium. Scenario 8 has been reported by candidates but content is limited.

---

## Summary: All Gaps Identified & Status

| # | Topic | Gap Type | Status |
|---|-------|----------|--------|
| 1 | Mid-conversation system messages | Missing detail | 🔴 NEW - covered in this supplement |
| 2 | Format normalization rules | Missing | 🔴 NEW - covered in this supplement |
| 3 | Pydantic validation details | Missing | 🔴 NEW - covered in this supplement |
| 4 | Rendering by content type | Missing detail | 🔴 NEW - covered in this supplement |
| 5 | Incremental investigation strategy | Missing | 🔴 NEW - covered in this supplement |
| 6 | Read+Write fallback when Edit fails | Missing | 🔴 NEW - covered in this supplement |
| 7 | disable_parallel_tool_use | Missing | 🟡 In corrections file, now in supplement |
| 8 | strict: true for tool definitions | Missing | 🟡 In corrections file, now in supplement |
| 9 | Extended thinking | Missing detail | 🟡 Supplement covers basics |
| 10 | Scenario 8 - Agentic AI Tools | Community guide has gap | 🟡 Supplement adds coverage |

**Coverage check:** All 10 gaps from the community guide are now addressed. The original 110 concepts + these 10 supplements = complete coverage of the community guide.
