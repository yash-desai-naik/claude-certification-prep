# Claude Certified Architect Foundations (CCA-F) — Exam Overview

---

## What Is This Certification?

CCA-F is an official Anthropic certification for solution architects who build production systems with Claude. It tests your ability to design agent architectures, integrate tools via MCP, configure Claude Code, engineer prompts, and manage context at scale.

This is NOT a multiple-choice trivia test. It is a **scenario-based exam** — you get realistic situations and must pick the right design decision.

---

## Who Should Take It

**Target candidate:** Solution architect or senior developer with **6+ months hands-on experience** building with Claude.

You should already know:
- How agentic loops work (tool calls, stop_reason, tool_use vs end_turn)
- Basics of MCP servers and tools
- Prompt engineering patterns (few-shot, structured output)
- Claude Code configuration (CLAUDE.md, rules, commands, skills)

This guide assumes you have that experience. It focuses on **exam strategy**, not teaching fundamentals from scratch.

---

## Exam Format

| Feature | Detail |
|---|---|
| Question type | Multiple choice — **1 correct answer out of 4** |
| Scoring scale | 100 – 1000 |
| Passing score | **720** |
| Guessing penalty | **None** — always answer, even if unsure |
| Scenario pool | 8 total scenarios, **4 randomly selected** per attempt |
| Time limit | Not publicly specified (scenario-based MCQs, manage time wisely) |
| Delivery | Online proctored / at test center |

---

## Scoring Detail

- Scale: 100 to 1000
- Passing: **720**
- No penalty for wrong answers — **never leave a question blank**
- Each question carries equal weight toward your final score
- Domain weights determine how many questions you get per domain

---

## Question Types

### Standard Multiple Choice
Straightforward knowledge check. One best answer.

**Example:**
> In an agentic loop, which `stop_reason` signals that the model has generated a complete response and is not requesting tool execution?
>
> A) `tool_use`
> B) `end_turn`
> C) `max_tokens`
> D) `stop_sequence`
>
> Correct: **B**

### Scenario-Based Multiple Choice
A realistic situation with context, followed by a design question. Tests applied knowledge.

**Example:**
> You are designing an agent that reviews pull requests. The agent must verify code style, run unit tests, and check for security vulnerabilities — in that specific order, every time. Which task decomposition strategy should you use?
>
> A) Dynamic adaptive decomposition
> B) Fixed pipeline (prompt chaining)
> C) Coordinator-subagent pattern
> D) Multi-pass code review
>
> Correct: **B**

---

## Scenario-Based Nature (Important!)

- The exam has a pool of **8 scenarios**
- Each attempt randomly picks **4 scenarios**
- Each scenario has **multiple questions** attached to it
- You won't know which 4 you get until you sit the exam
- This means you must study ALL domains — you can't skip one and hope it doesn't appear

Key implication: **Prepare across all 5 domains equally.** Domain 1 is heaviest (27%), but a scenario from Domain 5 (15%) can still sink you if you ignore it.

---

## Domain Weightage Table

| # | Domain | Weight | Priority |
|---|---|---|---|
| 1 | Agent Architecture and Orchestration | **27%** | High |
| 2 | Tool Design and MCP Integration | **18%** | Medium |
| 3 | Claude Code Configuration and Workflows | **20%** | Medium |
| 4 | Prompt Engineering and Structured Output | **20%** | Medium |
| 5 | Context Management and Reliability | **15%** | Low |

---

## High Scoring Strategy

### Focus on Domain 1 (27%)
It carries the most weight. Master agentic loops, coordinator-subagent patterns, and task decomposition. If you ace Domain 1, you are more than a quarter of the way to passing.

### Don't Ignore Domain 5 (15%)
It has the lowest weight, but scenario questions can come from any domain. Context management and error propagation are common failure points in real systems — expect at least one question from this domain.

### Answer Every Question
No guessing penalty. If you are stuck, eliminate the two obviously wrong options and pick between the remaining two. You have a 50% chance of getting it right.

### Manage Scenario Time
You get 4 scenarios with multiple questions each. If one scenario feels too hard, flag the questions and come back later. Don't let one scenario eat all your time.

### Read the FULL Scenario Before Answering
Scenarios contain context that directly eliminates wrong answers. Skipping a detail in the scenario description is the #1 reason candidates get questions wrong.

### Use Process of Elimination
With 4 options and 1 correct answer, eliminate the options that:
- Violate Anthropic best practices (e.g., parsing assistant text for completion)
- Suggest anti-patterns (arbitrary iteration limits)
- Use wrong concepts for the task (e.g., Batch API for real-time checks)

---

## Mistakes Students Make

