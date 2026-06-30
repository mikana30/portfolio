# Mike Bauman — Project Portfolio

I design, build, and ship production **agentic-AI systems** end to end. Below are nine, spanning agent infrastructure, applied-AI services, autonomous research, security automation, and endpoint tooling.

The throughline comes from two decades across law enforcement, investigations, and digital forensics, where an unverifiable claim gets evidence thrown out of court: I build AI that **verifies before it speaks, attributes its sources, recovers from its own failures, and never fails silently.**

Most of these run in production today. Source is private (client and live systems), so each entry below is a one-page spec sheet — the public view of the work.

---

## Projects

| # | Project | Discipline |
|---|---------|-----------|
| 1 | [OpenBrain](#1-openbrain) | AI infrastructure / agent memory |
| 2 | [Adversarial Threat-Intel Verification Pipeline](#2-adversarial-threat-intel-verification-pipeline) | High-stakes applied AI / verification |
| 3 | [ForensicHunter](#3-forensichunter) | Digital forensics |
| 4 | [ARGUS](#4-argus) | Autonomous research / entity resolution |
| 5 | [InsightForge](#5-insightforge) | Applied-AI service (B2B sales intelligence) |
| 6 | [InterviewEdge](#6-interviewedge) | Applied-AI service (career / hiring equity) |
| 7 | [Quick Hit](#7-quick-hit) | OT/ICS security automation |
| 8 | [JobForge](#8-jobforge) | Multi-agent service (career) |
| 9 | [GRAM](#9-gram) | Endpoint AI tooling |

---

## 1. OpenBrain
*Persistent-memory backend that gives AI agents a durable, searchable source of truth.*

- **Discipline:** AI infrastructure / agent memory
- **Problem:** LLM agents start every session cold, and the usual memory fixes fail *silently* — a vector store that misses the memory, a context window that drops, an agent that invents because it never checked.
- **What it does:** Any agent gains durable memory over the **Model Context Protocol** (`remember`/`recall`/`forget`) with zero changes to its reasoning loop.
- **Built with:** Node.js · MCP server (Streamable HTTP) + Express REST · SQLite (WAL) · **hybrid retrieval** — `sqlite-vec` 3072-dim vector search fused with FTS5 keyword search · temporal knowledge graph + contextual retrieval · async embedding worker with a circuit breaker.
- **Scale / status:** 400 MB+ knowledge base, running 24/7 for months as shared memory for a fleet of agents across multiple machines; a resilience proxy plus a 3-layer cross-platform watchdog keep it self-healing.
- **Why it matters:** memory is the difference between a chatbot and an assistant — and it's only useful if it's *always there*. → **[Full case study](./openbrain-case-study.md)**

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
*Senior-grade B2B prospect dossiers in under 24 hours: org profile, decision-makers, tech and security posture, attack-surface findings, and talking points adapted to **what you're actually selling them**. $50 per dossier, auto-refunded if the data comes up thin.*

- **Discipline:** applied-AI service (B2B sales intelligence)
- **For:** B2B sales pros — account execs, MSSPs, brokers, staffing — who need a depth-of-prep brief in 24 hours instead of a week of manual digging.
- **What you get:** a prospect-specific dossier covering org structure and key decision-makers, current tech/security stack, passive attack-surface findings, regulatory and incident posture, and selling-angle talking points calibrated to your offering — every claim sourced, nothing invented.
- **Engage:** first report free, then $50 per dossier — with a full automatic refund if the data comes up thin. → **[insightforge.bluelinescannables.com](https://insightforge.bluelinescannables.com/)** · **[free sample report](https://insightforge.bluelinescannables.com/sample-insightforge-report.html)**
- **Built with:** Python web service · SQLite job queue · Stripe (payments + refunds) · 10+ structured searches per dossier, adapted at runtime to the selling context · BBOT-style passive recon · Node.js + `docx` renderer · **cold-session verification** before delivery · prompt-injection hardening (all user input treated as untrusted) · nginx / Let's Encrypt.
- **Scale / status:** Live service — served customers, admin console, affiliate program, refund flow, sample reports.
- **Why it matters:** the quality gate and the auto-refund are the economic guardrails that keep the AI honest — when the data isn't there, the system says so and the customer gets their money back. AI you can charge for and stand behind.

---

## 6. InterviewEdge
*A deep interview-prep service that triangulates **the company, the specific role, and your own resume** — so the prep dossier is YOU walking into THAT company for THAT job, not a generic company brief.*

- **Discipline:** applied-AI service (career / hiring equity)
- **For:** veterans transitioning out of service, college students entering the workforce, career-changers — anyone who deserves a fair shot at the answers a senior hire would already have walked in with.
- **What you get:** a bespoke per-interview dossier — the company's financials, org structure, leadership bios, comp benchmarks, culture-fit signals, regulatory landscape — **cross-referenced against your actual resume** so the prep, the angles, and the suggested talking points are specific to you, not boilerplate.
- **Engage:** first report free, then $25 per report — with a full automatic refund if the prep falls short. → **[interviewedge.bluelinescannables.com](https://interviewedge.bluelinescannables.com/)** · **[free sample dossier](https://interviewedge.bluelinescannables.com/sample-interviewedge-report.html)**
- **Mission:** Veterans transitioning out of service and college students entering the workforce often have the most to prove and the fewest resources for deep, role-specific prep. InterviewEdge gives them the dossier a senior candidate would spend a week building — in minutes.
- **Built with:** Python web service · SQLite · Stripe · Node.js dossier generator · resume / file-upload intake · per-candidate cross-referencing of resume against the target company and role.
- **Proof:** **3 of 3 candidates** in real-world trial runs converted to an offer or a follow-up interview.
- **Why it matters:** productizing a senior-grade research engine, candidate-specific, so the playing field gets a little flatter for the people who need it most.

---

## 7. Quick Hit
*A self-healing OT/ICS threat-advisory monitor that repairs its own broken scrapers at near-zero cost.*

- **Discipline:** OT/ICS security automation
- **Problem:** OT/ICS teams can't manually track dozens of vendor advisory pages whose layouts constantly change and break scrapers.
- **What it does:** Scrapes **58+** OT/ICS and IT advisory sources hourly, deduplicates findings (page- and item-level SHA-256), enriches with CVE data, and emails consolidated, severity-ranked HTML reports.
- **Built with:** Python (`ThreadPoolExecutor`) · `requests` / BeautifulSoup / `cloudscraper` + a real-browser CDP fetch for hard targets · NVD + CISA ICS APIs · a **tiered LLM self-healing chain** — when a parser breaks, extraction falls back through local-first inference to cloud as a last resort, with a hard daily cap on paid calls.
- **Scale / status:** 58+ sources, hourly cadence, in production.
- **Why it matters:** the cost-tiered self-healing is the clever part — broken scrapers get repaired by LLM fallback that prefers free/local inference before paid, so reliability climbs while run-cost stays near zero.

---

## 8. JobForge
*A weekly **weighted** job-search service: every verified-open match comes with **its own custom resume AND its own custom cover letter** — not one generic resume sprayed across twenty postings.*

- **Discipline:** multi-agent service (career)
- **For:** active job-seekers — especially career-changers, veterans translating military experience into civilian terms, and students with thin resumes — whose energy shouldn't be burned on spray-and-pray and whose applications shouldn't all read like the same template.
- **What you get each week:** a small set of *verified-open* roles, **weighted to your own priorities** (location, role type, comp floor, industry, remote vs hybrid, anything you tell it to care about), each one with its own ATS-tailored resume AND its own cover letter — bespoke per role, none of them generic, none fabricated, packaged and emailed.
- **Engage:** $40/month subscription. → **[jobforge.bluelinescannables.com](https://jobforge.bluelinescannables.com/)**
- **What it does (the pipeline):** three isolated phases per week — **(1)** weighted search and scoring against subscriber priorities; **(2)** independent verification that every posting is actually open, with backfill for dead ones; **(3)** per-job custom resume + custom cover letter generation, packaged and delivered.
- **Built with:** Python + a Node.js document generator · SQLite · each phase a fresh, isolated Claude session · bash orchestration / dispatch · large structured prompt specs per phase.
- **Why it matters:** verify-*before*-generate means tokens never get spent tailoring to dead postings; per-job bespoke document generation at scale means every application is the candidate's actual best foot forward; and a strict "never fabricate experience" floor protects the candidate from the most expensive mistake.

---

## 9. GRAM
*Geek Squad on a USB — with a portable Claude built in. Plug it into a broken Windows machine and an AI repair agent goes to work with full view of the running system.*

- **Discipline:** endpoint AI tooling
- **What it is, in one line:** Imagine a Geek Squad tech who can teleport to any broken PC, already has every diagnostic and repair tool ready, and brings an AI co-pilot that can actually *see* the machine — not a generic chatbot guessing from a copy-pasted error message. **GRAM is that, on a USB.**
- **What no one else offers:** Linux rescue media is blind to the running OS (it can read files, but it can't query event logs, services, the live registry, loaded drivers, network state, or active processes). Generic AI chatbots are blind to the *specific* machine. GRAM combines both — a portable AI repair agent with hands-on visibility into the live, broken Windows box.
- **What it does:** Plug in the USB; it launches a zero-dependency local web dashboard, runs PowerShell diagnostics, a security sweep, and 16+ repair actions — and an "AI mode" runs portable Claude Code with full live-system context to diagnose and fix in place.
- **Built with:** Node.js (zero-dependency local HTTP server, CORS-locked to localhost) · PowerShell (diagnostics / security / repair / drive-imaging / file-recovery) · portable Node + Claude Code CLI · integrity manifest for tamper detection.
- **For (commercial service):** repair shops, MSPs, IT consultancies — teams who need AI-augmented live-OS diagnostics across many machines at predictable, metered cost. → **[gram.bluelinescannables.com](https://gram.bluelinescannables.com/)** for pricing or a trial.
- **Scale / status:** Free for individuals; commercial service adds metered, multi-seat billing (credit packs and seat plans) for repair shops and MSPs.
- **Why it matters:** safe by construction — localhost-only networking, integrity-checked binaries, no cloud dependency once on-site. A field tech (or a shop running a fleet of them) gets an AI co-pilot that's actually *looking at* the machine, not guessing from a transcript.

---

*Each system was designed, built, and is operated end to end — architecture, implementation, deployment, and the runbooks.*
