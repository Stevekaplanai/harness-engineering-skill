# Permissions, Sandboxing & Security

An agent that consumes untrusted content (web pages, emails, tool outputs, PR comments)
while holding credentials and network access is an attack surface, not just a product.
Security here is structural — enforced by the harness and environment — never prompt-level
trust.

## Design patterns

### Threat model first

- **Indirect prompt injection is the defining risk.** Agents actively ingest attacker-
  controllable content that can redirect their actions (Simon Willison's series; OWASP
  LLM01). Assume every external input is adversarial.
- **The lethal trifecta:** private data access + exposure to untrusted content + ability
  to communicate externally. Any tool *combination* that completes the trifecta is an
  exfiltration channel even if each tool is individually safe. Audit combinations, not
  tools.
- **Excessive agency** (OWASP LLM06): over-provisioned tools, unnecessary permissions,
  missing approval gates. The audit checklist is principle-of-least-privilege applied to
  the tool surface.
- **The weakest link is usually custom harness plumbing**, not the sandbox (Anthropic's
  containment write-up: exfiltration through an allowlisted domain). Model-layer defenses
  alone miss ~17% of overeager actions — environmental isolation must be the primary
  boundary.

### Permission architecture

- **Layered, deterministic evaluation.** The reference design (Claude Agent SDK): hooks →
  deny rules → permission mode → allow rules → runtime callback. Declarative
  allow/deny tool scoping plus a programmable gate for the gray zone.
- **Enforce on intent, not command names.** The same binary is benign or destructive
  depending on arguments; map calls to an intent taxonomy (`filesystem_delete`,
  `network_outbound`, `lang_exec`, …) rather than string-matching command allowlists (nah).
- **Pre-action authorization as a harness layer.** Intercept tool calls synchronously,
  evaluate against declarative policy, emit signed audit records (Open Agent Passport:
  restrictive policy = 0% attack success vs. 74.6% permissive, ~53ms overhead).
  ALLOW / DENY / REQUIRE_APPROVAL / MASK as decision outcomes (Microsoft's authorization
  fabric).
- **Two authorization models with different threat surfaces:** on-behalf-of (agent uses
  the end user's credentials — needs identity mapping and per-user isolation) vs.
  fixed-credential (agent has its own account — needs HITL gates on high-risk actions).
  Decide which you're building before placing enforcement.
- **Agents must not be able to edit their own harness config.** Protect MCP server
  configuration, hooks files, and permission settings from agent writes — an agent that
  can edit its own guardrails can escalate itself (NVIDIA red team guidance).

### Credential isolation

- **Agents should never possess secrets.** Inject credentials at a boundary the agent
  can't read: a credential broker/proxy that adds auth to outbound requests (Agent Vault,
  Cloudflare Dynamic Workers' interception, LangSmith Sandboxes' auth proxy), or managed
  OAuth (Nango, Composio). This converts prompt injection from "keys stolen" to "requests
  the proxy refused."
- Fine-grained, repo-scoped tokens over user-wide tokens, always.

### Sandboxing

