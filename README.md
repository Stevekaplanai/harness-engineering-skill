<div align="center">

# 🛠️ harness-engineering

**A Claude Code skill that makes Claude a harness engineer.**

Diagnose and fix unreliable AI agents by engineering the system around the model —
context delivery, tools, loops, memory, permissions, sandboxes, and verification.

[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-FF6A3D)](https://code.claude.com/docs/en/skills)
[![License: MIT](https://img.shields.io/badge/License-MIT-lightgrey.svg)](LICENSE)
[![Grounded in](https://img.shields.io/badge/grounded%20in-awesome--harness--engineering-blue)](https://github.com/ai-boost/awesome-harness-engineering)

*Built by [Steve Kaplan](https://github.com/Stevekaplanai) · creator of [AgentHost](https://agenthost.space) — your AI agent harness, running 24/7 in the cloud, live from any browser or phone.*

</div>

---

## Why

**Agent = model + harness.** When an agent fails, the model gets the blame — but the
harness is usually the binding constraint:

- Harness-only changes moved a coding agent from **rank 30 → top 5** on Terminal Bench 2.0 (LangChain)
- Shrinking the tool space took local models from **2/10 → 10/10** on a SWE-bench subset (statewright)
- Harness configuration alone swings benchmarks by **5+ percentage points** (Anthropic)
- Harness tuning got a model to within one point of a frontier model at **~1/10th the cost**

This skill loads that discipline into Claude Code. Instead of endlessly rewriting your
prompt, Claude classifies the failure — **context / constraint / verification / planning** —
and fixes the component that actually caused it.

## What Claude gets

| Piece | What it does |
|---|---|
| **Diagnostic workflow** | Symptom → failure class → harness component → smallest fix, with a symptom map for the 11 most common agent complaints |
| **8 operating principles** | Distilled from Anthropic, OpenAI, LangChain, Martin Fowler, Microsoft + 2026 research: design-for-removal, steering-in-context, navigation-over-loading, determinism at the edges, evals-as-training-data |
| **6 domain references** | Context & memory · tools/skills/MCP · loops & orchestration · security & permissions · verification & observability · foundations — each with patterns + annotated primary sources |
| **4 ready-to-copy templates** | `AGENTS.md`, `PLAN.md`, `IMPLEMENT.md`, and a pre-production `HARNESS_CHECKLIST.md` |

## Install

**Project-level** — applies to one repo:

```bash
git clone https://github.com/Stevekaplanai/harness-engineering-skill.git
mkdir -p .claude/skills
cp -r harness-engineering-skill/harness-engineering .claude/skills/
```

**Personal** — available everywhere:

```bash
mkdir -p ~/.claude/skills
cp -r harness-engineering-skill/harness-engineering ~/.claude/skills/
```

Start a new Claude Code session and it triggers automatically on agent-building and
agent-debugging work — or invoke it explicitly with `/harness-engineering`.

## When it fires

You don't have to say "harness." Any of these will do:

> *"My agent keeps looping and never finishes."*
> *"Claude forgets my style rules halfway through long tasks."*
> *"Design the tool set for my support agent."*
> *"Set up evals before we ship this agent."*
> *"Write an AGENTS.md for this repo."*
> *"Is this agent ready for production?"*

## Layout

```
harness-engineering/
├── SKILL.md                              # methodology + triggering
├── references/                           # loaded on demand, per problem
│   ├── context-and-memory.md             # compaction, caching, navigation > loading
│   ├── tools-and-interfaces.md           # tool design, MCP, skills
│   ├── loops-and-orchestration.md        # stop conditions, planning, multi-agent, HITL
│   ├── security-and-permissions.md       # sandboxes, injection, the lethal trifecta
│   ├── verification-and-observability.md # evals, tracing, debugging method
│   └── foundations-and-resources.md      # canonical essays + codebases to study
└── templates/                            # copy into your project and fill in
    ├── AGENTS.md
    ├── PLAN.md
    ├── IMPLEMENT.md
    └── HARNESS_CHECKLIST.md
```

## The core move

When an agent misbehaves, Claude works the failure back to a component instead of
guessing at prompts:

| Failure class | The agent... | Fix lives in |
|---|---|---|
| **Context** | didn't know something it needed | context delivery, compaction, memory |
| **Constraint** | did something it shouldn't have | permissions, hooks, tool scoping |
| **Verification** | believed it succeeded but hadn't | feedback loops, tests it can run |
| **Planning** | lost the thread on a long task | planning artifacts, decomposition, handoffs |

One change at a time, verified against the same failing scenario, with a note on when the
scaffolding can be removed — because every harness component is a bet that the model
can't do something *yet*.

## Attribution

Curated links, templates, and much of the distilled guidance derive from
[ai-boost/awesome-harness-engineering](https://github.com/ai-boost/awesome-harness-engineering)
(CC0) — check it for newly published resources. This skill is a point-in-time
distillation (July 2026).

## License

[MIT](LICENSE)

---

<div align="center">

**Want your harness running around the clock?**
[**AgentHost**](https://agenthost.space) moves your local Claude Code setup to a 24/7
container in *your own* cloud account — live terminal in any browser, phone included.

</div>
