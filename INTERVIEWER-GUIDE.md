# Interviewer Guide: Staff Software Engineer, AI

## Overview

This interview evaluates a candidate's ability to read a technical document, synthesize its concepts, and orchestrate AI coding agents to build a working system. The task is to build an **eval system for an email-processing AI agent** based on the Anthropic blog post on agent evals.

**Duration**: ~90 minutes
**Artifacts in this repo**: Reading material (condensed blog post), email scenario JSON files, this guide.

---

## Timeline

| Phase | Duration | What Happens |
|-------|----------|-------------|
| **Intro & Setup** | 5 min | Explain the exercise, confirm they have their tools ready |
| **Silent Reading** | 10 min | Candidate reads `reading-material.md` (condensed Anthropic blog post) |
| **Build Phase 1** | 25 min | Candidate builds general eval system from concepts in the reading |
| **In-Between Q1** | 5 min | Email domain integration question (introduce `data/` scenarios) |
| **Build Phase 2** | 25 min | Candidate integrates email scenarios into their eval system |
| **In-Between Q2-3** | 5 min | Attachment handling + non-LLM integration questions |
| **Build Phase 3** | 10 min | Candidate addresses advanced scenarios (safety, ambiguity) |
| **Wrap-up** | 5 min | Candidate walks through their system, discusses tradeoffs |

---

## Phase Details

### 1. Intro (5 min)

Say something like:

> "Today we're going to do something a bit different. Instead of whiteboard coding, we want to see how you work with AI tools to bring a technical concept to life. You'll read a condensed technical article about building evals for AI agents, then use your preferred AI coding tools to build an eval system. Use whatever tools you normally use — Claude Code, Cursor, Copilot, whatever you're comfortable with. We want to see your real workflow."

Confirm:
- They have their dev environment ready
- They can clone/access this repo
- They understand they'll be building, not just designing

### 2. Silent Reading (10 min)

Direct them to `reading-material.md`. This is a condensed version of the Anthropic blog post "Demystifying Evals for AI Agents."

**What to observe**: Do they take notes? Do they seem to grasp the hierarchy of concepts (tasks → graders → suites → harness)?

### 3. Build Phase 1 — General Eval System (25 min)

Prompt:

> "Now build an eval system based on what you just read. Start with the general framework — we'll plug in a specific domain shortly. Use your AI tools however you like."

**What to watch for** (see scorecard below):
- Do they paste/reference the article in their AI prompts? (+1 Context Inclusion)
- Do they decompose into subsystems? (+1 System Split)
- Do they fire off parallel agents? (+1 Orchestrating Agents)
- Do they run QA/tests alongside building? (+1 QA Agents)

### 4. In-Between Question 1 — Email Domain (5 min)

Once they have a general framework, ask:

> "Now let's make this concrete. We have an email-processing AI agent that classifies vendor emails, extracts invoice data, decides on escalations, and determines next actions. How would you integrate email conversation evaluation into your general eval system? Take a look at `data/email-scenarios-basic.json` for the scenario format."

**What you're probing**: Can they take a general system and specialize it for a new domain? Do they see natural boundaries between classification scoring, extraction scoring, and workflow scoring?

### 5. Build Phase 2 — Email Integration (25 min)

Let them integrate the email scenarios. Point them to both JSON files if they haven't found them.

**Key scenarios to discuss if they surface them:**
- `email_003` (vacation auto-reply): Tests classification + CC extraction — two scoring dimensions from one email
- `email_006` (multi-invoice partial match): Tests partial credit grading — not just pass/fail
- `email_008` (prompt injection): Tests safety awareness

### 6. In-Between Questions 2 & 3 (5 min)

**Q2 — File Attachments:**
> "How would you architect the system to handle file attachments — PDFs, spreadsheets, source code — as inputs to your eval pipeline?"

Look for: Do they think about attachment parsing as a separate subsystem? Do they consider mock attachments for reproducible evals? Do they think about token/cost implications?

**Q3 — Non-LLM Integration:**
> "How would you integrate non-LLM systems — a database lookup, a rules engine, a traditional backend service — into the agent's evaluation pipeline?"