- **Run the agent loop outside the sandbox; run execution inside.** Credentials stay out
  of the untrusted container; sandboxes become disposable cattle (Mendral's
  harness-outside argument; OpenAI's control-plane/compute-plane separation).
- **Options by isolation/latency trade-off:** microVMs (E2B, ~150ms; CubeSandbox <60ms;
  Firecracker), OCI containers with persistence (Daytona), V8 isolates (millisecond
  start, for generated-code execution), kernel-level policy (Landlock/seccomp/nsjail —
  Cursor's sandbox cut user interruptions 40%; NVIDIA OpenShell enforces at kernel +
  network proxy so a compromised agent can't override), K8s-native (agent-sandbox CRD,
  gVisor/Kata) when you must run in existing clusters. Fork-from-warm microVMs (forkd,
  zeroboot) make per-tool-call isolation feasible in the inner loop.
- **Sandboxing is necessary, not sufficient** (CNCF): you still need egress restriction,
  workspace-escape blocking, config protection, and output staging. GitHub's Agentic
  Workflows architecture (isolated container + firewall + MCP gateway + API proxy +
  staged outputs + zero-secret execution) is the defense-in-depth reference.
- **Tell the model about its sandbox.** Agents must be taught constraints exist or they
  thrash against them (Cursor's training insight).

### Defenses for untrusted content

- Sanitize/inspect tool results before they enter context (StackOne Defender: 22MB
  CPU model, ~4ms, 89% balanced accuracy); keep an eye on canary-token and
  input-validation patterns (tldrsec catalog is the checklist).
- Human approval gates on memory writes and other persistence (injection via stored
  memory outlives the session that planted it).
- Treat repo content, PR comments, CI logs, and fetched pages as untrusted input in
  coding agents specifically — they are the classic injection vectors.

## Curated resources

**Threat model**
- [Prompt Injection — Simon Willison's series](https://simonwillison.net/series/prompt-injection/) — why agents make injection uniquely dangerous.
- [OWASP LLM01 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/) / [OWASP LLM06 Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/) — the standard classifications.
- [The Attack and Defense Landscape of Agentic AI](https://arxiv.org/abs/2603.11088) — 128-paper survey; the 2026 threat-modeling reference.
- [How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude) — containment architectures; allowlist exfiltration case study.
- [Trustworthy agents in practice](https://www.anthropic.com/research/trustworthy-agents) — five governance principles.

**Permissions**
- [Beyond Permission Prompts](https://www.anthropic.com/engineering/beyond-permission-prompts) — structured authorization over prompt-level grants.
- [Claude Agent SDK — Configure Permissions](https://platform.claude.com/docs/en/agent-sdk/permissions) — the five-layer reference design.
- [Claude Code Auto Mode](https://www.anthropic.com/engineering/claude-code-auto-mode) — risk-tiered auto-approval.
- [Two Different Types of Agent Authorization](https://blog.langchain.com/two-different-types-of-agent-authorization/) — on-behalf-of vs. fixed-credential.
- [Open Agent Passport](https://arxiv.org/abs/2603.20953) — deterministic pre-action authorization with signed audit.
- [nah](https://github.com/manuelschipper/nah) — intent-taxonomy permission guard.
- [IETF draft-klrc-aiagent-auth](https://datatracker.ietf.org/doc/draft-klrc-aiagent-auth/) — cross-trust-domain agent identity (OAuth/WIMSE-based).

**Credentials**
- [Agent Vault](https://github.com/Infisical/agent-vault) — broker injects credentials; agent never holds them.
- [Nango](https://nango.dev) — managed OAuth for 700+ APIs.
- [Cloudflare Dynamic Workers](https://blog.cloudflare.com/dynamic-workers/) — isolate-level credential interception.

**Sandboxes**
- [The Agent Harness Belongs Outside the Sandbox](https://www.mendral.com/blog/agent-harness-belongs-outside-sandbox) — the architectural argument.
- [E2B](https://github.com/e2b-dev/E2B) / [Daytona](https://github.com/daytonaio/daytona) / [Alibaba OpenSandbox](https://github.com/alibaba/OpenSandbox) / [Kubernetes agent-sandbox](https://github.com/kubernetes-sigs/agent-sandbox) — the main open options.
- [NVIDIA OpenShell](https://github.com/NVIDIA/OpenShell) — kernel-level policy-driven sandbox runtime.
- [Cursor's agent sandboxing](https://cursor.com/blog/agent-sandboxing) — cross-platform OS sandboxing + the 40% interruption result.
- [Practical Security Guidance for Sandboxing Agentic Workflows](https://developer.nvidia.com/blog/practical-security-guidance-for-sandboxing-agentic-workflows-and-managing-execution-risk/) — egress, escape, config-protection controls.
- [Under the Hood: Security Architecture of GitHub Agentic Workflows](https://github.blog/ai-and-ml/generative-ai/under-the-hood-security-architecture-of-github-agentic-workflows/) — defense-in-depth reference.
- [Why sandboxing your agent is not enough](https://www.cncf.io/blog/2026/07/07/why-sandboxing-your-agent-is-not-enough/) — agent-substrate pattern.

**Injection defense & auditing**
- [tldrsec/prompt-injection-defenses](https://github.com/tldrsec/prompt-injection-defenses) — the defense catalog/checklist.
- [StackOne Defender](https://github.com/stackoneHQ/defender) — tool-result inspection at the boundary.
- [AI Harness Scorecard](https://github.com/anthropics/ai-harness-scorecard) — audit checklist for harness safeguards.
- [RAMPART](https://github.com/microsoft/RAMPART) — red-team findings as repeatable, statistical CI tests.
- [Microsoft Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit) — runtime policy enforcement across OWASP agentic risks.
