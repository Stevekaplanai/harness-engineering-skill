# Agent Loops, Planning & Orchestration

The loop (observe → plan → act → verify) is the harness's skeleton. Loop structure — not
model identity — largely determines agent behavior; the same loop ported across model
backends behaves recognizably the same (deepclaude), and 60% of surveyed agent systems
converge on the same Agent Loop pattern.

## Design patterns

### The single-agent loop

- **Explicit stop conditions and budgets.** Every loop needs deterministic termination:
  step limits, token budgets, wall-clock timeouts, and a definition of "done" the agent
  can check. Match loop primitive to task shape (turn-based, goal-based, time-based)
  rather than defaulting to one conversational turn cycle.
- **Loop detection.** Agents repeat failing actions; middleware that detects repetition
  and injects a nudge (or halts) was one of the changes that moved LangChain's agent from
  rank 30 to top 5 on Terminal Bench.
- **Spend reasoning where it pays.** The "reasoning sandwich": maximum thinking budget at
  planning and verification phases, lean execution in between. If using extended thinking
  with tool use, preserve thinking blocks when returning tool results — dropping them
  silently breaks multi-step reasoning.
- **Inject situational context.** Directory maps, current time, deadlines, and time-budget
  warnings measurably improve behavior — temporal awareness in particular must be fed in;
  models don't track it on their own.
- **Hooks for deterministic control.** Lifecycle hooks (SessionStart, PreToolUse,
  PostToolUse…) are where guardrails, audits, and policy live — enforcement the model
  can't talk its way around. Claude Code, Codex, and LangChain middleware
  (`before_model`, `wrap_tool_call`, …) all expose this; use it instead of prompt-level trust.
- **Honor runtime semantics the model learned.** e.g., interpreter state persistence:
  mismatching persistence modes produces 80% missing-variable errors or 3.5× token
  overhead. When a runtime behaves unusually, say so in context.

### Planning & long-horizon work

- **Planning artifacts as harness state.** Plan.md / Implement.md files (see bundled
  `templates/`) survive compaction, transfer across sessions, and give humans an audit
  surface. The agent updates them during execution, not after.
- **Milestones with verification gates.** Each milestone gets its own "verify:" command;
  don't mark complete until it passes.
- **Separate planning from execution** when tasks are long: a planner produces the step
  list; an executor works through it, replanning only on failure (Plan-and-Execute).
  Decompose into a dependency graph so replanning is localized instead of cascading (TDP).
