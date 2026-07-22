# Tool Design, Skills & MCP

Tool design is agent UX. The tool surface — names, schemas, error messages, return
shapes, and above all *how many tools there are* — is one of the highest-leverage harness
variables, and one of the cheapest to fix.

## Design patterns

### Designing individual tools

- **One conceptual action per tool, unambiguous name, minimal schema.** Drop optional
  fields the agent won't use. Strict/validated schemas at the interface eliminate a whole
  class of parsing failures (structured outputs; `outlines` for local models; `instructor`
  for typed extraction with retry-on-validation-error).
- **Error messages are steering.** A tool error should tell the agent what to do next,
  not just what went wrong. Errors are one of the few places guidance is guaranteed to be
  in context at the moment of action (Stripe's finding: hard signals in loaded context
  change behavior; passive docs don't).
- **Consistent return shape on success and failure.** Agents build implicit expectations;
  shape changes break them silently.
- **Structured output over screenshots/raw dumps.** playwright-mcp's accessibility-tree
  snapshots (vs. screenshots) are the canonical example: same capability, far cheaper and
  more reliable.
- **Match the interface to the domain.** SWE-agent's Agent-Computer Interface (purpose-built
  file viewer/search/editor with explicit constraints and feedback) beats generic bash for
  its domain; conversely, generic bash + files beats 100 bespoke wrappers for exploratory
  ops work. Pick per task class, don't dogmatize.

### Sizing the tool surface

- **Fewer tools, better decisions.** Too many MCP servers bloat context and degrade tool
  selection. Open SWE enforces ~15 tools at harness design time; consolidating to ≤12
  skills improved accuracy in LangChain's skill evals. Audit for unused tools regularly.
- **Constrain tools by phase.** A state machine that only exposes phase-appropriate tools
  turned 2/10 into 10/10 runs for local models (statewright). Schema-level constraint
  beats runtime permission checks for enforcing workflow.
- **Code execution as a mega-tool.** Having the agent write code that calls services
  (instead of one context-visible tool call per operation) cut token overhead up to 98.7%
  (Anthropic's MCP code-execution pattern; Microsoft's CodeAct: −52% latency, −64% tokens).
  Reach for this when tool schemas + intermediate results dominate context.
- **Leverage vocabulary the model already has.** Mounting services as a virtual filesystem
  (`grep`/`cat`/`cp` over S3, Slack, GitHub — Mirage) or as CLIs beats teaching N novel
  APIs. Build on tools the model already knows (Anthropic's harness design patterns).

### Skills (progressive disclosure of expertise)

- A skill = a named, versioned instruction bundle loaded only when relevant: metadata
  always in context, body on trigger, resources on demand. This is the standard answer to
  "the agent ignores our conventions" without permanently spending context.
- **Descriptions drive routing.** OpenAI improved skill routing 73% → 85% by adding
  negative examples to descriptions; make descriptions state both when to use *and* when
  not to.
- **Skills are optimizable artifacts, not static prose.** Treat them like parameters:
  measure trigger accuracy and task outcomes, iterate (SkillOpt, AIP execution graphs —
  compiling prose skills to typed graphs moved pass rates 53% → 67%).
- Skills can hide multi-tool strategies: "when to use MCP, when to drop to an SDK, when to
  call raw API" behind one intent (Microsoft's Dataverse skills).

### MCP specifics

- MCP is the default protocol for tool/data connectivity; A2A covers agent-to-agent,
  AG-UI covers agent-to-frontend. Choose by boundary being crossed.
- **Production gaps to plan around** (enterprise field reports): identity propagation
  (who is this call for?), per-tool timeout contracts, structured error semantics, and
  session state vs. load balancers. The 2026 spec work moves toward a stateless core —
  check the current spec before building remote MCP infrastructure.
- **Annotations are hints, not enforcement.** `readOnlyHint`/`destructiveHint` etc. are
  inputs to your permission layer, not contracts. Analyze tool *combinations*: private
  data + untrusted content + external communication = the lethal trifecta (see
  `security-and-permissions.md`).
- Test MCP servers like software: MCP Inspector for interactive debugging,
  mcp-test-harness for CI regression tests.

## Curated resources

**Tool design**
- [Writing Effective Tools for Agents](https://www.anthropic.com/engineering/writing-effective-tools-for-agents) — the canonical naming/schema/error guide.
- [Tool Use — Claude API Docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) — client vs. server execution models.
- [Function Calling — OpenAI Docs](https://platform.openai.com/docs/guides/function-calling) — de facto JSON Schema conventions.
- [You can't whisper at an AI agent](https://stripe.dev/blog/ai-steering-experiments) — steering must be in loaded context.
- [Tool Annotations as Risk Vocabulary](https://blog.modelcontextprotocol.io/posts/2026-03-16-tool-annotations/) — annotations + the lethal trifecta.
- [outlines](https://github.com/dottxt-ai/outlines) / [instructor](https://python.useinstructor.com/) — structured output enforcement.
- [SWE-agent](https://github.com/SWE-agent/SWE-agent) — the ACI reference design.
- [statewright](https://github.com/statewright/statewright) — phase-constrained tool spaces.
- [Mirage](https://github.com/strukto-ai/mirage) — services as a virtual filesystem.
- [CLI-Anything](https://github.com/HKUDS/CLI-Anything) / [tui-use](https://github.com/onesuper/tui-use) — agent-native CLI/TUI surfaces for unwrapped software.

**Skills**
- [Shell + Skills + Compaction Tips](https://developers.openai.com/blog/skills-shell-tips) — versioned skill bundles, routing accuracy data.
- [Microsoft Skills Framework](https://github.com/microsoft/skills) — cross-platform skill definition/versioning.
- [SkillOpt](https://github.com/microsoft/SkillOpt) — skills as optimizable parameters.
- [superpowers](https://github.com/obra/superpowers) — cross-harness workflow skills (TDD, review gates) with an eval harness.
- [Evaluating Skills](https://blog.langchain.com/evaluating-skills/) — 82% vs. 9% completion with/without curated skills; ≤12-skill finding.
- [AIP: Graph Representation for Skills](https://arxiv.org/abs/2606.04781) — compiling skills to typed execution graphs.

**MCP & protocols**
- [Model Context Protocol](https://modelcontextprotocol.io/introduction) + [reference servers](https://github.com/modelcontextprotocol/servers) + [MCP Inspector](https://github.com/modelcontextprotocol/inspector).
- [Design Patterns for Deploying AI Agents with MCP](https://arxiv.org/abs/2603.13417) — the three production gaps and mitigations.
- [The 2026 MCP Roadmap](https://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/) + [2026-07-28 release candidate](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/) — stateless core, extensions, deprecation policy.
- [Developer's Guide to AI Agent Protocols](https://developers.googleblog.com/en/developers-guide-to-ai-agent-protocols/) — MCP vs. A2A vs. AG-UI vs. payment/commerce protocols.
- [A2A Protocol](https://github.com/a2aproject/A2A) / [AG-UI](https://github.com/ag-ui-protocol/ag-ui).
- [Composio](https://github.com/ComposioHQ/composio) — 250+ SaaS APIs as managed, authenticated actions.
- [mcp-test-harness](https://github.com/vaquarkhan/mcp-test-harness) — CI tests for MCP servers.
- [vurb.ts](https://github.com/vinkius-labs/vurb.ts) — authoring MCP servers that are safe/governable by default.
