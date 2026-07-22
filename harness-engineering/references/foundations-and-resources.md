# Foundations, Reference Implementations & Further Study

Use this file when the user wants conceptual grounding, a reading list, a codebase to
model a design on, or the frontier (self-improving harnesses). Everything here is curated
from [ai-boost/awesome-harness-engineering](https://github.com/ai-boost/awesome-harness-engineering)
(CC0), which is also the place to check for updates.

## The concept in four framings

1. **Constitutive definition** ("What makes a harness a harness," arXiv 2606.10106): a
   runtime layer with four necessary and sufficient elements — an agent loop, a tool
   interface, context management, and control mechanisms. Use this as the inclusion test
   for whether something is a harness vs. a wrapper, guardrail, or generator.
2. **Fowler's three interlocking systems:** context engineering (curating what the agent
   knows), architectural constraints (deterministic linters and structural tests), and
   entropy management (periodic agents repairing drift) — maintained by humans *on* the
   loop rather than reviewing each output.
3. **LangChain's five primitives:** filesystem (durable state), code execution
   (autonomous problem-solving), sandbox (isolation + verification), memory (cross-session
   persistence), context management (compaction against context rot). Warning attached:
   models co-evolve with harnesses and can overfit to a harness design.
4. **The expiry principle** (Anthropic): every harness component assumes the model can't
   do something; those assumptions expire as models improve. The best harnesses are
   designed knowing their components will become unnecessary — keep/add/remove scaffolding
   deliberately over time.

## Canonical essays (read in roughly this order)

