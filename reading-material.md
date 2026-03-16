# Demystifying Evals for AI Agents — Condensed Reading Guide

**Source**: [Anthropic Engineering Blog](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
**Reading time**: ~10 minutes

---

## 1. What Is an Eval?

An evaluation ("eval") is a test for an AI system: give an AI an input, apply grading logic to its output, measure success.

- **Single-turn evals**: prompt → response → grade
- **Multi-turn evals**: extended interactions where agents call tools across multiple steps
- **Agent evals**: additional complexity — frontier models discover creative solutions that may exceed static eval expectations

### Key Terminology

| Term | Definition |
|------|-----------|
| **Task** | Single test with defined inputs and success criteria |
| **Trial** | One attempt at a task; run multiple to account for variability |
| **Grader** | Scoring logic for agent performance; tasks support multiple graders |
| **Transcript** | Complete interaction record (outputs, tool calls, reasoning) |
| **Outcome** | Final environmental state post-trial |
| **Evaluation Harness** | End-to-end infrastructure: instructions, tools, execution, recording, grading |
| **Evaluation Suite** | Related task collection measuring specific capabilities |

---

## 2. Why Build Evals?

Without evals, teams get stuck in reactive loops: wait for complaints → manual reproduction → fix → hope nothing regresses.

**With evals you can:**
- Distinguish genuine regressions from noise
- Test hundreds of scenarios pre-deployment automatically
- Quantify improvement with metrics instead of guesswork
- Upgrade models in days instead of weeks of manual testing

**Real examples:**
- **Claude Code**: progressed from employee feedback → narrow evals (concision, file edits) → complex behavioral evals (over-engineering)
- **Descript**: evolved from manual grading → LLM graders with product-defined criteria → dual suites for quality + regression
- **Bolt**: static analysis grading + browser-agent testing + LLM judges for instruction-following

---

## 3. Grader Types

### Code-Based Graders
- String matching, regex, fuzzy matching
- Static analysis (linting, type checking, security scanning)
- Outcome verification, tool call verification
- Transcript analysis (turn counts, token usage)
- **Pros**: Fast, cheap, objective, reproducible
- **Cons**: Brittle against valid variations, lack nuance

### Model-Based Graders (LLM-as-Judge)
- Rubric-based scoring, natural language assertions
- Pairwise comparison, multi-judge consensus
- **Pros**: Flexible, captures nuance, handles open-ended tasks
- **Cons**: Non-deterministic, expensive, requires human calibration

### Human Graders
- Expert review, crowdsourced judgment, spot-check sampling
- **Pros**: Gold-standard quality, calibrates model-based graders
- **Cons**: Expensive, slow, doesn't scale

**Scoring**: Weighted (combined scores hit threshold), binary (all must pass), or hybrid.

---

## 4. Capability vs. Regression Evals

| | Capability Evals | Regression Evals |
|---|---|---|
| **Question** | "What can this agent do well?" | "Does it still handle previous tasks?" |
| **Target pass rate** | Low (agent struggles) | ~100% |
| **Purpose** | Advancement hills to climb | Detect drift |

**Graduation pattern**: High-performing capability evals transition to regression suites. Tasks progress from "Can we do this?" → "Can we still do this reliably?"

---

## 5. Agent-Type-Specific Approaches

### Coding Agents
- Grade via test suite execution (pass/fail)
- Add depth: code quality rubrics, static analysis, tool interaction assessment
- Example benchmarks: SWE-bench Verified, Terminal-Bench

### Conversational Agents
- Simulate user personas via a second LLM
- Multidimensional: ticket resolution (state check) + speed (turn count) + tone (LLM rubric)
- Example benchmarks: τ-Bench, τ2-Bench

### Research Agents
- Groundedness checks (claims supported by sources)
- Coverage checks (key facts required)
- Source quality checks (authoritative sources)
- Example benchmark: BrowseComp

### Computer Use Agents
- Run in real or sandboxed environments
- Verify outcomes via environment state inspection
- Example benchmarks: WebArena, OSWorld

---

## 6. Handling Non-Determinism

Agent behavior varies between runs. Two key metrics:

- **Pass@k**: Probability of ≥1 success in k attempts (use for coding agents where finding a solution matters)
- **Pass^k**: Probability of ALL k trials succeeding (use for customer-facing agents where reliability matters)

At k=1 they're identical. At k=10: pass@k → 100%, pass^k → 0%. Choose based on your reliability requirements.

---

## 7. Practical Roadmap (8 Steps)

1. **Start early** — 20-50 tasks from real failures is enough. Don't wait for perfection.
2. **Begin with manual checks** — Convert user-reported failures into test cases.
3. **Write unambiguous tasks** — Include reference solutions. If domain experts can't pass it, refine it.
4. **Build balanced problem sets** — Test when behaviors should AND shouldn't occur.
5. **Build stable eval harness** — Each trial needs isolated, clean environment. Shared state corrupts results.
6. **Design thoughtful graders** — Grade outputs not paths. Build partial credit. Calibrate LLM judges against humans.
7. **Check transcripts** — Read many trial transcripts. Verify evals measure what matters.
8. **Monitor for saturation** — 100% pass rate = no improvement signal. Graduate and create harder evals.

---

## 8. Example Task Structure (Conversational Agent)

```yaml
task:
  id: "refund-processing_1"
  desc: "Process refund for damaged item with empathetic communication"
  graders:
    - type: llm_rubric
      rubric: prompts/support_quality.md
      assertions:
        - "Agent showed empathy for customer's frustration"
        - "Resolution was clearly explained"
    - type: state_check
      expect:
        tickets: {status: resolved}
        refunds: {status: processed}
    - type: tool_calls
      required:
        - {tool: verify_identity}
        - {tool: process_refund, params: {amount: "<=100"}}
    - type: transcript
      max_turns: 10
  tracked_metrics:
    - n_turns, n_toolcalls, n_total_tokens
    - time_to_first_token, time_to_last_token
```

---

## 9. Key Takeaways

- **Start early**: Don't wait for hundreds of tasks. 20-50 from real failures is excellent.
- **Source realistic tasks** from observed failures and user reports.
- **Define unambiguous success criteria** — vagueness becomes metric noise.
- **Combine grader types** thoughtfully (deterministic where possible, LLM where needed, human for calibration).
- **Read the transcripts** — verify evals measure what actually matters.
- **No single evaluation layer catches everything** — combine evals, monitoring, A/B testing, and human review (Swiss Cheese Model).
