# OpenBrain — a persistent-memory backend for reliable AI agents

**A production memory system that gives autonomous AI agents a durable, searchable source of truth — engineered so they recall what they know before they act, and never silently lose it.**

## The problem

LLM agents are amnesiacs. Every session starts cold; anything learned last week is gone. The usual bolt-on fixes fail *quietly* — a vector store that misses the relevant memory, a context window that silently drops, an agent that confidently invents something because it never checked.

I spent two decades in digital forensics, where an unverifiable claim gets evidence thrown out of court. I wanted agents held to that same standard: check what you know, attribute it, and don't drift. OpenBrain is the memory layer I built to make that real. It runs in production 24/7 as the shared long-term memory for a fleet of agents across multiple machines.

## What it is

A standalone memory service that any agent talks to over the **Model Context Protocol (MCP)** — `remember`, `recall`, `forget` exposed as tools. Drop it in and an agent gains durable, queryable memory with zero changes to its reasoning loop.

## Architecture & stack

- **Node.js** service exposing an **MCP** server (Streamable HTTP) plus a stateless **REST API** (Express).
- **SQLite** (WAL mode) as the system of record — a single, portable, 400 MB+ knowledge base with no external database to operate.
- **Hybrid retrieval:** semantic vector search (`sqlite-vec`, 3072-dim embeddings) fused with **FTS5** full-text keyword search.
- **Temporal knowledge graph + contextual retrieval:** entries are linked and time-aware — the system traverses *related* knowledge and resolves what was true *when*.
- **Domain-tagged, person-attributed:** every memory is routed to one of ten domains and attributed to a person, so multiple agents share one brain without bleeding context.
- **Async embedding worker with a circuit breaker:** embeddings are queued off the write path; three consecutive errors trips the breaker and stops cleanly with repair instructions, rather than corrupting the index.

## What it unlocks

Memory is the difference between a chatbot and an assistant. Without it, every conversation starts from zero — I'm re-explaining context, the model is re-deriving last week's decisions, and "knowing what's going on" lives only in my head. With OpenBrain in the loop, the agent walks in already knowing what we settled, what's open, and who said what. It stops being a smart parser and starts being a colleague who's been there all along.

The most concrete way I feel it: **I can pick up an issue from any machine, anywhere in the world.** I have agents running on my desktop, my laptops, and my phone — every one of them queries the same OpenBrain. I can be mid-debug on one box, step away, open a session from a completely different machine ninety minutes later, and the agent there already knows what I was chasing, what I tried, and what didn't work. Software-engineering work that used to require me to *re-establish state* now just continues across boxes. The fleet is one mind with many bodies.

Other things it unlocks, all from the same property:

- **Long-running autonomous workflows.** A research pipeline that runs for hours can checkpoint its findings, recover its place, and hand off to the next stage without losing the thread.
- **Multi-agent collaboration without context bleed.** Several agents share one store, but every entry is domain-tagged and person-attributed — a finance agent and a security agent can both read from the same brain without confusing whose facts are whose.
- **Cited, attributed recall.** Every result carries its timestamp, source agent, and tags. The agent answers with provenance — and stops hallucinating because it's trained to check first.
- **Memory that improves with use.** A weekly curation pass consolidates signal, resolves conflicts, and prunes noise — so the store sharpens with use instead of bloating.

## How recall actually works

When an agent calls a query like:

```
recall({
  query: "the parser regression from last week",
  domain?: "dfir" | "general" | ...,   // one of 10 domains
  agent_scope?: "all",                 // include cross-agent entries
  limit?: 10
})
```

OpenBrain does three things and fuses the result:

1. **Semantic search** via `sqlite-vec` — embeds the query with the same model used at write time, computes cosine similarity against the 3072-dim entry vectors, returns top-K.
2. **FTS5 keyword search** — catches exact-term matches that semantic similarity misses ("CVE-2024-12345" or "TLS handshake" won't always vector-cluster cleanly).
3. **Temporal-graph expansion** — pulls related entries the graph links to, so the answer carries context, not isolated blobs.

Scores from (1) and (2) are fused — balancing exact-term precision against semantic recall — and (3) widens the lens around the top hits. Every returned entry carries `{ text, score, domain, person, timestamp, source }` so the agent can show its work and the next session can re-trace the chain of reasoning.

That fusion is the *why* behind "no silent knowledge gaps." Semantic alone misses literal strings; keyword alone misses paraphrases. One or the other quietly failing has historically been how memory systems disappoint — the user assumes the relevant entry was in there, the system returned nothing, and nobody noticed until much later. Running both together, and fusing, removes that failure mode by construction.

## Always-there: the resilience underneath

Memory is only useful if it's there *every* time. The engineering that makes "always there" actually true:

- A **resilience proxy** holds one stable upstream session indefinitely — heartbeats every minute to dodge idle eviction; exponential backoff and a circuit breaker on upstream errors. A backend restart is now invisible to the agent above.
- A **three-layer cross-platform self-healing watchdog** keeps the proxy itself healthy: a periodic check restarts a stalled proxy within two minutes, a pre-flight hook heals it sub-second when an agent wakes, and a human only gets paged — debounced — when auto-recovery actually fails.
- Unglamorous production hygiene: a fleet of thin clients over a private tailnet; **split-secret HMAC bearer auth** (one half in the install repo, the other off-machine — either alone is useless); **age-encrypted multi-recipient backups** with a documented restore drill.

## What it demonstrates

I designed, built, and operate this end to end — schema, retrieval, transport, resilience, deployment, and the runbooks. It's the backbone everything else I build runs on. The throughline is the one from my forensics career: **build for the case where being wrong has consequences** — verify, attribute, recover, and never fail silently.

---

**Stack:** Node.js · Model Context Protocol (MCP) · SQLite · sqlite-vec (vector search) · FTS5 · vector embeddings · hybrid retrieval · temporal knowledge graph · Express / REST · systemd · Tailscale · age encryption