Look for: Do they think about mocking external services for deterministic evals? Do they distinguish between testing the agent's decision-making vs. testing the integration? Do they consider environment isolation (from the reading)?

### 7. Build Phase 3 — Advanced Scenarios (10 min)

If time allows, direct them to `data/email-scenarios-advanced.json` and specifically:
- `email_008` (prompt injection) — do they build safety-aware grading?
- `email_010` (ambiguous intent) — do they build flexible grading that accepts multiple valid answers?

### 8. Wrap-up (5 min)

> "Walk me through the system you built. What are the grader types? How do scenarios flow through? What would you add with more time?"

---

## Scorecard

Score each criterion 0 or +1. **Target: 8+/12 for strong hire.**

| # | Criterion | +1 Signal | 0 Signal |
|---|-----------|-----------|----------|
| 1 | **Context Inclusion** | Includes blog link or content in AI prompts | Prompts from scratch with no grounding |
| 2 | **Conceptual Hierarchy** | Decomposes: tasks, graders, suites, harness | Flat structure, no layering |
| 3 | **System Split** | Separates concerns (graders, harness, scenarios, reporting) | Monolithic single-file approach |
| 4 | **Subsystem Flow** | Clear data flow: scenario → solver → graders → report | Disconnected pieces |
| 5 | **Systems Understanding** | System works cohesively end-to-end | Parts exist but don't connect |
| 6 | **End Goal Focus** | Clear goal in prompts ("build an eval harness that...") | Vague prompting ("write some code for evals") |
| 7 | **Prompt Refinement** | Iterates prompts, refines based on output | Single prompt, accepts first output |
| 8 | **Orchestrating Agents** | Runs multiple agents/tabs in parallel | Sequential single-agent workflow |
| 9 | **QA Agents** | Spawns agents to test/review alongside building | No testing until the very end |
| 10 | **Evaluate While Running** | Monitors agent output, corrects course mid-stream | Set-and-forget, reviews only at end |
| 11 | **AI Safety** | Considers prompt injection, PII, data leakage, unbounded access | No mention of safety |
| 12 | **Guide to Completion** | Drives agents to a working, demo-able system | Leaves things half-built |

---

## What Good Looks Like

A strong candidate (30-40 min in) will have:

1. **Eval harness** that loads scenario JSON, runs scenarios through a solver, and applies graders
2. **Multiple grader types**: at minimum a deterministic grader (string/value matching) and an LLM-based grader (rubric scoring)
3. **Email-specific scoring**: classification accuracy, invoice extraction precision/recall, escalation correctness
4. **Partial credit**: not just pass/fail but graduated scoring for multi-component tasks
5. **Isolation awareness**: each scenario runs independently
6. **Safety consideration**: at least mentions prompt injection or data leakage risks
7. **Clear subsystem boundaries**: scenarios, solver interface, graders, and reporting are separate

A **great** candidate will also:
- Have tests running alongside the build
- Handle the ambiguous scenario (`email_010`) with flexible grading
- Discuss capability vs. regression eval distinction from the reading
- Build something that actually runs and produces output

---

## Red Flags

- Cannot synthesize the article concepts after 10 min of reading
- Writes code manually instead of leveraging AI tools
- Single long prompt with no iteration
- No awareness of grader types (only uses one approach)
- Cannot explain what their agents are building
- Ignores the prompt injection scenario entirely
- Cannot connect the general framework to the email domain

---

## Anti-Patterns to Avoid (as Interviewer)

- **Don't penalize tool choice**: Cursor, Claude Code, Copilot, ChatGPT — all valid
- **Don't penalize language choice**: Python, TypeScript, whatever they're productive in
- **Don't expect the reading to be memorized**: They can reference it during building
- **Don't rush the reading phase**: The 10 min is important for synthesis
- **Don't look for specific frameworks**: They don't need to use Inspect AI, Promptfoo, etc.
- **Do keep the energy collaborative**: This should feel like a fun pairing session, not an exam
