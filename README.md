# Hammerin'Claude

**Intelligent multi-agent orchestrator for Claude Code — Constructor Method.**

Build software like you build a building: from the ground up, layer by layer, verifying each floor before building the next one.

---

## The Idea

When an AI agent needs to develop a complex feature — database, API, auth, frontend — it faces a critical choice: do everything inline or launch sub-agents in parallel. Both choices have a cost. Going inline on a 600-line task produces messy code. Launching 4 agents on a 20-line task wastes tokens and coordination time.

Hammerin'Claude solves this with a simple insight: **don't choose upfront**. Always start at the lightest level and scale only if the work demands it. Exactly like a construction site that starts with a small crew and calls for reinforcements only when the ground turns out to be harder than expected.

> Named after *Hammerin' Harry* (Irem, 1990) — a construction worker armed with a mallet who builds and fights. Hammerin'Claude does the same: builds software layer by layer, with the tenacity of a worker and the intelligence of an architect.

### v3 — Lean Rewrite

The skill was rewritten from 892 to **245 lines** (-73%), eliminating the dual format system (caveman/verbose) in favor of a single compact output format. This reduced token usage by ~19% while maintaining 100% benchmark pass rate. The checkpoint schema was extracted to a reference file loaded on demand, further reducing base context size.

---

## The Constructor Method

| Layer | Construction | Software |
|-------|-------------|----------|
| 1 — Foundation | Excavation + concrete | DB schema, types, interfaces, contracts |
| 2 — Structure | Load-bearing pillars | API routes, controllers, business logic |
| 3 — Walls | Enclosures + partitions | Frontend HTML/JS, UI components |
| 4 — Systems | Electrical + plumbing | Auth, logging, validation |
| 5 — Finishing | Plaster + paint | Empty states, loading, UX polish |

**Core principle: Opus designs, Sonnet builds.** Never reversed.

- **Opus** (claude-opus-4-6) acts as the architect — reads the codebase, decides strategy, defines contracts between layers with exact function names, endpoints, and DB columns.
- **Sonnet** (claude-sonnet-4-6) acts as the specialized worker — receives precise instructions and implements without deviation.

---

## Architecture

### Auto-trigger

The skill activates automatically across all sessions and projects when the user asks to develop a new feature, add a section, refactor, or any development task requiring planning. No manual invocation needed.

**Does NOT trigger for:** single-file bug fixes, questions, code explanations, pure CSS/UI changes, or deployments.

### Stack-agnostic

Works with any tech stack: React, Node.js, Laravel, Python, React Native, Docker. The layers and verifications are universal — only the specific content of each layer changes.

---

## The 5 Phases

### Phase 0 — Site Resume (centralized state markers)

Before any survey, checks for an interrupted construction site. Checkpoints are stored in a **centralized, protected folder** (`/home/webportal/.hammerin-state/`, chmod 700) — one JSON file per project (e.g., `sudo-support-it.json`). Each checkpoint tracks completed layers, modified files, real contracts between layers, mode (inline/crew), and the chosen budget. If found, the skill resumes exactly where it left off — no re-reading, no re-planning — even across different Claude Code sessions.

The checkpoint schema is defined in `references/checkpoint-schema.md` and loaded on demand only when writing a checkpoint, keeping the base context lean.

**Advantages of centralized state:**
- Survives `git clean`, repo reset, or re-cloning
- Doesn't pollute project root or `.gitignore`
- Single location to find all active construction sites
- Protected from unauthorized access (chmod 700)

### Phase 1 — Site Survey

Reads the minimum needed to understand the task weight:

1. **Project memory** — known context and conventions
2. **Entrypoint** — overall codebase structure
3. **1 pattern file** — the file most similar to what needs to be created
4. **(adaptive) 1-2 extra files** — only if the first pattern isn't enough

