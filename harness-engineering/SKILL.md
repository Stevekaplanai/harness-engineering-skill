---
name: harness-engineering
description: >-
  Design, build, audit, and debug AI agent harnesses — the scaffolding around a model
  (context delivery, tool interfaces, agent loops, planning artifacts, memory, permissions,
  sandboxing, verification, observability) that determines whether an agent succeeds on real
  tasks. Use this skill whenever the user is building an AI agent or agentic system, or when
  an existing agent is unreliable, even if they never say the word "harness": an agent that
  keeps failing, loops, or forgets instructions; running out of context / compaction problems;
  designing or trimming tools and MCP servers; writing AGENTS.md or CLAUDE.md files; choosing
  a multi-agent or orchestration pattern; setting up evals, permissions, sandboxes, or memory
  for agents; making long-running agent tasks survive across sessions. Also use it to review
  an agent setup before production ("is this agent ready to ship?").
---

# Harness Engineering

Harness engineering is the discipline of designing the scaffolding that surrounds an AI
agent — context delivery, tool interfaces, planning artifacts, verification loops, memory
systems, permissions, and sandboxes. The unit of capability is not the model; it is
**agent = model + harness**, and in practice the harness is usually the binding constraint:

- Harness-only changes moved LangChain's coding agent from rank 30 to top 5 on Terminal
  Bench 2.0 — no model swap.
- Constraining the tool space per workflow phase took local models from 2/10 to 10/10 on a
  SWE-bench subset (statewright).
- Harness configuration alone swings benchmark results by 5+ percentage points
  (Anthropic, 2026 Agentic Coding Trends), and infrastructure noise (container resources)
  can swing them 6+ points more.
- Harness-only tuning brought a much cheaper model within one point of a frontier model at
  ~1/10th the cost (LangChain/Nemotron playbook).

So when an agent misbehaves, treat it as a harness defect first and a model limitation
last. Most "the model is dumb" complaints are configuration problems with a mechanical fix.

## Operating principles

These recur across every serious treatment of the subject (Anthropic, OpenAI, LangChain,
Fowler, Microsoft, and the research literature). Apply them before reaching for patterns.

1. **Every harness component encodes an assumption that the model can't do something —
   and those assumptions expire.** When you add scaffolding, write down what capability
   improvement would make it unnecessary (the bundled HARNESS_CHECKLIST.md has a table for
   this). Design for removal, not accretion.

2. **If guidance wasn't in the loaded context, it didn't happen** (Stripe's steering
   studies). Agents ignore passive documentation; they obey what is actually in the context
   window — system prompts, skill files, tool error messages, CLI output. Put steering
   where the model will actually see it, at the moment it acts.

3. **Never rely on compaction to preserve critical rules.** Compaction keeps the current
   task and recent errors; it loses initial instructions, style rules, and intermediate
   decisions. Anything that must survive belongs in a file that is re-read or re-injected
   (CLAUDE.md / AGENTS.md / a plan file), not in conversation history.

4. **The context window is a finite, curated resource.** Prefer navigation over bulk
   loading (search/index/pointer tools instead of whole-file reads), compress tool outputs
   before they enter context, and keep durable state in the filesystem. "What configuration
   of context produces the desired behavior?" is the design question — not "what prompt
   wording."

5. **Tool design is agent UX.** Few tools, unambiguous names, minimal schemas, error
   messages that say what to do next, consistent return shapes. Too many MCP servers bloat
   context and degrade every decision; curate to roughly a dozen tools an agent actually
   needs (Open SWE caps at ~15).

6. **Make the deterministic parts deterministic.** Permissions, protocol gates, schema
   validation, lint/test gates, and state-machine phase constraints belong in code and
   hooks the model cannot override — not in prompt-level trust. Multi-agent systems are
   distributed systems: every handoff needs typed schemas and boundary validation.

7. **Verification is part of the loop, not a post-hoc step.** The agent must be able to
   run its own verification command and see the result before declaring success. Feedback
   must be terse: a few summary lines into context, full detail to a log file.

8. **Evals are the training data for harness work.** You cannot tune what you cannot
   measure. Before iterating on a harness, define what "better" means with runnable checks;
   separate capability evals (low pass rate, improvement target) from regression evals
   (near-100%, protection target).

## Workflow: diagnosing and fixing an agent

When asked to fix or improve an agent, do not start by rewriting the prompt. Work the
failure back to a component:

1. **Reproduce and read the trace.** Get the actual transcript/trace of a failure, not a
   description of it. Look at what was in context at the decision point that went wrong.
2. **Classify the failure** (deepset's framework — each class maps to a different fix):
   - **Context failure** — the agent didn't know something it needed (missing, evicted,
     or buried). Fix context delivery, not the model.
   - **Constraint failure** — the agent did something it shouldn't have. Fix permissions,
     hooks, tool scoping, or state-machine gating.
   - **Verification failure** — the agent believed it succeeded but hadn't. Fix the
     feedback loop: tests it can run, error surfaces it can read.
   - **Planning failure** — the agent lost the thread on a long task. Fix planning
     artifacts, decomposition, milestones, or handoff state.
