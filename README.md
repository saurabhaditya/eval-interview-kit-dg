# Email Agent Eval System — Interview Exercise

Welcome! In this exercise you'll read a technical article about building evaluations for AI agents, then use your preferred AI coding tools to build an eval system for an email-processing agent.

## What's in this repo

```
├── reading-material.md          # Condensed article (~10 min read) — start here
├── data/
│   ├── email-scenarios-basic.json      # 5 basic email test scenarios
│   └── email-scenarios-advanced.json   # 5 advanced scenarios (ambiguity, safety, attachments)
└── README.md                    # You are here
```

## Exercise Flow

1. **Read** `reading-material.md` — a condensed version of Anthropic's "Demystifying Evals for AI Agents"
2. **Build** an eval system based on the concepts from the reading
3. **Integrate** the email scenarios from `data/` into your eval system
4. **Discuss** tradeoffs, grader design, and architecture decisions with the interviewer

## The Email Agent

The system under test is an AI email agent that processes vendor emails for an accounts payable team. Given an email, the agent:

- **Classifies** it (confirmation, dispute, auto-reply, question, payment reminder, etc.)
- **Extracts** invoice IDs and amounts
- **Decides** whether to escalate to a human
- **Determines** the terminal state (processed, escalated, awaiting response, etc.)
- **Identifies** follow-up actions (writebacks, CC additions)

Your eval system should test these capabilities using the provided scenarios.

## Getting Started

Use whatever language, framework, and AI tools you prefer. The scenario JSON files define inputs and expected outputs — your eval system should load these, run them through grading logic, and report results.

Good luck!