After the first pattern, a **confidence check** evaluates whether the pattern covers the task domain, conventions are clear, and volume estimation is reliable. If not, reads up to 2 additional files (max 3 total). If the codebase remains opaque after 3 files, flags it and starts with "inline + escalation" to reduce estimation risk.

**Budget Guardrails — Quotes.** At the end of the survey, the skill presents the user with **2–4 quotes** (Economic / Balanced / Complete), each with token ceilings, estimated euro cost, per-phase breakdown, and explicit trade-offs on quality, finishing, and verification depth. The skill does not touch any file until the user picks a quote. The chosen budget becomes the guardrail for the entire execution; at the checkpoint after layer 2, if a phase is at risk of exceeding its ceiling, the skill raises a **Budget alert** and asks before continuing.

### Phase 2 — Design & Foundation

Foundations are what everything rests on: DB schema, types, interfaces, contracts. If working inline, reads the files to modify and proceeds. If calling the crew, an Opus (Plan) agent designs all layers with exact contracts.

### Phase 3 — Construction

Proceeds layer by layer, bottom to top. Within a layer, independent agents work in parallel. Between layers, verification before ascending.

**Checkpoint after layer 2:** if remaining volume has grown, scale to the crew. If it has shrunk, cancel the crew and finish inline.

> **Note:** Benchmarks verify that plans correctly describe layer-by-layer construction with checkpoints. Actual multi-agent execution (launching real Sonnet sub-agents) has not been benchmarked yet.

### Phase 4 — Inspection

The skill instructs verification appropriate to the stack:
- **Structural** — server started, endpoints respond, build compiles
- **Systems** — auth, logging, validation
- **UI** — features visible, empty states, permissions
- **Regression** — existing features not broken

> **Note:** Verification commands (curl, npm run build) appear in plan outputs. Actual execution of these commands in a live benchmark has not been tested.

### Phase 5 — Delivery

Report to the user: completed layers, agents used, decisions made during construction, and precise instructions to verify in person.

---

## Dynamic Scaling

Instead of deciding upfront how many agents to use, the skill instructs the model to start at the lightest level and scale only if necessary.

| Mode | When | Example |
|------|------|---------|
| **Inline safe** | Pattern replication, few dozen lines | Add 1 field to DB + API + frontend |
| **Inline + escalation** | Uncertain volume, might grow | New section with partially new logic |
| **Direct crew** | Full feature, 3+ domains, new logic | Complete CRUD + auth + import/export |

**Verified:** Sizing S tasks correctly produce inline plans. Sizing M/L tasks correctly produce multi-agent plans with crew allocation. **Not yet verified:** actual mid-execution transitions (e.g., starting inline then scaling to crew after layer 2).

---

## Construction Site Visibility

The skill instructs the main thread to narrate the construction site so the user can follow progress.

**Verified in benchmarks (plan-only evals):**
- **Phase banners** — `━━━ FASE N NOME ━━━` consistently produced in all outputs
- **Budget quotes** — 2-3 option table with token/€ estimates before any work
- **Delivery report** — structured CONSEGNA block with files, lines, budget comparison
- **Verification markers** — `[OK] Strato N — detail` between layers

**Instructed but not yet verified in live execution:**
- **Task tracking** — the skill instructs `TaskCreate` per phase, but benchmarks were plan-only (no real tool calls). Likely works in real execution but unconfirmed.
- **Pre/post agent reports** — `→`/`←` format defined, but no benchmark has launched real sub-agents yet.
- **Inline progress updates** — instructed for layers taking >2 minutes, untested.
- **Scaling decisions** — `[SCALE]` format defined, no benchmark has triggered a real scale transition.

> Benchmarks to date test plan quality, not live execution. A live benchmark on a real M/L task would verify these features.

---

## Graceful Degradation

The skill instructs fallback behavior for degraded conditions. These fallbacks are defined in SKILL.md but **have not been tested in benchmarks** — no eval has simulated rate limits or model unavailability.

