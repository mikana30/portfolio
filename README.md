# Mike Bauman — Project Portfolio

I design, build, and ship production **agentic-AI systems** — largely solo, with an AI-centric build method. Below are ten of them, spanning agent infrastructure, applied-AI products, autonomous research, security automation, mobile, and endpoint tooling.

The throughline comes from two decades in digital forensics, where an unverifiable claim gets evidence thrown out of court: I build AI that **verifies before it speaks, attributes its sources, recovers from its own failures, and never fails silently.**

Most of these run in production today. Source is private (client and live systems), so each entry below is a one-page spec sheet — the public view of the work.

---

## Projects

| # | Project | Discipline |
|---|---------|-----------|
| 1 | [OpenBrain](#1-openbrain) | AI infrastructure / agent memory |
| 2 | [Adversarial Threat-Intel Verification Pipeline](#2-adversarial-threat-intel-verification-pipeline) | High-stakes applied AI / verification |
| 3 | [ForensicHunter](#3-forensichunter) | Digital forensics |
| 4 | [ARGUS](#4-argus) | Autonomous research / entity resolution |
| 5 | [InsightForge](#5-insightforge) | Applied-AI product (B2B SaaS) |
| 6 | [InterviewEdge](#6-interviewedge) | Applied-AI product (career) |
| 7 | [JobForge](#7-jobforge) | Multi-agent pipeline (career) |
| 8 | [Quick Hit](#8-quick-hit) | OT/ICS security automation |
| 9 | [GRAM](#9-gram) | Endpoint AI tooling |
| 10 | [cc-launcher](#10-cc-launcher) | Mobile AI |

---

## 1. OpenBrain
*Persistent-memory backend that gives AI agents a durable, searchable source of truth.*

- **Discipline:** AI infrastructure / agent memory
- **Problem:** LLM agents start every session cold, and the usual memory fixes fail *silently* — a vector store that misses the memory, a context window that drops, an agent that invents because it never checked.
- **What it does:** Any agent gains durable memory over the **Model Context Protocol** (`remember`/`recall`/`forget`) with zero changes to its reasoning loop.
- **Built with:** Node.js · MCP server (Streamable HTTP) + Express REST · SQLite (WAL) · **hybrid retrieval** — `sqlite-vec` 3072-dim vector search fused with FTS5 keyword search · temporal knowledge graph + contextual retrieval · async embedding worker with a circuit breaker.
- **Scale / status:** 400 MB+ knowledge base, running 24/7 for months as shared memory for a fleet of agents; a resilience proxy plus a 3-layer cross-platform watchdog keep it self-healing.
- **Why it matters:** memory is only useful if it's *always there* — built to verify, attribute, recover, and never fail silently. → **[Full case study](./openbrain-case-study.md)**

---

## 2. Adversarial Threat-Intel Verification Pipeline
*A daily threat-intelligence pipeline where a separate AI model cold-verifies every claim before it ships.*

- **Discipline:** high-stakes applied AI / verification
- **Problem:** An energy-sector security client needs daily, SIEM-deployable detections during an active regional cyber-conflict — with zero tolerance for fabricated or conflated intelligence reaching the client.
- **What it does:** A three-stage, isolated-session pipeline: (1) research → structured manifest; (2) generate SIEM detection queries; (3) a **cold, separate model verifies every source, IOC, and query** against a 12-step ruleset, then renders a branded report. Delta-only against a maintained IOC baseline.
- **Built with:** multi-model orchestration (one model researches/generates, a separate one verifies) · isolated-context sessions so the verifier can't rubber-stamp · Node.js deterministic document generator with preflight checks · SIEM query generation across five dialects (SPL, KQL, Splunk CIM `tstats`, and forwarded variants) with 18+ operational quality checks · scheduled daily runs.
- **Scale / status:** In production, near-daily for months; ~89 reports shipped. Every safeguard traces to a specific past failure (e.g., a claim-vs-source actor/causation check added after a report once inverted an attribution).
- **Why it matters:** this is "reliable AI" as an *architecture*, not a slogan — empty findings hard-reject, conflation auto-rejects, and a verifier that arrives cold catches what a single-pass agent would confidently ship.

---

## 3. ForensicHunter
*An air-gapped DFIR analysis workstation: open a disk image, parse 50+ artifact types, build a timeline, map to MITRE ATT&CK.*

- **Discipline:** digital forensics
- **Problem:** DFIR analysts need broad artifact coverage *and* AI assistance — without ever leaking client evidence to the cloud.
- **What it does:** Opens forensic disk images (E01/EWF/RAW/VMDK) without mounting them; runs **50 artifact parsers** (EVTX, MFT, registry, prefetch, SRUM, shellbags, NTDS, memory/pagefile, PCAP, Linux logs, and more); correlates everything into a unified timeline; and auto-maps findings to **MITRE ATT&CK**. Cloud AI is used for guidance only — through an allowlist sanitizer with a tamper-evident audit log proving no evidence leaves the box.
- **Built with:** C# / .NET 8 · WPF · CommunityToolkit.Mvvm + DI · DiscUtils / EWF image handling · Eric Zimmerman registry parser, `evtx`, TraceEvent · SQLite · Sigma / LOLBAS detection · DPAPI + Windows Credential Manager.
- **Scale / status:** 50 parsers covering the bulk of common SANS-poster artifacts; an extensive automated test suite; built for real casework.
- **Why it matters:** the allowlist sanitizer + tamper-evident audit log is the reliability-and-chain-of-custody instinct from two decades of court-admissible evidence handling, expressed as software.

---

## 4. ARGUS
*An autonomous OSINT research engine with entity resolution, confidence scoring, and recursive pivot-following.*

- **Discipline:** autonomous research / entity resolution
- **Problem:** OSINT across many sources produces duplicate, unverified, low-trust entities with no principled way to merge or prioritize them.
- **What it does:** Maintains a canonical entity store with dedup / merge / alias resolution and human-in-the-loop review; a disambiguator that requires **two corroborating fields before auto-merging**; and a confidence engine (source trust + corroboration) that drives recursive pivot decisions. Sources are pluggable, with per-type "frames."
- **Built with:** Python · SQLite · `networkx` · NumPy / matplotlib · `python-docx` · source connectors (county assessor, CourtListener, FCC ULS, OpenCorporates, SEC EDGAR, WHOIS/DNS) · a transport layer with caching, retry, circuit breaker, health checks, and selective Tor (SOCKS5) routing.
- **Scale / status:** Substantial multi-module codebase with a broad test suite; topic-agnostic across people, organizations, domains, and infrastructure.
- **Why it matters:** the two-field merge gate plus a confidence-driven pivot planner is real entity-resolution engineering — the difference between "scraped a pile of links" and "resolved a verified picture."

---

## 5. InsightForge
*A monetized B2B sales-intelligence SaaS that generates verified prospect dossiers — and refunds itself when the data is thin.*

- **Discipline:** applied-AI product (B2B SaaS)
- **Problem:** Sales professionals walk into meetings without deep, tailored intel on the prospect — and shouldn't have to pay for thin results.
- **What it does:** A customer submits a prospect plus selling context; an LLM research worker runs a structured multi-search playbook (10+ searches, adapted to context), passes a quality gate, emits a JSON manifest, gets **cold-session verified**, then a deterministic renderer produces a branded report. Paid, with an automatic refund when the data is insufficient.
- **Built with:** Python web service · SQLite job queue · Stripe (payments + refunds) · web/search APIs · Node.js + `docx` renderer · prompt-injection hardening (all user input treated as untrusted) · nginx / Let's Encrypt deploy.
- **Scale / status:** Live paid product ($50/report) with an admin console, affiliate system, sample reports, and refund handling.
- **Why it matters:** clean separation of nondeterministic research from deterministic rendering, plus economic guardrails (quality gate + auto-refund) — AI you can charge money for and stand behind.

---

## 6. InterviewEdge
*A paid service that produces research-backed interview-prep dossiers on a target company and role.*

- **Discipline:** applied-AI product (career)
- **Problem:** Candidates give generic interview answers because they lack deep, current intel on the company, the role, and its priorities.
- **What it does:** A buyer submits a target company/role (and their resume); an LLM research worker assembles company financials, org structure, leadership bios, compensation benchmarks, culture signals, and the regulatory landscape into sourced talking points; a deterministic generator renders a dossier and emails it.
- **Built with:** Python web service · SQLite · Stripe · Node.js dossier generator · resume / file-upload intake · the same research → verify → render pipeline as the sales product.
- **Scale / status:** Live paid product with a full marketing site and sample reports.
- **Why it matters:** the manifest → deterministic-render pattern carried into a new market — productizing a research engine into customer-facing software, with file intake and payment.

---

## 7. JobForge
*A multi-agent pipeline that finds, independently verifies, and applies to matched jobs with ATS-tailored documents.*

- **Discipline:** multi-agent pipeline (career)
- **Problem:** Job seekers burn effort on dead postings and generic applications.
- **What it does:** A weekly three-phase pipeline per subscriber: **(1)** research/filter/score matching roles; **(2)** independently re-verify that every posting is actually open, backfilling dead ones; **(3)** generate ATS-tailored resumes and cover letters *only* for verified roles, then package and email them.
- **Built with:** Python + a Node.js document generator · SQLite · each phase a fresh, isolated Claude session · bash orchestration/dispatch · large structured prompt specs per phase.
- **Scale / status:** In production, delivering real weekly application packages.
- **Why it matters:** verify-*before*-generate ordering (don't spend tokens tailoring to dead jobs), per-phase context isolation, and a hard "never fabricate experience" floor — multi-agent orchestration with correctness gates built in.

---

## 8. Quick Hit
*A self-healing OT/ICS threat-advisory monitor that repairs its own broken scrapers at near-zero cost.*

- **Discipline:** OT/ICS security automation
- **Problem:** OT/ICS teams can't manually track dozens of vendor advisory pages whose layouts constantly change and break scrapers.
- **What it does:** Scrapes **58+** OT/ICS and IT advisory sources hourly, deduplicates findings (page- and item-level SHA-256), enriches with CVE data, and emails consolidated, severity-ranked HTML reports.
- **Built with:** Python (`ThreadPoolExecutor`) · `requests` / BeautifulSoup / `cloudscraper` + a real-browser CDP fetch for hard targets · NVD + CISA ICS APIs · a **tiered LLM self-healing chain** — when a parser breaks, extraction falls back through local-first inference to cloud as a last resort, with a hard daily cap on paid calls.
- **Scale / status:** 58+ sources, hourly cadence, in production.
- **Why it matters:** the cost-tiered self-healing is the clever part — broken scrapers get repaired by LLM fallback that prefers free/local inference before paid, so reliability climbs while run-cost stays near zero.

---

## 9. GRAM
*A portable USB toolkit that runs AI-driven diagnostics and repairs on a live, broken Windows machine.*

- **Discipline:** endpoint AI tooling
- **Problem:** Offline rescue media only sees a dead machine's filesystem; technicians need visibility into the *running* OS — event logs, services, registry, drivers, network — and the ability to actually fix it.
- **What it does:** Plug the USB into a running Windows machine and it launches a zero-dependency local web dashboard, runs PowerShell diagnostics, a security sweep, and 16+ repair actions — plus an "AI mode" that runs portable Claude Code with full live-system context to diagnose and fix.
- **Built with:** Node.js (zero-dependency local HTTP server, CORS-locked to localhost) · PowerShell (diagnostics / security / repair / drive-imaging / file-recovery) · portable Node + Claude Code CLI · an integrity manifest for tamper detection.
- **Scale / status:** Working toolkit; a commercial tier adds metered, multi-seat billing for repair shops and MSPs.
- **Why it matters:** packaging an autonomous AI agent to operate *safely* on a live, broken machine — localhost-only networking, integrity checks, and live-OS repair that rescue environments simply can't do.

---

## 10. cc-launcher
*A server-driven AI Android launcher: the home screen is synthesized by an LLM and updated without shipping a new app.*

- **Discipline:** mobile AI
- **Problem:** A standard home screen is static. This one is AI-driven and server-updatable, with live insight cards — and changes ship without a new APK every time.
- **What it does:** A thin Kotlin/Compose launcher renders its home screen from a server-synthesized JSON manifest (**server-driven UI**, 8 widget types). A backend bridge synthesizes that manifest by calling an LLM with pre-fetched memory context — generating greeting/insight cards, resolving pins/recents/calendar, and applying live user feedback — all without an app rebuild.
- **Built with:** Kotlin + Jetpack Compose · OkHttp · coroutines · Material3 · `NotificationListenerService` · Python aiohttp bridge · LLM tool-calling via inline sentinels (bounded calls per turn) · private-network transport.
- **Scale / status:** Running on a personal Pixel; has an evaluation harness and tests.
- **Why it matters:** server-driven UI plus a sentinel protocol that lets the model safely call tools (memory read/write, artifact write) within hard bounds, and live feedback reclassification with no rebuild — sophisticated mobile-AI plumbing.

---

*Each system was designed, built, and is operated end to end — architecture, implementation, deployment, and the runbooks.*