- **Cross-session handoff.** For tasks exceeding one context window: an initializer agent
  sets up the environment once; successor sessions get structured handoff state — feature
  lists, git commits, test gates (Anthropic's long-running-agents pattern). Git is the
  natural checkpoint/undo mechanism (Aider). For multi-day pipelines, checkpoint/hibernate
  and wake (Meta's REA).
- **The plan can live in code.** For work too large for any context window, generate an
  orchestration script that fans out to subagents (Claude Code dynamic workflows; used for
  a 750k-line port). Executable plans scale further than in-context plans.

### Multi-agent orchestration

- **Don't start multi-agent.** Add topology only when a measured problem demands it —
  context isolation, parallelism, or specialization. Subagents' main value is context
  isolation (67% fewer tokens than skills in multi-domain tests) — they're context
  firewalls, not just parallelism.
- **Multi-agent systems are distributed systems.** Every handoff needs typed schemas,
  constrained action spaces, and boundary validation (GitHub's failure analysis). Untyped
  natural-language handoffs are where work silently dies.
- **Pick topology by decision framework, not fashion:** subagents (context isolation),
  skills (shared context, distributed authoring), handoffs (sequential specialization),
  router (dispatch). Topology choice alone moves performance 12–23% (AdaptOrch).
- **Coordination without an orchestrator** works at small scale: agents claiming tasks
  via files in a shared git repo, collisions resolved by git itself (Anthropic's 16
  parallel Claudes building a C compiler).
- **Deterministic orchestration layers** (YAML/DSL-defined pipelines: Conductor, Roast,
  Hive) make workflows version-controlled and diffable — interleave deterministic steps
  with agentic ones instead of making everything agentic.

### Human-in-the-loop

- Three postures: humans **outside** the loop (review outputs), **in** the loop (approve
  actions), **on** the loop (maintain the harness itself). Only "on the loop" scales with
  agent throughput (Fowler); design toward it.
- Approval fatigue is real: users approve ~93% of prompts, making approvals meaningless.
  Replace blanket prompts with risk-tiered gates (auto-approve safe classes, escalate
  flagged actions — Anthropic's auto-mode two-stage classifier) and adaptive autonomy
  (trust grows with track record).
- Support "approve with changes" (modify tool input before execution), not just
  allow/deny. Structured clarifying questions mid-task (an `ask_human` tool) beat silent
  guessing — agents are measurably bad at knowing when to ask (HiL-Bench).
- Human corrections are training data for the harness: route them into prompt/tool/memory
  improvements, not just one-off overrides.

## Curated resources

**Loops**
- [ReAct](https://arxiv.org/abs/2210.03629) — the foundational Thought/Action/Observation loop.
- [Unrolling the Codex Agent Loop](https://openai.com/index/unrolling-the-codex-agent-loop/) — canonical decomposition of one iteration.
- [Improving Deep Agents with Harness Engineering](https://blog.langchain.com/improving-deep-agents-with-harness-engineering/) — rank 30 → top 5 via loop/context/verification changes; the single best case study.
- [Getting started with loops](https://claude.com/blog/getting-started-with-loops) — taxonomy: turn/goal/time-based loops, stop conditions.
- [How Middleware Lets You Customize Your Agent Harness](https://blog.langchain.com/how-middleware-lets-you-customize-your-agent-harness/) — six composable loop hooks.
- [Hooks – Codex](https://developers.openai.com/codex/hooks) — lifecycle hook reference.
- [Extended Thinking — Claude API Docs](https://docs.anthropic.com/en/docs/build-with-claude/extended-thinking) — thinking-block preservation rules.
- [LangGraph low-level concepts](https://langchain-ai.github.io/langgraph/concepts/low_level/) — loop as typed graph with checkpointing.
- [A Scheduler-Theoretic Framework for LLM Agent Execution](https://arxiv.org/abs/2604.11378) — loop/event/state-machine/graph trade-offs across 70 systems.
- [The Design Space of Today's and Future AI Agent Systems](https://arxiv.org/abs/2604.14228) — reverse-engineering Claude Code's five-stage compaction, hooks, subagent isolation.

**Planning & long-horizon**
- [Run Long-Horizon Tasks with Codex](https://developers.openai.com/blog/run-long-horizon-tasks-with-codex/) — Plan.md/Implement.md as harness artifacts.
- [Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) — initializer/coder handoff, cross-session state.
- [Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps) — multi-session planning; assumptions expire.
- [Plan-and-Execute Agents](https://blog.langchain.com/plan-and-execute-agents/) + [Plan-and-Act](https://arxiv.org/abs/2503.09572) + [TDP](https://arxiv.org/abs/2601.07577) — planner/executor separation.
- [Building a C Compiler with Parallel Claudes](https://www.anthropic.com/engineering/building-c-compiler) — orchestrator-free coordination, terse feedback loops.
- [Introducing dynamic workflows in Claude Code](https://claude.com/blog/introducing-dynamic-workflows-in-claude-code) — plans as executable orchestration code.

**Orchestration frameworks** (pick one; don't hand-roll coordination)
- [LangGraph](https://github.com/langchain-ai/langgraph) — graph state machines, checkpoint persistence; most adopted.
- [OpenAI Agents SDK](https://github.com/openai/openai-agents-python) — handoffs + guardrails.
- [Google ADK](https://github.com/google/adk-python) — code-first, Runner/AgentTool patterns, eval pipeline.
- [Claude Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) — inherit Claude Code's whole loop/tool layer programmatically.
- [AutoGen](https://github.com/microsoft/autogen) / [CrewAI](https://github.com/crewAIInc/crewAI) / [Pydantic AI v2](https://github.com/pydantic/pydantic-ai) / [Mastra](https://github.com/mastra-ai/mastra) / [Vercel AI SDK](https://github.com/vercel/ai) — alternatives by language/style.
- [Conductor](https://github.com/microsoft/conductor) / [Shopify Roast](https://github.com/Shopify/roast) / [Hive](https://github.com/aden-hive/hive) — deterministic workflow layers.
- [Choosing the Right Multi-Agent Architecture](https://blog.langchain.com/choosing-the-right-multi-agent-architecture/) — the topology decision framework.
- [Multi-Agent Workflows Often Fail](https://github.blog/ai-and-ml/generative-ai/multi-agent-workflows-often-fail-heres-how-to-engineer-ones-that-dont/) — typed handoffs, boundary validation.
- [Scaling Managed Agents](https://www.anthropic.com/engineering/managed-agents) — brain/hands/session separation; crash recovery by event replay.

**Human-in-the-loop**
- [Humans and Agents in Software Engineering Loops](https://martinfowler.com/articles/exploring-gen-ai/humans-and-agents.html) — outside/in/on the loop.
- [Claude Code Auto Mode](https://www.anthropic.com/engineering/claude-code-auto-mode) — replacing approval fatigue with a two-stage classifier.
- [Claude Agent SDK — user input](https://platform.claude.com/docs/en/agent-sdk/user-input) — canUseTool, approve-with-changes.
- [Measuring AI Agent Autonomy in Practice](https://www.anthropic.com/news/measuring-agent-autonomy) — adaptive permission data from real usage.
- [HiL-Bench](https://arxiv.org/abs/2604.09408) — do agents know when to ask for help?
- [LangGraph HITL concepts](https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/) + [AWS HITL patterns](https://github.com/aws-samples/sample-human-in-the-loop-patterns) + [HITL Protocol](https://github.com/rotorstar/hitl-protocol) — implementation references.
