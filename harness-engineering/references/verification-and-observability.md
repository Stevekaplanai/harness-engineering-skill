# Verification, Evals, Observability & Debugging

A harness without verification is a harness that lies to you: the agent declares success,
and nobody — including the agent — knows whether it's true. Verification closes the loop
in-flight; evals measure the harness across changes; observability makes failures
diagnosable; together they are the flywheel that makes harness iteration possible at all.

## Design patterns

### Verification inside the loop

- **The agent runs its own checks.** Tests, linters, builds, screenshots — the agent must
  be able to execute the verification command and read the result before marking work
  done. "Human review" is not a verification gate; it's a bottleneck.
- **Feedforward + feedback.** Böckeler's model: feedforward guides (conventions, specs,
  types) shape behavior before generation; feedback sensors (linters, tests, structural
  checks, LLM-as-judge) catch failures before a human sees them. Distinguish computational
  controls (deterministic, cheap, always-on) from inferential ones (LLM-judged, sampled).
- **Terse feedback.** Verbose test output is context pollution: a few summary lines into
  context, detail to a log file the agent can grep on failure.
- **Verification gates in artifacts.** Every PLAN.md milestone carries a `verify:`
  command; task completion is defined by gates passing (see bundled templates).
- **Architecture-level sensors.** Linters catch style and tests catch behavior; add
  structural/architectural checks (dependency rules, entropy/health scores) so agentic
  churn can't rot the codebase silently (sentrux; Fowler's "architectural constraints +
  entropy management").

### Eval design

- **Trajectory evals and outcome evals are different instruments.** Outcome: did the
  final state pass? Trajectory: was the path sound? Up to 23% of SWE-agent "passes" are
  lucky (regression cycles, blind retries) — process-quality scoring reorders model/harness
  rankings (AgentLens). Score both.
- **Capability evals ≠ regression evals.** Capability: low pass rate, the improvement
  target. Regression: near-100%, the protection target. Mixing them produces wrong
  priorities (LangChain's readiness checklist).
- **Binary pass/fail is wrong for stochastic systems.** Use pass^k (all k trials succeed)
  for reliability claims, statistical verdicts (PASS/FAIL/INCONCLUSIVE) for CI, and
  behavioral fingerprints for regression detection (AgentAssay: 86% of regressions caught
  vs. 0% with binary tests).
- **Layer cheap checks before expensive ones.** Deterministic trace checks (command
  sequences, token budgets, repo cleanliness) first; rubric/LLM-as-judge grading only
  where deterministic checks can't discriminate (OpenAI's skill-eval framework).