3. **Make the smallest harness change that addresses the class**, then re-run the same
   scenario. One variable at a time — harness changes compound and interact (Anthropic's
   April 2026 postmortem traced a visible quality regression to three independent,
   individually-minor harness changes).
4. **Record the assumption.** Note why the component exists and when it can be removed.

### Symptom → component map

| Symptom | Likely component | Reference |
|---|---|---|
| Forgets instructions mid-task, ignores style rules after a while | Context delivery / compaction | `references/context-and-memory.md` |
| Burns tokens re-reading files, context fills up fast | Context delivery (navigation, output compression) | `references/context-and-memory.md` |
| Doesn't retain anything across sessions; repeats old mistakes | Memory & state | `references/context-and-memory.md` |
| Wrong tool calls, malformed arguments, flailing between tools | Tool design / tool surface size | `references/tools-and-interfaces.md` |
| Ignores your SDK/library conventions | Skills / steering placement | `references/tools-and-interfaces.md` |
| Loops, stalls, never terminates, or stops too early | Agent loop (stop conditions, budgets, loop detection) | `references/loops-and-orchestration.md` |
| Falls apart on long/multi-day tasks | Planning artifacts + cross-session handoff | `references/loops-and-orchestration.md` |
| Subagents conflict or drop work at handoffs | Orchestration (typed handoffs, topology) | `references/loops-and-orchestration.md` |
| Does dangerous/unwanted things; approval fatigue | Permissions, sandboxing, HITL gates | `references/security-and-permissions.md` |
| Claims success on broken output | Verification loop | `references/verification-and-observability.md` |
| "It got worse and we don't know why" | Observability + regression evals | `references/verification-and-observability.md` |

## Building a harness from scratch

For a new agent, decide these in order — each layer constrains the next:

1. **Loop shape** — single conversational loop, plan-then-execute, or orchestrated
   subagents? Default to the simplest loop that fits; add topology only when a measured
   problem demands it (`references/loops-and-orchestration.md`).
2. **Tool surface** — the minimal set of well-named tools; bash + filesystem is often
   better than dozens of bespoke tools (Microsoft's SRE agent went from 100+ tools to
   files + grep and improved). (`references/tools-and-interfaces.md`)
3. **Context strategy** — what's always in context (conventions, constraints), what's
   loaded on demand (progressive disclosure), what lives in files.
4. **Control mechanisms** — permission rules, hooks, sandbox boundary, HITL gates
   (`references/security-and-permissions.md`).
5. **Verification** — the commands the agent runs to know it's done, wired into the loop
   (`references/verification-and-observability.md`).
6. **Instrumentation** — tracing from day one; you will need it the first time something
   inexplicable happens (`references/verification-and-observability.md`).

Then instantiate the planning artifacts:

## Bundled templates

Copy from `templates/` into the user's project and fill in — don't invent new formats
when these fit:

- **`templates/AGENTS.md`** — project-level agent instructions: conventions, explicit
  tool permissions (allowed / restricted / not allowed), verification gates. Works as a
  base for CLAUDE.md too.
- **`templates/PLAN.md`** — task planning artifact: milestones each with its own
  verification command, scope boundaries, running decision log. Create at task start for
  any non-trivial task; the agent updates it during execution.
- **`templates/IMPLEMENT.md`** — append-only implementation log: decisions, deviations
  from plan, open questions. Pairs with PLAN.md on long-running work.
- **`templates/HARNESS_CHECKLIST.md`** — pre-production review checklist across
  instructions, tools, context, planning, permissions, and verification — plus the
  "when should this component be removed" table. Use it when asked to review or audit an
  agent setup.

## Reference files

Read the one that matches the problem; each contains distilled design patterns followed by
a curated, annotated resource list (from ai-boost/awesome-harness-engineering) for deeper
research with WebFetch when you need a primary source.

- `references/context-and-memory.md` — context delivery, compaction, prompt caching,
  navigation-over-loading, memory architectures, cross-session state.
- `references/tools-and-interfaces.md` — tool naming/schema/error design, MCP, skills and
  progressive disclosure, code-execution-over-tool-calls.
- `references/loops-and-orchestration.md` — loop anatomy and stop conditions, planning
  artifacts, long-horizon/multi-session patterns, multi-agent topologies, human-in-the-loop.
- `references/security-and-permissions.md` — permission models, sandboxing options,
  prompt injection defenses, credential isolation, the lethal trifecta.
- `references/verification-and-observability.md` — verification loops, eval design, CI
  integration, tracing, debugging methodology, cost controls.
- `references/foundations-and-resources.md` — canonical essays, reference implementations
  worth studying, meta-harness/self-improving-harness work, production case studies. Read
  when the user wants background, a reading list, or examples to model a design on.

## Scope notes

- This skill is model-agnostic: the same discipline applies to Claude Code, Codex, custom
  Agent SDK apps, LangGraph/ADK/CrewAI systems, or hand-rolled loops.
- Distinguish harness engineering from prompt engineering: if the fix is "explain the task
  better," it's a prompt; if the fix changes what the agent can see, do, or verify — the
  loop, tools, context, state, or gates — it's the harness. This skill is for the latter.
- "Harness" here is the AI-agent scaffolding sense, not Harness the CI/CD company.