- [Harness Engineering](https://openai.com/index/harness-engineering/) — OpenAI's framing of the discipline.
- [Building Effective Agents](https://www.anthropic.com/research/building-effective-agents) — workflows vs. agents; composing primitives.
- [Harness Engineering (Fowler)](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html) — the clearest conceptual map of the practice.
- [The Anatomy of an Agent Harness](https://blog.langchain.com/the-anatomy-of-an-agent-harness/) — the five primitives.
- [Harness engineering for coding agent users (Böckeler)](https://martinfowler.com/articles/harness-engineering.html) — feedforward guides + feedback sensors; "harnessability" as an architecture criterion.
- [Agent Harness Design: 3 Patterns](https://claude.com/blog/harnessing-claudes-intelligence) — build on known tools, remove expired assumptions, set boundaries.
- [Skill Issue: Harness Engineering for Coding Agents](https://www.humanlayer.dev/blog/skill-issue-harness-engineering-for-coding-agents) — most agent failures are configuration problems; the single best one-page synthesis.
- [What makes a harness a harness](https://arxiv.org/abs/2606.10106) — the formal definition, applied to Claude Code/Codex/Aider/OpenHands/SWE-agent.
- [Architectural Design Decisions in AI Agent Harnesses](https://arxiv.org/abs/2604.18071) — 70-system empirical study; five recurring design dimensions.
- [Harness Engineering: How to Build Reliable AI Agents by Engineering the System, Not the Model](https://www.deepset.ai/blog/harness-engineering) — the failure-classification framework (context/constraint/verification/planning) this skill's workflow uses.
- [Tuning the harness, not the model](https://blog.langchain.com/tuning-the-harness-not-the-model-a-nemotron-3-ultra-playbook) — near-frontier quality at ~1/10th cost via harness fit.
- [RUCAIBox/awesome-agent-harness](https://github.com/RUCAIBox/awesome-agent-harness) — 500+ reference academic survey.
- [lopopolo/harness-engineering](https://github.com/lopopolo/harness-engineering) — the harness as carrier of organizational nonfunctional requirements; reusable AGENTS.md/CLAUDE.md artifacts.

## Reference implementations worth studying

**To learn loop mechanics (smallest first)**
- [rasbt/mini-coding-agent](https://github.com/rasbt/mini-coding-agent) — the six core components in one readable stdlib-Python file.
- [huggingface/smolagents](https://github.com/huggingface/smolagents) — ~1,000-line harness; the code-agent pattern (model writes Python instead of JSON tool calls).
- [browser-use](https://github.com/browser-use/browser-use) — minimal viable harness for browser automation.
- [shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) — step-by-step deconstruction of Claude Code.
- [Learn Harness Engineering](https://walkinglabs.github.io/learn-harness-engineering/en/) — project-based curriculum.

**Production-grade architectures**
- [OpenHands](https://github.com/OpenHands/OpenHands) — Runtime/Sandbox + EventStream + Controller three-layer design.
- [Aider](https://github.com/Aider-AI/aider) — architect/coder split; git as the undo mechanism.
- [OpenCode](https://github.com/anomalyco/opencode) — provider-agnostic terminal harness; build/plan agent split.
- [Open SWE](https://blog.langchain.com/open-swe-an-open-source-framework-for-internal-coding-agents/) — curated ~15-tool limit, sandbox-per-task, AGENTS.md conventions.
- [Confucius Code Agent](https://github.com/facebookresearch/cca-swebench) — AX/UX/DX-structured harness; persistent cross-session notes.
- [Minions (Stripe)](https://stripe.dev/blog/minions-stripes-one-shot-end-to-end-coding-agents-part-2) — 1,300 PRs/week unattended; blueprints + centralized tool fleet.
- [Azure SRE Agent](https://techcommunity.microsoft.com/blog/appsonazureblog/how-we-build-azure-sre-agent-with-agentic-workflows/4508753) — 35k+ incidents handled; the most data-backed production case study.
- [Pi](https://github.com/earendil-works/pi) — sub-1k-token system prompt via lazy skills.
- [SmallCode](https://github.com/Doorman11991/smallcode) — harness compensations for small local models; proof that harness design is the binding constraint.
- [everything-claude-code](https://github.com/affaan-m/everything-claude-code) — battle-tested skills/hooks/rules/MCP configs across harnesses.
- [agents-best-practices](https://github.com/DenisSergeevitch/agents-best-practices) — provider-neutral skill for designing and auditing harnesses (closest sibling to this skill).

**Meta-harnesses & self-improving harnesses (the frontier)**
- [Meta-Harness](https://arxiv.org/abs/2603.28052) + [stanford-iris-lab/meta-harness](https://github.com/stanford-iris-lab/meta-harness) — the harness as a joint optimization target.
- [AutoAgent](https://github.com/kevinrgu/autoagent) — overnight harness iteration against a benchmark; beat hand-engineered entries.
- [retro-harness](https://github.com/wbopan/retro-harness) — self-supervised harness optimization from the agent's own trajectories (SWE-Bench Pro 59% → 78%).
- [Self-Harness](https://arxiv.org/abs/2606.09498) / [Live-SWE-agent](https://arxiv.org/html/2511.13646v3) — harnesses that mine their own failures and adapt.
- [harness-evolver](https://github.com/raphaelchristi/harness-evolver) / [neosigmaai/auto-harness](https://github.com/neosigmaai/auto-harness) / [autocontext](https://github.com/greyhaven-ai/autocontext) — practical meta-harness tooling.
- [VeRO](https://arxiv.org/abs/2602.22480) — evaluating agent-on-agent optimization itself.

**Industry state & production operations**
- [2026 Agentic Coding Trends Report](https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf?hsLang=en) — harness config as a first-class optimization variable.
- [State of Agent Engineering 2026](https://www.langchain.com/state-of-agent-engineering) — 57% in production; quality the top barrier; 89% observability vs. 52% evals.
- [Scaling Managed Agents](https://www.anthropic.com/engineering/managed-agents) — brain/hands/session separation.
- [Amazon Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) / [Vertex AI agent governance](https://cloud.google.com/blog/products/ai-machine-learning/new-enhanced-tool-governance-in-vertex-ai-agent-builder) / [TrueFoundry's harness framing](https://www.truefoundry.com/blog/agent-harness-managed-ai-agents) — what managed/enterprise harness platforms provide.
- [AI Agent Cost Optimization Guide 2026](https://moltbook-ai.com/posts/ai-agent-cost-optimization-2026) — routing, caching, and overhead patterns (40–80% savings).

## Adjacent curated lists

- [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — Claude Code-specific resources.
- [Awesome Context Engineering](https://github.com/Meirtz/Awesome-Context-Engineering) — the context side in depth.
- [awesome-mcp-servers](https://github.com/appcypher/awesome-mcp-servers) — MCP server catalog.
- [VoltAgent/awesome-ai-agent-papers](https://github.com/VoltAgent/awesome-ai-agent-papers) — weekly-updated research tracking.
- [RyanAlberts/best-of-Agent-Harnesses](https://github.com/RyanAlberts/best-of-Agent-Harnesses) — ranked, machine-readable harness comparisons.