| Mistake | Why It Hurts |
|---|---|
| Reading too fast and missing scenario details | Scenario context directly rules out wrong answers |
| Treating it like a trivia test | It's scenario-based — you apply knowledge, not recall facts |
| Ignoring Domains 3, 4, or 5 because they weigh less | Scenarios randomly pick from all domains |
| Overthinking the wrong answer | If two options feel similar, one usually violates a best practice — find it |
| Leaving questions blank | No guessing penalty — always pick something |
| Memorizing instead of understanding | Questions test applied design decisions, not definitions |
| Focusing only on strengths | Balanced preparation across all 5 domains is safer |
| Not managing time per scenario | Hard scenarios can eat up time from easier ones |

---

## 2-Day Study Plan (Crunch Mode)

### Day 1: Foundation (6–8 hours)

| Time | Activity |
|---|---|
| 1 hour | Read this Exam Overview (you just did it) |
| 2 hours | Domain 1 — Agent Architecture (heaviest domain, do it first) |
| 1.5 hours | Domain 2 — Tool Design and MCP Integration |
| 1.5 hours | Domain 4 — Prompt Engineering and Structured Output |
| 30 min | Quick scan: Domains 3 and 5 |

**Evening:** Review 20 flashcards from each domain.

### Day 2: Application (6–8 hours)

| Time | Activity |
|---|---|
| 2 hours | Domain 3 — Claude Code Configuration (practical, easy to test) |
| 1.5 hours | Domain 5 — Context Management and Reliability |
| 2 hours | Mock exam questions from all domains |
| 1 hour | Review wrong answers — understand WHY you got it wrong |
| 1 hour | Architecture diagrams — draw key patterns from scratch |
| 30 min | Final review of cheat sheets |

---

## 3-Day Study Plan (Recommended)

### Day 1: Core Architecture (4–5 hours)

| Time | Activity |
|---|---|
| 30 min | Read Exam Overview |
| 2 hours | Domain 1 — Agent Architecture and Orchestration |
| 1.5 hours | Domain 2 — Tool Design and MCP Integration |
| 30 min | Make 10 flashcards for tricky concepts |

### Day 2: Configuration + Prompts (4–5 hours)

| Time | Activity |
|---|---|
| 1.5 hours | Domain 3 — Claude Code Configuration and Workflows |
| 1.5 hours | Domain 4 — Prompt Engineering and Structured Output |
| 1.5 hours | Mixed practice questions (10 per domain) |
| 30 min | Review mistakes |

### Day 3: Reliability + Mock Exam (4–5 hours)

| Time | Activity |
|---|---|
| 1.5 hours | Domain 5 — Context Management and Reliability |
| 2 hours | Full-length mock exam (timed) |
| 1 hour | Review wrong answers + weak areas |
| 30 min | Draw architecture diagrams from memory |

---

## 1-Day Revision Plan (Last Day Before Exam)

| Time | Activity |
|---|---|
| 30 min | Read cheat sheets for all 5 domains |
| 30 min | Review flashcards — focus on ones you keep getting wrong |
| 1 hour | Solve 10 MCQ questions from each domain |
| 45 min | Draw key architecture diagrams from memory (agentic loop, coordinator-subagent, MCP server flow) |
| 30 min | Read this Exam Overview again — especially Domain weights and Scoring strategy |
| 15 min | Go through Night Before + Exam Day checklists |
| Stop | Stop studying. Relax. Get good sleep. |

---

## Night Before Exam Checklist

```
[  ] Review Domain weightage — remind yourself Domain 1 is 27%
[  ] Read cheat sheets one final time
[  ] Draw 3 architecture diagrams from memory (agentic loop, coordinator-subagent, MCP flow)
[  ] Review 10 flashcards
[  ] Charge your laptop
[  ] Test your internet connection
[  ] Find a quiet, distraction-free room
[  ] Keep water and a light snack ready
[  ] Set 2 alarms
[  ] Sleep early — 7-8 hours minimum
[  ] NO last-minute cramming — if you don't know it by now, you won't learn it tonight
```

---

## Exam Day Checklist

```
[  ] Wake up early — leave buffer time
[  ] Have a light, protein-rich breakfast
[  ] No heavy food before exam (makes you sleepy)
[  ] Review cheat sheets only (30 min max, no deep study)
[  ] Check internet connection again
[  ] Close all browsers, apps, notifications
[  ] Keep water and ID ready
[  ] Use the restroom before starting
[  ] Take a deep breath
[  ] Read every scenario completely before answering
[  ] Eliminate wrong answers first
[  ] If stuck, skip and return later
[  ] Never leave a question blank
[  ] Manage time — 4 scenarios, pace yourself
[  ] Celebrate after — you earned it
```

---

## Quick Reference

| Item | Value |
|---|---|
| Certification | CCA-F (Claude Certified Architect Foundations) |
| Passing score | 720 / 1000 |
| Question format | Multiple choice (1 of 4) |
| Scenarios | 8 in pool, 4 per attempt |
| Guessing penalty | None |
| Heaviest domain | Agent Architecture and Orchestration (27%) |
| Target experience | 6+ months hands-on with Claude |
| Priority by domain | D1 > D3 = D4 > D2 > D5 |