| Scenario | Instructed fallback |
|----------|---------------------|
| **Opus unavailable** | Plan inline in the main thread (already running on Opus) |
| **Sonnet rate-limited** | Reduce parallelism, wait 60s, retry |
| **Both limited** | Full inline mode — same layers, same verifications, main thread executes |

**Principle:** the Constructor Method is the layered structure with verification, not the presence of sub-agents. Sub-agents are a speed optimization, not a requirement.

---

## Benchmark (v3 Lean)

Tested on 2 eval tasks (Sizing S + Checkpoint persistence), comparing with_skill vs without_skill and v2 vs v3.

### Quality — Pass Rate

| Eval | With skill | Without skill | Delta |
|------|-----------|---------------|-------|
| Sizing S (6 assertions) | **100%** | 33% | **+67%** |
| Checkpoint (4 assertions) | **100%** | 0% | **+100%** |

### Efficiency — Token Usage

| Eval | v2 (892 lines) | v3 (245 lines) | Reduction |
|------|-----------------|-----------------|-----------|
| Sizing S | 51.9K | 45.7K | **-12%** |
| Checkpoint | 35.3K | 24.6K | **-30%** |
| **Mean** | **43.6K** | **35.1K** | **-19.4%** |

### Token Overhead (skill vs baseline)

| Version | With skill | Without skill | Overhead |
|---------|-----------|---------------|----------|
| v2 | 43.6K | 35.8K | **+22% (skill costs extra)** |
| v3 | 35.1K | 35.5K | **~0% (skill is free)** |

**Summary:** The v3 rewrite eliminated the token overhead entirely — the skill now costs virtually zero extra tokens compared to raw Claude, while delivering 100% pass rate on structured output, budget tracking, and cross-session checkpoints.

---

## Use Cases

- Full-stack features (DB + API + auth + frontend + UX in one coordinated flow)
- Large-scale refactoring across DB, API, and frontend
- Adding sections to existing portals (reads patterns, replicates, verifies)
- Stack migrations (rebuild layer by layer with verification)
- Ambiguous tasks (dynamic scaling discovers complexity during construction)
- Enterprise portals, SaaS platforms, mobile apps with dedicated backends

---

## Installation

The skill consists of:

```
~/.claude/skills/hammerin-claude/
├── SKILL.md                           # Core skill (245 lines)
├── references/
│   └── checkpoint-schema.md           # JSON schema for cross-session checkpoints
└── evals/
    └── evals.json                     # Benchmark eval definitions
```

Place it in your Claude Code skills directory. It will auto-trigger on all sessions and projects. No configuration needed.

---

## Output Format

A single compact format for all communication — no toggles, no modes.

| Element | Format |
|---------|--------|
| Phase banners | `━━━ FASE 1 SOPRALLUOGO ━━━` + 1 context line |
| Agent launch | `→ Sonnet "Backend": turni.js, database.js [bg]` |
| Agent result | `← "Backend" OK: turni.js +85, database.js +12` |
| Verification | `[OK] Layer 2 — 4/4 curl pass` |
| Summary | `[2/4] 3 files, ~120 lines. Next: Layer 3` |
| Decision | `[SCALE] Post-layer 2: escalation, 3 domains` |

**Never compressed:** contracts, sub-agent prompts, budget tables, verification commands, error messages, delivery reports.

---

## Rules

- **Quotes before building** — never touch files without user approval
- **Budget is a ceiling, not a target** — use only the tokens needed
- **Opus designs, Sonnet builds** — never reverse
- **Read before write** — always, at every layer
- **No file overlap** — never assign the same file to two agents in the same layer
- **Verify before ascending** — don't build layer N+1 without inspecting layer N
- **Nothing generic** — exact names, not "create an appropriate function"
- **Economic decision** — if agent cost exceeds work cost, go inline
- **Stack-agnostic** — works with any stack
- **Centralized checkpoints** — all state in `/home/webportal/.hammerin-state/`, never in project root

---

*Created by Marco Simone — April 2026*
