# Context Delivery, Compaction & Memory

The context window is the agent's entire world. Most reliability problems trace back to
the wrong things being in it (or the right things missing/evicted). Treat context as a
finite, curated resource with an explicit budget.

## Design patterns

### Delivery: what goes in, and how

- **Navigation beats loading.** Give the agent pointers and search (symbol indexes, code
  search, `grep`-style tools) instead of injecting whole files. Symbol-indexed navigation
  cuts active tokens by ~77% (Token Savior) up to 120× (codebase-memory-mcp); semantic
  code search cuts ~98% vs. grep+read cycles (semble).
- **Filesystem as context interface.** Exposing everything (source, runbooks, notes) as
  files and letting the agent `read/grep/find` outperformed 100+ bespoke tools for
  Microsoft's SRE agent — "Intent Met" rose 45% → 75% on novel incidents. Durable state
  (plans, decisions, progress) belongs in files, not the prompt.
- **Progressive disclosure.** Always-in-context: a short constitution (conventions,
  constraints, safety rules). On-demand: skills, specs, domain docs loaded when relevant.
  Codified Context (283-session production study) splits a "hot-memory constitution" from
  a cold knowledge base of on-demand specs; Pi keeps the system prompt under 1k tokens via
  lazy skills (one-line descriptions until invoked).
- **Compress tool outputs before they enter context.** Bulky tool returns (logs, DOM
  snapshots, API responses) are the main pressure source; intercept and strip them
  (headroom: 60–95% reduction; context-mode sandboxes raw output and retrieves fragments
  via BM25). Verbose test output pollutes context — emit a few summary lines, log detail
  to file (Anthropic's parallel-Claudes C compiler lesson).
- **Retrieval as tool calls, not preprocessing.** Instead of stuffing retrieved documents
  in up front, expose search/read tools and let the agent pull incrementally (A-RAG).
- **Structure the prefix for caching.** Prompt caching is the single biggest cost lever
  (90% discount on cached tokens). Keep an immutable prefix (system prompt, tool defs),
  append-only history, and volatile scratch separated; DeepSeek-Reasonix hit 99.82%
  cache-hit rates and ~5× cost reduction by treating prefix stability as a loop invariant.
- **Serve agents clean inputs.** For web content, request/serve markdown (content
  negotiation, `Accept: text/markdown`) rather than scraping HTML.

### Compaction: surviving long sessions

- **Know what compaction destroys.** Auto-compaction preserves the current task, recent
  errors, file names; it loses initial instructions, style rules, and intermediate
  decisions. Critical rules go in CLAUDE.md / AGENTS.md (re-read, survives any
  compression) — never rely on conversation history for them.
- **Compact at task boundaries, not token thresholds.** Reactive at-limit compaction
  interrupts mid-subtask and corrupts in-flight reasoning. Prefer agent-controlled
  compression (a compaction tool the agent calls between tasks) or scheduled compaction
  at natural seams.
- **Compaction facts decay.** Cascaded summarization destroys ~60% of stored facts and
  drifts behavior ~54% (Facts as First Class Objects) — move durable facts to a structured
  store (files, KV, knowledge objects) instead of re-summarizing summaries.
- **Server-side compaction exists.** The Claude API's compaction endpoint reduced tokens
  84% in a 100-turn eval; Codex exposes `/responses/compact`. Use platform primitives
  before building your own.

### Memory: across sessions

- **Memory is a freshness problem more than a storage problem.** Stale, branch-specific
  memories are worse than none (GitHub Copilot's memory system verifies memories
  just-in-time against current code state before use). Build invalidation/eviction policy
  ("zombie memory" removal) into the design.
- **Gate memory writes.** Human-approved or validated writes block prompt-injection-via-
  memory and garbage accumulation (LangChain Agent Builder gates every write).
- **Tiered beats flat.** Three-tier (core/archival/recall — Letta/MemGPT; procedural/
  semantic/episodic — COALA) and hierarchical consolidation pipelines (episodes → facts →
  patterns) consistently beat flat vector stores on long-horizon tasks.
- **For coding agents, prefer local, file-backed memory primitives** (single binary,
  SQLite, MCP-served) over cloud memory platforms unless there's a real multi-surface
  requirement.

## Curated resources

Fetch these when you need the primary source:

**Context engineering**
- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic's systematic guide; the "what configuration of context produces the desired behavior" framing.
- [Compaction — Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/compaction) — server-side compaction reference.
- [Prompt Caching — Claude API Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) — where to place cache breakpoints.
- [Claude Code Compaction Explained](https://okhlopkov.com/claude-code-compaction-explained/) — what survives vs. what's lost; why critical rules go in CLAUDE.md.
- [Autonomous Context Compression](https://blog.langchain.com/autonomous-context-compression/) — agent-controlled compaction pattern.
- [Context Engineering Lessons from Azure SRE Agent](https://techcommunity.microsoft.com/blog/appsonazureblog/context-engineering-lessons-from-building-azure-sre-agent/4481200/) — filesystem-over-bespoke-tools case study with numbers.
- [Code Execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) — up to 98.7% token reduction by scripting tool interactions.
- [Making Agent-Friendly Pages with Content Negotiation](https://vercel.com/blog/making-agent-friendly-pages-with-content-negotiation) — `text/markdown` for agents.

**Context tooling (open source)**
- [context-mode](https://github.com/mksglu/context-mode) — sandbox bulky tool output outside the LLM, retrieve via BM25.
- [headroom](https://github.com/chopratejas/headroom) — compress tool outputs/logs/RAG chunks 60–95%.
- [Token Savior](https://github.com/Mibayy/token-savior) — symbol-index navigation, ~77% token cut.
- [codebase-memory-mcp](https://github.com/DeusData/codebase-memory-mcp) — tree-sitter knowledge graph, 120× token cut.
- [MinishLab/semble](https://github.com/MinishLab/semble) — NL code search replacing grep+read, ~98% cut, CPU-only.
- [Context7](https://github.com/upstash/context7) — up-to-date, version-specific library docs injection.
- [Trellis](https://github.com/mindfold-ai/Trellis) — progressive spec system replacing bloated CLAUDE.md.
- [OpenViking](https://github.com/volcengine/OpenViking) — ByteDance's filesystem-paradigm context database.
- [LLMLingua](https://github.com/microsoft/LLMLingua) — prompt compression (up to 20×) as a preprocessing layer.

**Memory**
- [Letta (MemGPT)](https://github.com/letta-ai/letta) — the reference three-tier stateful-agent architecture.
- [mem0](https://github.com/mem0ai/mem0) — lowest-integration-cost drop-in memory layer.
- [How We Built Agent Builder's Memory System](https://blog.langchain.com/how-we-built-agent-builders-memory-system/) — gated writes, memory-as-virtual-filesystem.
- [Building an Agentic Memory System for GitHub Copilot](https://github.blog/ai-and-ml/github-copilot/building-an-agentic-memory-system-for-github-copilot/) — freshness/invalidation as the core problem.
- [engram](https://github.com/Gentleman-Programming/engram) / [agentmemory](https://github.com/rohitg00/agentmemory) / [Stash](https://github.com/alash3al/stash) — local-first memory primitives for coding agents.
- [Facts as First Class Objects](https://arxiv.org/abs/2603.17781) — quantitative case for structured fact stores over in-context storage.
- [MemArchitect](https://arxiv.org/abs/2603.18330) — policy-driven memory governance (decay, conflict, privacy); the "zombie memory" problem.
- [Continual learning for AI agents](https://blog.langchain.com/continual-learning-for-ai-agents/) — model weights vs. harness behavior vs. contextual memory as distinct learning layers.
