# OpenBrain — a persistent-memory backend for reliable AI agents

**A production memory system that gives autonomous AI agents a durable, searchable source of truth — engineered so they recall what they know before they act, and never silently lose it.**

## The problem

LLM agents are amnesiacs. Every session starts cold; anything learned last week is gone. The usual bolt-on fixes fail *quietly* — a vector store that misses the relevant memory, a context window that silently drops, an agent that confidently invents something because it never checked.

I spent two decades in digital forensics, where an unverifiable claim gets your evidence thrown out of court. I wanted agents held to that same standard: check what you know, attribute it, and don't drift. OpenBrain is the memory layer I built to make that real. It has run in production 24/7 for months as the shared long-term memory for a fleet of autonomous agents.

## What it is

A standalone memory service that any agent talks to over the **Model Context Protocol (MCP)** — `remember`, `recall`, `forget` exposed as tools. Drop it in and an agent gains durable, queryable memory with zero changes to its reasoning loop.

## Architecture & stack

- **Node.js** service exposing an **MCP** server (Streamable HTTP) plus a stateless **REST API** (Express).
- **SQLite** (WAL mode) as the system of record — a single, portable, 400 MB+ knowledge base with no external database to operate.
- **Hybrid retrieval:** semantic vector search (`sqlite-vec`, 3072-dim embeddings) fused with **FTS5** full-text keyword search — recall catches both "means the same thing" and "exact term," instead of missing one or the other.
- **Temporal knowledge graph + contextual retrieval:** entries are linked and time-aware, so the system traverses *related* knowledge and resolves what was true *when* — not just nearest-neighbor blobs.
- **Domain-tagged, person-attributed:** every memory is routed to a domain and attributed to a person, so multiple agents share one brain without bleeding context into each other.
- **Async embedding worker with a circuit breaker:** embeddings are queued off the write path; three consecutive errors trips the breaker and stops cleanly with repair instructions, rather than corrupting the index.

## What it unlocks

When agents can rely on memory the way humans rely on muscle memory, the whole system compounds:

- **Continuity across sessions.** An agent picks up where the last conversation left off, cites what's already been settled, and stops re-litigating decisions. The 50th conversation knows what the first 49 figured out.
- **Long-running autonomous workflows.** A research pipeline that runs for hours can checkpoint its findings, recover its place, and hand off to the next stage without losing the thread.
- **Multi-agent collaboration without context bleed.** Several agents share one memory store, but every entry is domain-tagged and person-attributed — so a finance agent and a security agent can both read from the same brain without confusing whose facts are whose.
- **Memory that gets better over time.** A weekly curation pass consolidates signal, prunes noise, and resolves conflicts — so the store sharpens with use instead of bloating.
- **Cited, attributed answers.** Every recall returns source attribution. Agents stop hallucinating because they're trained to check first and answer with provenance.

The engineering underneath is built to deserve that trust. A **resilience proxy** holds a single stable upstream session that survives backend restarts and idle eviction — heartbeats, exponential backoff, and a circuit breaker mean a downstream restart is invisible to the agent above. A **three-layer cross-platform self-healing watchdog** keeps the proxy itself healthy: a periodic check restarts a stalled proxy within two minutes, a pre-flight hook heals it sub-second when an agent wakes, and a human only gets paged — debounced — when auto-recovery actually fails. Plus the unglamorous production hygiene: a fleet of thin clients over a private tailnet, split-secret HMAC bearer auth, and encrypted multi-recipient daily/weekly backups with a documented restore drill.

Memory you can build on, because it's memory that's always there.

## What it demonstrates

I designed, built, and operate this end to end — schema, retrieval, transport, resilience, deployment, and the runbooks. It's the backbone the rest of my agentic systems run on. The throughline is the one from my forensics career: **build for the case where being wrong has consequences** — verify, attribute, recover, and never fail silently.

---

**Stack:** Node.js · Model Context Protocol (MCP) · SQLite · sqlite-vec (vector search) · FTS5 · vector embeddings · hybrid retrieval · temporal knowledge graph · Express / REST · systemd · Tailscale · encrypted backups