- **Control the environment.** Container resources alone swing scores 6+ points
  (Anthropic's infrastructure-noise study); web-enabled evals are vulnerable to the agent
  researching the benchmark itself (documented eval-awareness incidents) — isolate
  networks for evals. Report results as model+harness pairs, never model alone
  (Harness-Bench).
- **Evals are the training data for harness work.** The meta-harness result: given a
  benchmark and traces, harness optimization can be automated (AutoAgent, retro-harness:
  SWE-Bench Pro 59% → 78% self-supervised). Even manually, the loop is the same —
  propose, measure, keep or revert.

### Observability

- **Instrument from day one, on standards.** OpenTelemetry GenAI semantic conventions make
  traces portable; add spans to every inference and tool call (OpenLLMetry). Self-hostable
  UIs: Langfuse, Phoenix, Opik; eval-first platforms: Braintrust, Weave, LangSmith.
- **Traces are data, not just dashboards.** Land agent telemetry somewhere queryable
  (SQL over traces — Logfire, BigQuery Agent Analytics) so evaluators, regression
  detection, and debugging agents can be built on production behavior.
- **Correlate agent and infrastructure.** Failures often root in the environment (network,
  memory pressure, rate limits) — unified observability beats isolated LLM call logs.
- **Deep traces exceed human capacity.** Plan for AI-assisted trace analysis (trace-
  summarizing assistants, causal root-cause tools: AgentTrace at 69× faster than
  LLM-based analysis) rather than manual scanning of hundred-step runs.

### Debugging methodology

- **Evidence before fixes.** Reproduce, capture the trace, and require the diagnosis to
  cite specific evidence (stack traces, context state at the failure point) before
  proposing changes — "patch and pray" burns iterations (debug-skill's discipline).
- **Use a fault taxonomy.** Failures cluster: initialization, role deviation,
  memory/state deficiencies, orchestration failures, tool-integration errors (375-issue
  empirical taxonomy); memory/reflection/planning/action/system (AgentDebug). Classify
  first; each class has a known fix pattern. Most failures are mismatches between
  probabilistic outputs and deterministic interfaces — a structural problem.
- **One variable at a time.** Anthropic's April 2026 postmortem: three individually-minor
  harness changes (a default parameter, a caching bug, a prompt tweak) compounded into
  visible regression. Change, measure, then change again — and keep harness changes in
  code review with eval gates (VS Code's PR-gated harness changes).
- **Watch cost as a first-class metric.** Loop limits, tool-call caps, per-run token
  budgets, wall-clock timeouts, per-tenant budgets (FinOps guardrails); cost per accepted
  outcome, not tokens, is the real unit economic.

## Curated resources

**Verification & evals**
- [Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — trajectory vs. outcome; building reliable eval harnesses.
- [Harness engineering for coding agent users](https://martinfowler.com/articles/harness-engineering.html) — feedforward/feedback mental model.
- [Testing Agent Skills Systematically with Evals](https://developers.openai.com/blog/eval-skills) — layered deterministic-then-rubric methodology.
- [Agent Evaluation Readiness Checklist](https://blog.langchain.com/agent-evaluation-readiness-checklist/) — 33 items; capability vs. regression separation.
- [AgentAssay](https://arxiv.org/abs/2603.02601) — statistical regression testing for non-deterministic workflows.
- [AgentLens](https://arxiv.org/abs/2605.12925) — the lucky-pass problem; process-level scoring.
- [Harness-Bench](https://arxiv.org/abs/2605.27922) — report capability at the model+harness level.
- [Quantifying Infrastructure Noise](https://www.anthropic.com/engineering/infrastructure-noise) — resource config swings scores 6+ points.
- [Eval Awareness in BrowseComp](https://www.anthropic.com/engineering/eval-awareness-browsecomp) / [Designing AI-Resistant Technical Evaluations](https://www.anthropic.com/engineering/AI-resistant-technical-evaluations) — evals that survive capable models.
- [promptfoo](https://github.com/promptfoo/promptfoo) / [DeepEval](https://github.com/confident-ai/deepeval) / [Inspect AI](https://github.com/UKGovernmentBEIS/inspect_ai) — practical eval frameworks (CI-friendly; Inspect evaluates external agents black-box).
- [SWE-bench](https://www.swebench.com) / [tau-bench](https://github.com/sierra-research/tau-bench) / [StaminaBench](https://arxiv.org/abs/2606.19613) — outcome, policy-compliance, and long-horizon-stamina benchmarks.
- [Backtesting AI Agents](https://drdroid.io/blog/backtesting-ai-agents-how-sre-teams-prove-reliability-before-production) — pass^k, SLO thresholds, dataset composition.
- [sentrux](https://github.com/sentrux/sentrux) — architectural health sensor between linters and tests.

**Observability**
- [OTel GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) — the naming baseline.
- [OpenLLMetry](https://github.com/traceloop/openllmetry) — OTEL instrumentation for LLM/tool calls.
- [Langfuse](https://github.com/langfuse/langfuse) / [Arize Phoenix](https://github.com/Arize-ai/phoenix) / [Opik](https://github.com/comet-ml/opik) / [W&B Weave](https://github.com/wandb/weave) / [Helicone](https://github.com/Helicone/helicone) / [Pydantic Logfire](https://github.com/pydantic/logfire) — self-hostable tracing/eval stacks.
- [Braintrust](https://www.braintrust.dev) — eval-first tracing without sampling.
- [BigQuery Agent Analytics](https://cloud.google.com/blog/products/data-analytics/introducing-bigquery-agent-analytics/) — traces as queryable data.
- [Distributed Tracing for Agentic Workflows with OpenTelemetry](https://developers.redhat.com/articles/2026/04/06/distributed-tracing-agentic-workflows-opentelemetry) — context propagation across agents/MCP.

**Debugging**
- [An Update on Recent Claude Code Quality Reports](https://www.anthropic.com/engineering/april-23-postmortem) — how minor harness changes compound; diagnostic rigor.
- [Where LLM Agents Fail (AgentDebug)](https://arxiv.org/abs/2509.25370) — error taxonomy + corrective feedback (+24% accuracy).
- [Characterizing Faults in Agentic AI](https://arxiv.org/abs/2603.06847) — 37 fault types from 13,602 real issues; debugging checklist.
- [AgentTrace](https://arxiv.org/abs/2603.14688) — causal root-cause localization, 69× faster than LLM analysis.
- [AgentRx](https://www.microsoft.com/en-us/research/blog/systematic-debugging-for-ai-agents-introducing-the-agentrx-framework/) — constraint-based failure localization.
- [Debugging Deep Agents with LangSmith](https://blog.langchain.com/debugging-deep-agents-with-langsmith/) — AI-assisted trace analysis.
- [claude-devtools](https://github.com/matt1398/claude-devtools) — per-turn token attribution, subagent trees for Claude Code.
- [Syncause/debug-skill](https://github.com/Syncause/debug-skill) — evidence-cited debugging discipline.
- [AgentOps](https://github.com/AgentOps-AI/agentops) — session replay + cost tracking across frameworks.

**Cost & production ops**
- [FinOps for Agents](https://www.infoworld.com/article/4138748/finops-for-agents-loop-limits-tool-call-caps-and-the-new-unit-economics-of-agentic-saas.html) — the five budget guardrails; cost-per-accepted-outcome.
- [How My Agents Self-Heal in Production](https://blog.langchain.com/production-agents-self-heal/) — detect → attribute → auto-fix-PR loop.
- [The Coding Harness Behind GitHub Copilot in VS Code](https://code.visualstudio.com/blogs/2026/05/15/agent-harnesses-github-copilot-vscode) — harness changes as PR-gated, eval-checked code.
