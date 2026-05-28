# OpenBrain — a persistent-memory backend for reliable AI agents

**A production memory system that gives autonomous AI agents a durable, searchable source of truth — engineered so they recall what they know before they act, and never silently lose it.**

## The problem

LLM agents are amnesiacs. Every session starts cold; anything learned last week is gone. The usual bolt-on fixes fail *quietly* — a vector store that misses the relevant memory, a context window that silently drops, an agent that confidently invents something because it never checked.

I spent two decades in digital forensics, where an unverifiable claim gets your evidence thrown out of court. I wanted agents held to that same standard: check what you know, attribute it, and don't drift. OpenBrain is the memory layer I built to make that real. It has run in production 24/7 for months as the shared long-term memory for a household of autonomous agents.

## What it is

A standalone memory service that any agent talks to over the **Model Context Protocol (MCP)** — `remember`, `recall`, `forget` exposed as tools. Drop it in and an agent gains durable, queryable memory with zero changes to its reasoning loop.

## Architecture & stack

- **Node.js** service exposing an **MCP** server (Streamable HTTP) plus a stateless **REST API** (Express).
- **SQLite** (WAL mode) as the system of record — a single, portable, 400MB+ knowledge base with no external database to operate.
- **Hybrid retrieval:** semantic vector search (`sqlite-vec`, 3072-dim embeddings) fused with **FTS5** full-text keyword search — recall catches both "means the same thing" and "exact term," instead of missing one or the other.
- **Temporal knowledge graph + contextual retrieval:** entries are linked and time-aware, so the system traverses *related* knowledge and resolves what was true *when* — not just nearest-neighbor blobs.
- **Domain-tagged, person-attributed:** every memory is routed to a domain and attributed to a person, so multiple agents share one brain without bleeding context into each other.
- **Async embedding worker with a circuit breaker:** embeddings are queued off the write path; three consecutive errors trips the breaker and stops cleanly with repair instructions, rather than corrupting the index.

## The hard part: memory you can trust

A memory system is only useful if it's *always there*. Two production reliability problems I had to solve:

1. **Silent failure.** Agent frameworks give up reconnecting after a couple of tries. I found a 45-hour agent session whose memory had been dead for 8 hours with *no error surfaced* — a backend restart had invalidated the session and the client quietly stopped retrying. I built a **resilience proxy** that holds one stable session indefinitely, heartbeats to dodge idle eviction, and transparently re-establishes the upstream on failure with exponential backoff and a circuit breaker. After it shipped, restarting the backend mid-session is invisible to the agent.
2. **Recovery without a human.** A three-layer autonomous watchdog (cross-platform: Linux/macOS/Windows) detects a stalled proxy within two minutes and restarts it; a pre-flight hook heals it sub-second when an agent wakes; and it only pages a human — debounced — when auto-recovery actually fails or thrashes.

Plus the unglamorous production hygiene: a fleet of thin clients over a private tailnet, split-secret HMAC bearer auth, and encrypted multi-recipient daily/weekly backups with a documented restore drill.

## What it demonstrates

I designed, built, and operate this end to end — schema, retrieval, transport, resilience, deployment, and the runbooks. It's the backbone the rest of my agentic systems run on. The throughline is the one from my forensics career: **build for the case where being wrong has consequences** — verify, attribute, recover, and never fail silently.

---

**Stack:** Node.js · Model Context Protocol (MCP) · SQLite · sqlite-vec (vector search) · FTS5 · vector embeddings · hybrid retrieval · temporal knowledge graph · Express / REST · systemd · Tailscale · encrypted backups
