# How I Claude

> Patterns, guardrails, and hard-won lessons from building with Claude Code every day.

A battle-tested methodology for building reliable software with [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Developed over months of daily use across multiple active projects.

This isn't a tutorial. It's a system of patterns, conventions, and guardrails that solve real problems: context loss during long builds, security gaps, deployment failures, and the natural tendency of AI assistants to rationalize their own mistakes.

**Take what's useful, ignore what isn't.** Each pattern stands on its own.

> **Quick start:** Losing context mid-build? Start with [Context Survival](#context-survival). Shipping bugs? Start with [The Generator-Evaluator Pattern](#the-generator-evaluator-pattern). Want defense in depth? Start with [The 10-Stage Security Lifecycle](#the-10-stage-security-lifecycle).

### Patterns at a Glance

| Pattern | Problem It Solves | Section |
|---------|-------------------|---------|
| **The Planning Pipeline** | Scope creep, building before understanding | [Link](#the-planning-pipeline) |
| **File-Based Progress Tracking** | Losing place after context compaction | [Link](#context-survival) |
| **The Stack Frame Pattern** | Nested sub-plans causing parent plan amnesia | [Link](#nested-sub-plans-the-stack-frame-pattern) |
| **Generator-Evaluator** | AI confidently approving its own broken code | [Link](#the-generator-evaluator-pattern) |
| **Anti-Rationalization Rules** | Evaluators talking themselves into a pass | [Link](#anti-rationalization-rules) |
| **Fix Discipline** | Applying fixes without understanding blast radius | [Link](#fix-discipline) |
| **Correction Capture** | Same mistake repeated across sessions | [Link](#correction-capture) |
| **Multi-Perspective Council** | Single-viewpoint bias on business decisions | [Link](#the-multi-perspective-council) |
| **10-Stage Security Lifecycle** | Forgetting security checks at any stage | [Link](#the-10-stage-security-lifecycle) |
| **Path-Scoped Rules** | Forgetting domain-specific security context | [Link](#path-scoped-security-rules) |
| **Three-Tier Permissions** | Too permissive or too restrictive tool access | [Link](#the-permission-model) |
| **Red Lines** | Safety rules lost on context compaction | [Link](#red-lines) |
| **Model Routing** | Overspending on cheap tasks, underpowering hard ones | [Link](#model-routing) |
| **Skills & Plugins Ecosystem** | Reinventing workflows every session | [Link](#the-skills--plugins-ecosystem) |
| **MCP Integration Layer** | Manual context-switching between external tools | [Link](#mcp-integration-layer) |
| **Do It Yourself** | AI delegating work back to the developer | [Link](#the-do-it-yourself-principle) |
| **Design Discovery** | Starting UI work without visual direction | [Link](#design-discovery) |
| **Learning Promotion Pipeline** | Lessons learned once, forgotten everywhere else | [Link](#the-learning-promotion-pipeline) |
| **Staleness Check** | Knowledge files drifting from reality over time | [Link](#periodic-staleness-check) |
| **Config Drift Guard** | Settings changes going uncommitted and diverging | [Link](#config-drift-guard) |
| **Assumption Verification** | AI silently applying training data as fact | [Link](#assumption-verification) |
| **DNS-First Deployment** | NXDOMAIN caching breaking new subdomains | [Link](#dns-first-deployment) |
| **Continuation Prompts** | "Where were we?" lag between sessions | [Link](#continuation-prompts) |

---

## Table of Contents

- [Philosophy](#philosophy)
- [The Planning Pipeline](#the-planning-pipeline)
- [Context Survival](#context-survival)
- [The Generator-Evaluator Pattern](#the-generator-evaluator-pattern)
- [Fix Discipline](#fix-discipline)
- [Feedback Loops](#feedback-loops)
- [The 10-Stage Security Lifecycle](#the-10-stage-security-lifecycle)
- [Code Quality Gates](#code-quality-gates)
- [Assumption Verification](#assumption-verification)
- [Deployment Patterns](#deployment-patterns)
- [Knowledge Management](#knowledge-management)
- [The Permission Model](#the-permission-model)
- [Model Routing](#model-routing)
- [The Skills & Plugins Ecosystem](#the-skills--plugins-ecosystem)
- [MCP Integration Layer](#mcp-integration-layer)
- [Implementation Reference](#implementation-reference)
- [Starter Template](#starter-template)
- [Security Note](#security-note)

---

## Philosophy

Five design threads run through everything:

```mermaid
mindmap
  root((How I Claude))
    Silent Automation
      Hooks fire on save, commit, session start
      Developer never remembers to run checks
      Invisible until something fails
    Skepticism
      Separate generation from evaluation
      Ban cop-out language
      Independent agents evaluate work
    File-Based State
      Plans, progress, contracts on disk
      Survives context compaction
      Agents read files, not chat history
    Defense in Depth
      5 automatic + 5 manual security stages
      No single check catches everything
      Layer cheap fast checks
    Explicit Decisions
      One recommendation, not a menu
      Permission prompts explain risk
      Continuation prompts orient sessions
```

1. **Silent automation** — Hooks, rules, and agents run invisibly. The developer shouldn't have to remember to run security scans or format code. If you have to remember to do it, you'll forget.

2. **Skepticism & anti-rationalization** — AI models confidently approve their own mediocre work. Separate generation from evaluation. Tune evaluators to be skeptical. Ban cop-out language ("minor issue", "edge case", "works for common case").

3. **File-based state survives compaction** — Claude Code compacts conversation context when it gets long. Anything stored only in conversation memory gets lost. Plans, progress, contracts, and safety rules must live in files.

4. **Defense in depth** — No single check catches everything. Layer automatic checks (session start, file save, pre-commit) with manual reviews (code review, deploy check, quarterly audit). Five fast, cheap checks beat one slow, expensive one.

5. **Explicit decision-making** — One recommendation, not a menu. Permission prompts explain what/why/scope/risk. Continuation prompts orient new sessions. No hidden state changes.

---

## The Planning Pipeline

### The Problem

Jumping straight to code leads to scope creep, rework, and features that don't match requirements. AI assistants are especially prone to this — they'll happily start coding before understanding the full picture.

### The Pipeline

Every non-trivial feature follows this sequence. Plan Exit Review is the mandatory gate — it catches design-level issues that code review can't, because the code hasn't been written yet.

```mermaid
flowchart LR
    WB["Working Backwards<br/><i>external deps only</i>"]:::optional --> BR
    BR[Brainstorming] --> WP[Writing Plans]
    WP --> PER[Plan Exit Review]
    PER -->|"mandatory gate"| EP[Executing Plans]
    EP --> EV["/evaluate"]

    classDef optional fill:#f5f5f5,stroke:#999,stroke-dasharray: 5 5
```

| Stage | Purpose | Output |
|-------|---------|--------|
| **Working Backwards** | Trace from the user's feature moment backward through dependencies | Dependency map, external blockers |
| **Brainstorming** | Explore intent, requirements, design options | Design notes, tradeoffs |
| **Writing Plans** | Create bite-sized implementation plan | `docs/plans/YYYY-MM-DD-feature.md` |
| **Plan Exit Review** | Validate plan before any code is written | `docs/contracts/` with acceptance criteria |
| **Executing Plans** | Implement in batches of 3 tasks, pause for review | Working code + verification output |

**Working Backwards** is optional — use it when the feature involves external dependencies (partner APIs, store submissions, payment processors, regulatory approvals). For internal-only work, start at Brainstorming.

**Plan Exit Review is mandatory.** This is the gate that catches scope creep, missing conventions, untested paths, and deployment risks. The only exception: the developer explicitly says "skip review" or "just execute it."

### Entry Point Detection

When a feature request comes in, detect which entry point fits:

- **External dependencies** (partner APIs, payment processors, store submissions, regulatory approvals, multi-team coordination) → start with Working Backwards
- **Internal-only** (single codebase, no external approvals) → start with Brainstorming
- **Unclear** → ask: "This touches [X external system]. Should we do a Working Backwards analysis first?"

### Plan Format

Plans use bite-sized tasks (2-5 minutes each) with exact file paths, complete code, and verification commands:

```markdown
# Feature Name — Implementation Plan

**Goal:** One sentence
**Architecture:** 2-3 sentences about approach
**Tech Stack:** Key technologies

---

- [ ] Task 1: Component Name
- [ ] Task 2: API Routes
- [ ] Task 3: Tests
...

### Task 1: Component Name

**Files:**
- Create: `src/components/Widget.tsx`
- Modify: `src/pages/Dashboard.tsx:45-60`
- Test: `tests/components/Widget.test.tsx`

**Step 1:** Write the failing test
**Step 2:** Run test to verify it fails
**Step 3:** Write minimal implementation
**Step 4:** Run test to verify it passes
**Step 5:** Commit
```

The checkboxes (`- [ ]`, `- [~]`, `- [x]`) are critical — they enable [Context Survival](#context-survival).

### Batch Execution with Checkpoints

Don't execute the entire plan in one go. Execute in batches of 3 tasks, then pause:

1. Show what was implemented
2. Show verification output
3. Wait for feedback before continuing

This prevents runaway execution where the AI builds 10 tasks on a wrong foundation.

### Plans Are Living Documents

When new information changes the plan mid-execution, update the plan directly — rewrite remaining tasks to reflect reality. Don't preserve outdated tasks. The plan should always represent the current path forward.

Add a changelog entry at the bottom explaining what changed and why:

```markdown
## Changelog
- 2026-03-28 14:30: Tasks 5-6 rewritten — JWT library doesn't support RS256,
  switching to ed25519. Original tasks in git history.
```

Git history captures the exact diff. The changelog captures the *why*.

### Task Output Logs

While executing tasks, append timestamped work notes directly in the plan file:

```markdown
### Task 3: Authentication

**Output log:**
- 14:30 — Started. Created `src/auth/middleware.ts`, failing test written.
- 14:35 — Test passes. Discovered JWT library needs v4+ for RS256. Installing.
- 14:42 — JWT upgrade broke 2 existing tests. Fixing before proceeding.
```

This complements checkboxes (task-level status) with step-level detail. On resume after compaction, the output log tells you exactly where you stopped mid-task — not just which task, but which step within it.

---

## Context Survival

### The Problem

Claude Code compacts conversation context when it approaches the context window limit. After compaction, the AI loses track of where it was in a multi-step plan. This is especially bad when a plan step triggers a nested sub-plan — the AI dives into the sub-plan and forgets the parent plan entirely.

In-memory task tracking doesn't survive compaction. Conversation-only state is ephemeral.

### Solution: File-Based Progress Tracking

Progress is tracked directly in plan files using checkboxes:

```markdown
- [x] Task 1: Database schema          ← completed
- [x] Task 2: API routes               ← completed
- [~] Task 3: Authentication           ← in progress
- [ ] Task 4: Frontend components       ← pending
- [ ] Task 5: E2E tests                ← pending
```

**Rules:**
1. Before starting a task, mark it `- [~]` (in progress) in the plan file
2. After completing a task, mark it `- [x]` (done)
3. On resume (after compaction or new session): read the plan file, find the first unchecked task — that's where you pick up

### Nested Sub-Plans (The Stack Frame Pattern)

When a step spawns its own multi-step process (e.g., Task 3 needs a database migration that requires its own plan):

```mermaid
flowchart TD
    A["Executing parent plan<br/>Task 3 needs sub-plan"] --> B["Write docs/tasks/current.md<br/>(push stack frame)"]
    B --> C["Execute sub-plan steps"]
    C --> D{"Sub-plan complete?"}
    D -->|No| C
    D -->|Yes| E["Delete docs/tasks/current.md<br/>(pop stack frame)"]
    E --> F["Resume parent plan at Task 4"]

    G["On resume after compaction"] --> H{"docs/tasks/current.md<br/>exists?"}
    H -->|Yes| I["Finish sub-plan first"]
    H -->|No| J["Read plan file,<br/>find first unchecked task"]
    I --> E
```

The `docs/tasks/current.md` file acts as a stack frame:

```markdown
parent_plan: docs/plans/2026-03-28-feature-name.md
parent_task: 3
status: in_progress
reason: "Task 3 requires database migration with its own multi-step process"

## Sub-plan
- [x] Step 1: Create migration file
- [~] Step 2: Run migration on staging
- [ ] Step 3: Verify data integrity
```

On resume: if `docs/tasks/current.md` exists, finish the sub-plan first, then return to the parent. Delete the file when the sub-plan completes.

### File-Based Handoff (Multi-Phase Builds)

When an implementation plan has 3+ phases, write intermediate state to files so agents can be spun up fresh without conversation context:

| File | Purpose |
|------|---------|
| `docs/plans/` | Implementation plan (generated by writing-plans skill) |
| `docs/contracts/` | Acceptance criteria (generated by plan-exit-review) |
| `docs/build-state/<feature>/eval-round-<N>.md` | Evaluator reports per round |
| `docs/tasks/current.md` | Active nested sub-plan stack frame |

Agents read these files rather than relying on conversation context. Delete `docs/build-state/` after the feature is merged.

### What Survives Compaction

| Mechanism | Survives | Use For |
|-----------|----------|---------|
| CLAUDE.md | Always (reloaded every session) | Safety rules, conventions, red lines |
| Plan files with checkboxes | Always (on disk) | Execution progress |
| `docs/tasks/current.md` | Always (on disk) | Nested sub-plan stack |
| `docs/contracts/` | Always (on disk) | Acceptance criteria for evaluator |
| Session logs | Always (on disk) | Decision history across sessions |
| MEMORY.md | Always (reloaded every session) | Cross-session context |
| In-memory tasks | **No** | Only within single context window |
| Conversation history | **No** | Lost on compaction |

### PreCompact Hook

A hook that fires before context compaction can save a snapshot of what was in-flight — which files were changed, what branch you're on. This creates an audit trail, not a recovery mechanism.

The key lesson: **anything important must already be on disk before compaction triggers.** The PreCompact hook is a safety net, not a primary strategy. Write progress to plan files as you go.

---

## The Generator-Evaluator Pattern

### The Problem

Self-evaluation bias. When the same AI that wrote the code also evaluates it, it confidently approves its own work — even when it's broken. "It looks correct to me" is the most dangerous phrase in AI-assisted development.

### The Pattern

Separate code generation from code evaluation using independent agents:

```mermaid
flowchart TD
    DEV["Developer Agent<br/>(writes code)"] --> MC["Mechanical Checks<br/><i>build, lint, type-check, tests</i><br/>(parallel, fast)"]
    MC -->|fail| DEV
    MC -->|pass| EVAL["Independent Evaluator Agent<br/><i>tests live app vs. acceptance criteria</i><br/>(separate instance, skepticism-tuned)"]
    EVAL -->|fail| FIX["Developer fixes"]
    FIX --> EVAL
    EVAL -->|"fail 3x"| HUMAN["Flag to developer:<br/>needs human judgment"]
    EVAL -->|pass| CR["Code Review Agent<br/><i>security, quality, error handling</i><br/>(background, 80%+ confidence)"]
    CR --> DONE["Done"]
```

**The evaluator agent must be independent:**
- **Separate agent instance** — not the same conversation that wrote the code
- **Skepticism-tuned** with anti-rationalization rules (see below)
- **Tests the live application** — not just reads the code
- **Uses pre-agreed acceptance criteria** from the contract — not criteria it generates about its own code
- **Uses the most capable model** available — evaluation is where reasoning quality matters most

### Anti-Rationalization Rules

The evaluator agent has explicit rules to prevent talking itself into a pass:

- If a bug is found → **FAIL**. No downgrading to "minor issue."
- Binary output only: **PASS** or **FAIL**. No "partial pass."
- **Forbidden phrases:** "minor issue", "edge case", "works for common case", "non-blocking", "acceptable for now"
- Cannot adjust acceptance criteria to match actual behavior.
- No "PASS but..." language.
- Better to false-positive (developer verifies) than false-negative (ship broken).

### Auto-Fix Loop

When evaluation fails:
1. The evaluator reports what failed and why
2. The developer agent attempts to fix
3. Re-evaluate (max 3 rounds)
4. If still failing after 3 rounds → flag to the human developer

### When to Evaluate

Evaluate automatically after non-trivial changes:
- More than 10 lines changed, OR
- More than 1 file modified

**Skip for:** documentation-only changes, config edits, git operations, CLAUDE.md updates.

### Choosing Testing Tools

Match the tool to what you need to verify:

| Need | Tool | Why |
|------|------|-----|
| Authenticated user flows | Browser automation with real sessions | Tests what users actually see |
| Quick pass/fail checks | Headless browser (Playwright) | Fast, low token cost |
| Multi-step user journeys | E2E framework (Maestro/Cypress) | Declarative, reusable flows |
| Security headers (CSP, CORS, HSTS) | **Real browser, not curl** | These headers are browser-enforced |

---

## Fix Discipline

### The Problem

A fix without understanding its blast radius is a guess. AI assistants — and developers — tend to apply the first plausible fix without tracing callers, checking for related tests, or considering downstream effects. The result: fixes that work in isolation but break something else.

### The Impact Map (Before Every Fix)

Before writing a single line of fix code, map the blast radius:

```mermaid
flowchart TD
    BUG["Bug reported"] --> E["1. Identify epicenter<br/><i>file(s) and function(s)</i>"]
    E --> CALLERS["2. Trace callers<br/><i>grep imports/usages</i>"]
    E --> CALLEES["3. Trace callees<br/><i>read imports/deps</i>"]
    E --> GIT["4. Check recent changes<br/><i>git log -10 per file</i>"]
    E --> TESTS["5. Find related tests<br/><i>glob for test files</i>"]
    CALLERS --> FIX["Apply fix"]
    CALLEES --> FIX
    GIT --> FIX
    TESTS --> FIX
```

1. **Identify epicenter** — the file(s) and function(s) at the center of the bug
2. **Trace callers** — grep for imports/usages across the project
3. **Trace callees** — read imports to understand downstream dependencies
4. **Check recent changes** — `git log --oneline -10 -- <each affected file>`
5. **Find related tests** — glob for test files matching module names

### Verify After Fix

1. Run the failing test, then module tests, then caller tests from the impact map
2. No tests exist → flag explicitly: "No test coverage — regression risk"
3. Spawn a fix-reviewer agent in the background with: bug description, impact map, and git diff

### Escalation — Automatic

- **First fix fails** → invoke the debug pipeline immediately. No second attempt without structured debugging.
- **3 failed attempts within the debug pipeline** → STOP. Flag to the developer as an architectural problem.

This discipline applies to **bug fixes and unexpected behavior only** — not TDD red-green-refactor cycles where failures are expected.

---

## Feedback Loops

### Correction Capture

#### The Problem

The AI makes a mistake. You correct it. Next session, it makes the same mistake because corrections live only in conversation context — which gets compacted or ends.

#### The Pattern

When the developer corrects the AI's approach (explicit: "no, don't do X" / "always do Y" / "that's wrong"), the AI silently appends a one-liner to a corrections file:

```
- 2026-03-28 | project-name | what was corrected and the right approach
```

No announcement, no asking — just log it alongside the normal response. At session end, the save-and-move routine processes corrections into their proper destinations:
- **Specific to a project** → project's CLAUDE.md
- **Cross-project pattern** → global CLAUDE.md or path-scoped rule
- **Personal preference** → memory file

```mermaid
flowchart LR
    C["Correction happens<br/>in conversation"] --> L["Silently append to<br/>corrections.md"]
    L --> S["Session end:<br/>/save-and-move"]
    S --> R{"Route by scope"}
    R -->|project| P["Project CLAUDE.md"]
    R -->|cross-project| G["Global CLAUDE.md<br/>or rule file"]
    R -->|preference| M["Memory file"]
```

**Why this works:** Corrections are high-signal, low-volume. They represent exactly the behaviors the developer wants changed. By routing them to persistent config files, the correction survives across all future sessions — the AI literally can't make the same mistake twice.

**Record from failure AND success.** If you only save corrections, you'll avoid past mistakes but drift away from validated approaches. Watch for quiet confirmations too — "yes exactly", "perfect, keep doing that", accepting an unusual choice without pushback.

### The Multi-Perspective Council

#### The Problem

When you ask an AI a business question, you get one perspective. The AI optimizes for a single frame (usually the most conventional answer) and misses angles that a real advisory board would surface.

#### The Pattern

For high-stakes business or product decisions, spawn multiple independent advisor agents with distinct personas, then synthesize:

```mermaid
flowchart TD
    Q["Business question"] --> A1["Contrarian Advisor<br/><i>challenges assumptions,<br/>identifies hidden risks</i>"]
    Q --> A2["Creative Advisor<br/><i>unconventional approaches,<br/>opportunity cost analysis</i>"]
    Q --> A3["CFO Advisor<br/><i>financial implications,<br/>unit economics, ROI</i>"]
    A1 --> PR["Peer Review<br/><i>each advisor critiques<br/>the others anonymously</i>"]
    A2 --> PR
    A3 --> PR
    PR --> CH["Chairman Synthesis<br/><i>reconciles perspectives,<br/>delivers verdict + rationale</i>"]
```

**Key design choices:**
- **Advisors run in parallel** (independent, no groupthink)
- **Anonymous peer review** — each advisor sees the others' arguments but not who wrote them, preventing authority bias
- **Chairman uses the most capable model** — synthesis requires reconciling contradictory arguments
- **Structured output** — verdict, confidence, minority dissent, action items with owners and deadlines

**When to use:** Strategic decisions, pricing changes, market entry, product pivots, partnership evaluations — anything where "what am I missing?" matters more than "what's the obvious answer?"

---

## The 10-Stage Security Lifecycle

### The Problem

Security checks are easy to skip. Developers (human or AI) forget to scan for secrets, validate inputs, or check dependencies. A single missed check can leak credentials or introduce vulnerabilities.

### The Lifecycle

Five automatic stages fire without invocation. Five manual stages require explicit action. Together they create defense in depth — no single check catches everything, but the combination is comprehensive.

```mermaid
flowchart TB
    subgraph AUTO["Automatic (fire invisibly)"]
        direction LR
        S1["1. Session Start<br/><i>tracked .env scan</i>"]
        S2["2. File Write<br/><i>secret pattern scan</i>"]
        S3["3. Dependency Install<br/><i>typosquatting warning</i>"]
        S4["4. Pre-Commit<br/><i>staged diff scan</i>"]
        S5["5. Pre-Compaction<br/><i>state snapshot</i>"]
    end

    subgraph MANUAL["Manual (require explicit action)"]
        direction LR
        S6["6. Code Review<br/><i>OWASP, auth, input</i>"]
        S7["7. Sprint Planning<br/><i>security posture</i>"]
        S8["8. Pre-Deploy<br/><i>header + secret check</i>"]
        S9["9. Quarterly Audit<br/><i>comprehensive scan</i>"]
        S10["10. Post-Incident<br/><i>root cause + rules</i>"]
    end

    AUTO --> MANUAL
```

**Automatic stages:**

| # | Stage | When | What |
|---|-------|------|------|
| 1 | Session Start | Every session | Scan for `.env` files accidentally tracked by git |
| 2 | File Write | Every code edit | Scan for hardcoded secrets — known API key formats, credential patterns, tokens |
| 3 | Dependency Install | `npm install`, `pip install` | Warn to verify package name on registry (typosquatting defense) |
| 4 | Pre-Commit | `git commit` | Git hook scans staged diff for credential patterns |
| 5 | Pre-Compaction | Before context compaction | Snapshot git state to log file |

**Manual stages:**

| # | Stage | When | What |
|---|-------|------|------|
| 6 | Code Review | Before merge | Security-first review (OWASP top 10, auth, input validation) |
| 7 | Sprint Planning | Start of sprint | Include security posture check in sprint scope |
| 8 | Pre-Deploy | Before production | Enhanced credential scan + security header verification |
| 9 | Quarterly Audit | Every 3 months | Comprehensive audit: dependencies, configs, access controls, secrets |
| 10 | Post-Incident | After security event | Root cause analysis → update rules to prevent recurrence |

### Path-Scoped Security Rules

Security context auto-loads when editing sensitive files — no manual invocation needed. When the AI edits an authentication file, stricter rules about input validation, session handling, and CSRF protection activate automatically.

Organize rules by domain:
- **Infrastructure security** — files related to auth, sessions, tokens, config, middleware, routes, controllers
- **LLM/AI security** — files related to prompts, agents, RAG, embeddings, vector stores

The path matching is glob-based (e.g., `**/auth/**`, `**/payment/**`). Each rule file contains domain-specific reminders and requirements.

### Secret Scanning Approach

The scanning strategy uses two layers:

1. **Known format detection** — scan for well-known API key formats (provider-specific prefixes, known token patterns). Each major cloud/SaaS provider has a distinctive key format.

2. **Generic credential patterns** — scan for `key=value` patterns that commonly indicate hardcoded secrets, with false-positive filtering to exclude environment variable getters and placeholder values.

Both layers run on every file write (PostToolUse hook) and on every commit (pre-commit git hook). The defense-in-depth principle means catching a secret at either stage is a win.

### Pre-Commit Git Hook

A zero-dependency bash script installed in `.git/hooks/pre-commit` that scans staged changes for credential patterns. It should:

1. Read the staged diff (`git diff --cached`)
2. Check for known API key formats
3. Check for generic credential assignment patterns
4. Filter out false positives (env getters, placeholder values, test fixtures)
5. Block the commit if any match is found

> **Note:** Build your own scanning patterns rather than copying them from public repos — published regex patterns can be studied and evaded. The approach matters more than the specific implementation.

### False Positive Gates (for HIGH findings)

Before reporting any finding as HIGH severity during audits, run through 6 verification gates. All must be YES to confirm. If any is NO, downgrade or skip.

1. **Data flow proven?** — Can you trace untrusted input reaching the vulnerable sink through actual code paths?
2. **Attacker can trigger?** — Is the vulnerable code reachable from an external interface?
3. **Impact is real?** — Would exploitation cause actual harm (data leak, auth bypass, RCE)?
4. **PoC credible?** — Can you describe a concrete attack scenario, not just "this pattern is risky"?
5. **No other protections?** — Is there no upstream middleware, WAF, or framework protection?
6. **Exploitable in practice?** — Given the deployment context (internal tool vs. public API), is this realistic?

This prevents alert fatigue from theoretical findings.

---

## Code Quality Gates

### Pre-Commit Self-Checks

Before every commit, the AI silently scans its own changes for:

1. **Swallowed errors** — empty catch blocks, errors logged but not re-thrown, missing error handling on async calls
2. **Function complexity** — functions over 40 lines or cyclomatic complexity over 10 → split them
3. **Cop-out language** — scan for: "pre-existing issue", "out of scope", "follow-up needed", "beyond current task", "works for now". If found: fix the issue or explicitly flag it as a known trade-off with reasoning. Don't bury it.

### Progressive Code Review (No Manual Triggers)

Reviews run automatically at natural workflow moments. The developer never triggers these manually:

```mermaid
flowchart TD
    EV["/evaluate passes"] --> CR["Code Review Agent<br/><i>background, mid-tier model</i><br/>80%+ confidence only"]
    CR --> SC["Self-Checks<br/><i>parallel, before commit</i>"]

    subgraph SC["Pre-Commit Self-Checks (parallel)"]
        direction LR
        S1["Swallowed errors<br/><i>empty catch, missing async</i>"]
        S2["Anti-rationalization<br/><i>cop-out language scan</i>"]
        S3["CLAUDE.md sync<br/><i>code↔conventions match?</i>"]
    end

    SC --> COMMIT["Commit"]
```

1. **After `/evaluate` passes** — code review agent runs in background. Only surfaces 80%+ confidence issues. Fix what you can, flag what you can't.
2. **Before every commit** — parallel self-checks: swallowed errors, anti-rationalization gate, bidirectional CLAUDE.md sync.
3. **After major features** — suggest E2E test generation (don't auto-run).

### Code Standards Worth Enforcing

These prevent classes of bugs, not just style issues:

| Standard | Why |
|----------|-----|
| **Never use display text for program logic** | `btn.textContent` changes with i18n/theming. Use `data-*` attributes or state variables for conditionals. |
| **Always show the latest version of data** | When re-scanning/regenerating, clear stale results before writing new ones. Users expect current state, not accumulation. |
| **Delete from server, not just UI** | Archive/dismiss must hit the DB in the same request. Removing from a JS array only hides until refresh. |
| **AI features are optional enhancements** | If an LLM API call fails, the tool must still work as manual CRUD. Never block a workflow on an AI response. |
| **Functions under 40 lines** | Split if cyclomatic complexity exceeds 10. Smaller functions are easier to test and reason about. |
| **Persist valuable data immediately** | If it took an API call, user input, or >2 seconds to generate — persist to DB. Server memory is not persistence. |

### Bidirectional CLAUDE.md Sync

When code changes affect conventions, architecture, or project structure — update CLAUDE.md. Don't just follow CLAUDE.md; keep it current. It's a living document, not a stale reference.

---

## Assumption Verification

### The Problem

AI assistants silently apply training data as fact. When editing a file that integrates with an external API, the AI assumes the response shape, auth method, or error format based on "industry standard" patterns — not your actual system. Three categories of wrong assumptions:

1. **Library/API behavior** — assumed defaults that changed between versions
2. **Infrastructure state** — assumed environment variables, DNS records, or service configurations that don't match reality
3. **Business logic** — assumed user flows or domain rules that aren't documented in code

The cost isn't the wrong assumption itself — it's the rework cycle. The AI writes code confidently, tests pass locally, and the bug surfaces in production or integration testing. By then, understanding the assumption requires re-reading the code path.

### The Assumption Taxonomy

Classify every assumption the AI makes about systems outside the current codebase:

| Type | Source | Action |
|------|--------|--------|
| **Stated** | Developer explicitly said it | Proceed — ground truth |
| **Inferred** | Read from code/config/docs this session | Proceed but mention the source |
| **Assumed** | Training data / "industry standard" | **Must verify before implementing** |
| **Uncertain** | No evidence either way | **Must ask the developer** |

The dangerous type is **Assumed** — it feels confident because it matches training data, but training data is generic. Your project's Cloudflare configuration, API versioning, or database schema is not generic.

### Defense in Depth (4 Layers)

No single mechanism catches all silent assumptions. Layer them:

```mermaid
flowchart TB
    subgraph PLAN["Planning Time"]
        L1["Layer 1: Plan Exit Review<br/><i>Assumptions Register in every plan</i>"]
    end

    subgraph IMPL["Implementation Time"]
        L2["Layer 2: CLAUDE.md Self-Check Rule<br/><i>6 triggers to stop and verify</i>"]
        L3["Layer 3: PostToolUse Hook<br/><i>mechanical nudge on external-system files</i>"]
    end

    subgraph RETRO["Retrospective"]
        L4["Layer 4: Session-End Pattern Collection<br/><i>self-calibrating pattern list</i>"]
    end

    PLAN --> IMPL --> RETRO
    L4 -.->|"adds new patterns"| L3
```

**Layer 1 — Assumptions Register (planning gate).** During plan review, extract every assumption about external systems. Classify each using the taxonomy above. Any **Assumed** or **Uncertain** items that can't be verified during review become blockers on the tasks that depend on them.

```markdown
## Assumptions Register
| # | Assumption | Type | Verified? | Action |
|---|-----------|------|-----------|--------|
| 1 | CF Pages uses SHA-256 for asset hashing | Assumed | No | Must verify before Task 3 |
| 2 | Database schema has `users.email` column | Inferred | Yes | Read from schema.sql |
| 3 | Webhook payload includes `event_type` field | Uncertain | No | Ask developer |
```

**Layer 2 — Self-check triggers (CLAUDE.md rule).** Six conditions where the AI should stop and verify before proceeding:

1. Using knowledge not from this session (training data is not fact for your project)
2. First time touching this area of the codebase
3. Inferring behavior from naming conventions instead of reading code
4. External system behavior (APIs, libraries, services, infrastructure)
5. Business logic intent ("I can see *what* the code does, never *why*")
6. Own uncertainty signals ("should be", "probably", "typically")

This is judgment-based — it fades with context compaction. That's why it needs the mechanical layers below.

**Layer 3 — PostToolUse hook (mechanical nudge).** A hook that fires on every file edit, checking whether the filename matches patterns associated with external systems (e.g., `api`, `client`, `webhook`, `auth`, `config`, `deploy`). When it matches, it emits a short reminder:

> "This file likely interacts with an external system. Have you verified your assumptions about its behavior?"

This survives context compaction because it's a hook, not a conversation instruction. It's a nudge, not a gate — it reminds rather than blocks.

**Layer 4 — Self-calibrating pattern list.** At session end, scan the session log for rework signals ("fixed", "wrong", "actually", "turns out", "assumed"). For each rework event involving an external system, check whether the filename would have triggered the Layer 3 hook. If not, propose adding a new pattern.

Periodically review proposals: add confirmed patterns, remove stale ones that haven't triggered in 30+ days. The pattern list evolves based on real misses, not theoretical coverage.

### Scope

Only gate decisions that are **expensive to reverse** — cross-system integrations, external API calls, infrastructure state, business logic. Don't gate code-internal work, styling, refactoring within a module, or following an established pattern already verified this session.

The goal is preventing costly rework, not adding friction to every edit.

---

## Deployment Patterns

### DNS-First Deployment

When deploying to a new subdomain:

```mermaid
flowchart LR
    DNS["1. Create DNS<br/>A record"] --> WAIT["2. Wait for<br/>propagation"] --> DEPLOY["3. Deploy<br/>application"]

    style DNS fill:#d4edda
    style DEPLOY fill:#d4edda
```

**Why this order:** A deployed app with no DNS triggers NXDOMAIN negative caching (up to 1 hour — every resolver that looked up the name will cache "doesn't exist"). A dangling A record just gets a 404 from the reverse proxy — harmless and immediately fixable once the app is deployed.

### Health-Check-Gated Deploys

Every application must have a `/health` endpoint. Configure health checks before the first deploy:

```
Interval: 5s    Timeout: 5s    Retries: 3    Start period: 5s
```

The deploy fails if the health check doesn't pass. This catches crashes, missing environment variables, and broken dependencies before traffic reaches the app.

### Deploy, Don't Restart

After pushing code changes, always **rebuild from git** (deploy). Never restart — a restart reuses the old container image and silently ignores your code changes. This is one of those mistakes you make once, spend an hour debugging "why didn't my change take effect?", and then never make again.

### Post-Deploy Verification

After every deploy, verify the change is actually live. Curl a changed endpoint, check a new header, or load a modified page. Don't assume the deploy picked up the latest code.

### Persistent Storage Before First Deploy

Every application with a database needs persistent storage (volume mounts) configured **before the first deploy**. Without it, every deploy wipes the database. This is non-recoverable.

### Background Task Recovery

Long-running background tasks (analysis, processing, generation) are silently killed on container restart/redeploy. Always provide a **recovery endpoint** ("Re-analyze All", "Reprocess Queue") for any background work that produces important data. Without this, a mid-session deploy silently loses in-flight work.

### Container Permission Gotcha

When using host-mounted directories with non-root container users, the mount is owned by root. The app can't write to it. Fix: run the container as root, use an entrypoint script that `chown`s the data directory, then drops privileges to the app user with `exec su-exec <user> <command>`.

---

## Knowledge Management

### The Problem

Lessons learned in one project don't propagate to others. The same mistake gets made three times before someone writes it down. And in long sessions, the AI forgets what was decided earlier.

### Session Logs

Every work session gets a log entry with a specific structure:

```markdown
## 2026-03-28 14:30 — Feature Name

**Context:** What we were trying to do
**Decisions:** What we chose and why
**Rejected:** What we considered but didn't do (and why)
**Done:** What was actually completed
**Next:** What to do next session
```

Key insight: **log rejected approaches**, not just decisions. This prevents re-exploring dead ends in future sessions. "We tried X and it didn't work because Y" is more valuable than "we did Z."

### Continuation Prompts

At the end of each session, generate a continuation prompt stored in a persistent file (MEMORY.md):

```
"Last session: implemented X, Y, Z. Discovered blocker on W.
Next up: (1) fix W, (2) add tests for Y, (3) deploy."
```

Next session starts by reading this prompt and orienting automatically — no "where were we?" lag. A SessionStart hook can display this automatically.

### The Learning Promotion Pipeline

Not every lesson belongs in global config. A four-gate pipeline prevents premature promotion:

```mermaid
flowchart LR
    S["Session learning"] --> L["LEARNINGS.md<br/><i>project-level</i>"]
    L -->|"14-day maturity"| G{"Meets all 4<br/>criteria?"}
    G -->|Yes| C["Global CLAUDE.md"]
    G -->|No| L
    L -->|"90 days unreviewed"| F["Flagged for<br/>confirmation"]
```

**Four criteria for promotion:**
1. **Generality** — applies to 3+ projects
2. **Durability** — proven useful over 14+ days
3. **Actionable** — "always do X" or "never do Y"
4. **Not duplicate** — not already documented

Learnings not reviewed in 90+ days get flagged for confirmation (never auto-deleted — you might need them).

### Periodic Staleness Check

Regularly scan your knowledge files — topic memories, learnings, CLAUDE.md sections. For each, ask: is this still accurate?

Flag anything that looks stale: outdated tool versions, patterns superseded by new conventions, references to files that no longer exist. Update or remove. Build it into your session-end routine so it happens automatically, not as a separate task you'll forget.

A dedicated `/memory-audit` skill can automate this — scanning topic files for mentions of files, skills, MCPs, or hooks that no longer exist and reporting findings before making changes.

### Config Drift Guard

A SessionStart hook that counts uncommitted changes in your `~/.claude/` directory and warns if the number exceeds a threshold (e.g., 10 files). This catches configuration drift — skills, rules, hooks, and settings that were modified during work sessions but never committed.

Without this, your local config silently diverges from what's in version control. The `/save-and-move` session-end routine should include a mandatory check for uncommitted config changes.

### Data Persistence Tiers

When implementing state persistence, explicitly choose the right tier. Mixing them up causes data loss:

| Tier | Survives | Use For |
|------|----------|---------|
| JS variable / server memory | Nothing (lost on refresh/restart) | Transient UI state only |
| `sessionStorage` | Page refresh within same tab | Step position, scroll state |
| `localStorage` | Browser close, new tabs | User preferences (language, theme) |
| Database (SQLite/Postgres) | Everything | Anything users create or that took effort |

**Rule:** If it took an API call, user input, or >2 seconds to generate — persist to DB immediately. AI-generated artifacts (summaries, analysis) must be saved the moment they're produced. Server memory is not persistence.

---

## The Permission Model

### Three Tiers

| Tier | Behavior | Examples |
|------|----------|---------|
| **Auto-approved** | Runs without asking | Read, Edit, Write, `git status/log/diff/add/commit`, `npm test`, `ls`, `mkdir` |
| **Ask-first** | Shows permission prompt | `git push`, `npm install <pkg>`, `curl`, `ssh`, `kill`, `mv` |
| **Hard-blocked** | Cannot run even with approval | Destructive commands (`rm -rf`), privilege escalation (`sudo`), piped installs (`curl\|sh`), global installs, force pushes |

### Structured Permission Prompts

Before any ask-first command, show a structured explanation so the developer can decide in 5 seconds:

```
PERMISSION NEEDED
What:  [plain English — what this command does]
Why:   [why it's needed for the current task]
Scope: [what it touches — which server, repo, file, service]
Risk:  [what could go wrong — data loss, external visibility, irreversibility]
```

### Red Lines

Actions that should never happen without explicit permission, regardless of context:

- **Destructive:** Delete files/branches, overwrite config/env/database files, truncate/drop tables
- **Remote:** Push to remote, create/modify PRs/issues, send messages on external services, download from external internet
- **System:** Run sudo, install global packages, modify system files, kill processes without context
- **Communication:** Send emails, messages, or notifications on the developer's behalf

These rules must live in **persistent config** (CLAUDE.md), not conversation. Conversation-only rules get lost on context compaction — this was learned the hard way when an agent forgot a safety instruction after compaction and took a destructive action.

### The "Do It Yourself" Principle

The AI should never delegate to the developer what it can do with its own tools. Before telling the developer "please go run this manually" or "do this from the browser," check if there's a tool available. The only exceptions:

1. Tasks requiring a GUI you genuinely can't access **and you've confirmed no MCP tool covers it**
2. Actions blocked by a red line — in which case, say "I can do this if you approve" not "please go do this"

This sounds obvious, but AI assistants default to giving instructions rather than taking action. Explicitly codifying "do it yourself" in CLAUDE.md changes the behavior.

---

## Model Routing

### The Problem

Using the most expensive model for everything wastes money. Using the cheapest model for everything produces bad results on complex tasks.

### Start Cheap, Escalate by Signal

Default to a capable but cost-efficient model for daily work. Escalate to the most powerful model only when you detect specific signals:

| Signal | Why It Needs Escalation |
|--------|------------------------|
| 3+ interacting constraints | Too many variables to hold in one pass |
| 2+ failed approaches | Problem likely needs deeper reasoning |
| Cross-domain architecture (3+ modules across frontend + backend + infra) | Coordination complexity |
| Security audit with nuanced judgment | Stakes are too high for a cheap model |
| Contradictory requirements | Need to reconcile conflicting constraints |

**Don't escalate for:** simple code edits, file search, config changes, git operations, debugging with clear error messages, single-file changes.

### Subagent Model Selection

When delegating to subagents, match model to task type:

| Task Type | Model Tier | Examples |
|-----------|------------|---------|
| Reading + summarizing | Cheapest | File search, log analysis, status checks, context gathering |
| Judgment + opinions | Mid-tier | Code review, security analysis, verification |
| Critical evaluation | Best | Independent evaluator (anti-rationalization requires maximum reasoning) |
| Multi-perspective synthesis | Best | Council chairman (reconciling contradictory arguments) |

The evaluator is always the best model regardless of session model — that's where reasoning quality has the highest impact.

### Mechanical Enforcement

A hook can enforce model selection by blocking subagent calls that don't specify an explicit model parameter. This prevents accidental use of expensive models for trivial tasks and ensures the routing rules are followed consistently.

---

## The Skills & Plugins Ecosystem

### The Problem

Without reusable workflows, you reinvent processes every session. "How do I deploy again?" or "What's the review checklist?" gets answered differently each time.

### Skills: Reusable Workflow Recipes

Skills are markdown-defined workflows that get loaded into context when invoked. They're not scripts — they're structured instructions that the AI follows, adapting to the current project.

**Planning & Design:**
| Skill | Purpose |
|-------|---------|
| `/brainstorming` | Explore intent, requirements, design options before coding |
| `/writing-plans` | Generate structured implementation plans with checkboxes |
| `/plan-exit-review` | Validate plans before code (mandatory gate) |
| `/working-backwards` | Trace from user moment backward through dependencies |
| `/design-discovery` | Visual design exploration — generate mockups, pick direction, save theme tokens |

**Execution & Quality:**
| Skill | Purpose |
|-------|---------|
| `/evaluate` | Mechanical checks + independent evaluator agent |
| `/debug` | Structured debugging pipeline with impact mapping and root cause investigation |
| `/review` | Security-first code review on current branch |
| `/e2e-tests` | Generate Gherkin scenarios + Playwright implementations |
| `/deploy-check` | Pre-deployment checklist with secret scan + header check |
| `/security-audit` | Comprehensive 6-step audit (dependencies, configs, access, secrets) |

**Decision & Analysis:**
| Skill | Purpose |
|-------|---------|
| `/council` | Multi-perspective advisory council for business decisions |

**Session Management:**
| Skill | Purpose |
|-------|---------|
| `/save-and-move` | Save progress, run learning pipeline, generate continuation prompt |
| `/memory-audit` | Detect stale references in knowledge files |
| `/plan-sprint` | Sprint planning with security posture check |

**Content & Documents:**
| Skill | Purpose |
|-------|---------|
| `/presentation-generator` | HTML slides with dual outputs (PDF + responsive web) |
| `/contract-draft` | Full lifecycle contract drafting with review and redlines |
| `/docx-tracked-changes` | Word docs with revision marks and comments |

### Design Discovery

When starting UI work on a project with no established visual direction, the design-discovery skill auto-triggers:

1. Generates multiple visual mockups in different directions (using design tools)
2. Presents them for the developer to pick a direction
3. Saves the chosen theme tokens to the project
4. Future UI work follows the established direction

This prevents the "generic Bootstrap look" that AI-generated UIs tend toward.

### Plugins: Third-Party Skill Packs

Plugins extend Claude Code with curated skill sets from the community:

| Plugin | Capabilities |
|--------|-------------|
| **superpowers** | Planning, brainstorming, debugging, TDD, code review, verification workflows |
| **pr-review-toolkit** | PR review, comment analysis, silent failure hunting, type design analysis |
| **hookify** | Create and manage hook rules from conversation analysis |
| **skill-creator** | Create, modify, and benchmark custom skills |
| **frontend-design** | Creative, production-grade UI design guidance |
| **claude-md-management** | Audit and improve CLAUDE.md files |

### Building Your Own Skills

Skills are markdown files with frontmatter (name, description, trigger conditions). The skill content is loaded into the AI's context when invoked — it's not executed, it's followed.

The best skills:
- **Solve a repeatable process** (not a one-off task)
- **Encode hard-won knowledge** (the "why" behind each step)
- **Include verification steps** (don't trust the AI to self-verify)
- **Adapt to context** (read project files, don't assume stack)

---

## MCP Integration Layer

### The Problem

Context-switching between tools breaks flow. Checking deployment status in one browser tab, looking up docs in another, managing infrastructure in a third — each switch costs cognitive overhead and conversation context.

### MCP Servers

[Model Context Protocol (MCP)](https://modelcontextprotocol.io/) servers connect external tools directly to Claude Code. The AI can deploy apps, query dashboards, manage infrastructure, and access design files — all within the same conversation.

**Categories of MCP integration:**

| Category | Examples | Tools |
|----------|----------|-------|
| Infrastructure & Deployment | Container orchestration, logs, env vars, rebuilds | 30-40 |
| Design & UI | Design tools, style guides, component registries | 20-30 |
| Browser Automation | Authenticated browsing, headless testing, E2E flows | 3 complementary tools |
| Research & Documentation | Library docs, web search, crawling | 5-10 |
| Business Tools | Email, calendar, spreadsheets, dashboards | Varies |

**Browser automation routing rule:** Authenticated → real browser with sessions. Quick checks → headless browser. E2E flows/mobile → dedicated E2E framework. All browser ops serialized (never parallel).

### MCP Security Model

- **Auto-allowed** — infrastructure tools, design tools, component registries
- **Per-call permission** — business tools (email, calendar, docs) require approval each time
- **Secrets in Keychain** — API tokens stored in OS keychain, pulled by wrapper scripts at launch. Never hardcoded in config files.

---

## Implementation Reference

### File Structure

```
~/.claude/
├── CLAUDE.md              # Global preferences, standards, red lines
├── settings.json          # Hooks, permissions, deny rules
├── corrections.md         # Correction capture log (processed by /save-and-move)
├── agents/                # Custom agent definitions (5-6 agents)
│   ├── evaluator.md       # Independent functional tester (best model)
│   ├── code-reviewer.md   # Quality review (mid-tier)
│   ├── fix-reviewer.md    # Post-fix blast radius review (mid-tier)
│   ├── quick-verify.md    # Parallel verification runner
│   └── context-gatherer.md # Session start context (cheapest)
├── hooks/                 # PostToolUse / PreToolUse hook scripts
│   ├── assumption-check.sh        # Layer 3: nudge on external-system files
│   ├── assumption-check-patterns.txt  # Pattern list (self-calibrating)
│   └── assumption-check-proposals.txt # Proposed new patterns from sessions
├── skills/                # Reusable workflow skills (~20 skills)
│   ├── check-assumptions/ # Mid-session assumption surfacing
│   ├── evaluate/          # Generator-evaluator pattern
│   ├── debug/             # Structured debugging pipeline
│   ├── review/            # Code review
│   ├── deploy-check/      # Pre-deployment checklist
│   ├── plan-sprint/       # Sprint planning
│   ├── security-audit/    # Comprehensive security audit
│   ├── save-and-move/     # Session save + learning pipeline + pattern collection
│   ├── memory-audit/      # Staleness detection for knowledge files
│   ├── council/           # Multi-perspective decision council
│   ├── design-discovery/  # Visual design exploration
│   ├── working-backwards/ # Product planning
│   └── ...                # + more via plugins
├── rules/                 # Path-scoped auto-loading rules
│   ├── security.md        # Infrastructure security patterns
│   ├── llm-security.md    # LLM/AI-specific security patterns
│   ├── browser-testing.md # Browser tool routing
│   └── <language>.md      # Language-specific conventions
└── projects/              # Per-project memory (auto-managed)
    └── <project>/memory/  # MEMORY.md + topic files

~/Projects/<any-project>/
├── CLAUDE.md              # Project-specific conventions
├── docs/
│   ├── plans/             # Implementation plans (with checkboxes)
│   ├── contracts/         # Acceptance criteria (from plan review)
│   ├── tasks/             # Active execution state (nested sub-plans)
│   └── build-state/       # Evaluator reports (deleted after merge)
└── .git/hooks/
    └── pre-commit         # Secret scanning hook
```

### Hook Lifecycle

Claude Code hooks fire at specific events. Here's what each event is good for:

| Hook Event | Fires When | Use For |
|------------|-----------|---------|
| `SessionStart` | New conversation begins | Environment checks, orientation, loading context, config drift detection |
| `PreToolUse` | Before a tool runs | Dependency install warnings, command validation, TDD guards |
| `PostToolUse` | After a tool runs | Auto-formatting, secret scanning, evaluate reminders |
| `PreCompact` | Before context compaction | Saving in-flight state to disk |

Hooks are configured in `~/.claude/settings.json` and can be scoped by tool name (e.g., only fire on `Edit` and `Write` tools).

### Custom Agents

| Agent | Model Tier | Purpose |
|-------|------------|---------|
| Evaluator | Best | Independent functional testing with anti-rationalization rules |
| Code Reviewer | Mid-tier | Background quality review, 80%+ confidence threshold |
| Fix Reviewer | Mid-tier | Post-fix blast radius review — checks root cause coverage and caller impact |
| Quick Verify | Mid-tier | Parallel multi-criteria verification |
| Context Gatherer | Cheapest | Session start context loading |

### Numbers at a Glance

| Component | Count |
|-----------|-------|
| Allow rules | ~130 |
| Deny rules | ~60 |
| Hooks | 6 (across 4 event types) |
| Skills | ~20 custom + ~20 via plugins |
| Agents | 5-6 custom definitions |
| Path-scoped rules | ~9 files |
| Plugins | ~20 enabled |
| MCP servers | 8-10 connected |

### Adoption Guide

1. **Start with what hurts.** Don't implement everything at once.
   - Losing context? → [File-based progress tracking](#solution-file-based-progress-tracking)
   - Shipping bugs? → [Generator-evaluator pattern](#the-generator-evaluator-pattern)
   - Fixes causing regressions? → [Fix discipline](#fix-discipline)
   - Security gaps? → [10-stage lifecycle](#the-10-stage-security-lifecycle)
   - Wasting on models? → [Model routing](#model-routing)
   - Same mistakes repeating? → [Correction capture](#correction-capture)

2. **Automate the boring stuff.** If you have to remember to do it, you'll forget. Put it in a hook.

3. **Trust files, not memory.** Anything that matters goes in a file. Conversation context is ephemeral.

4. **Separate generation from evaluation.** The AI that wrote the code shouldn't be the only one evaluating it.

5. **Layer your defenses.** No single check catches everything.

6. **Build skills for repeatable processes.** If you explain the same workflow twice, it should be a skill.

---

## Starter Template

A minimal CLAUDE.md you can drop into any project today. It covers the highest-value patterns without requiring the full setup:

```markdown
# Project Name

## Code Standards
- Functions under 40 lines; split if cyclomatic complexity exceeds 10
- Never use display text (labels, button text) for program logic — use data attributes or state
- Conventional Commits: feat:, fix:, docs:, refactor:, chore:
- Never commit .env files, API keys, or credentials

## Safety Rules
<!-- These survive context compaction because they're in a file, not conversation -->
- Never delete files or branches without asking
- Never push to remote without asking
- Never run destructive commands (rm -rf, git reset --hard, DROP TABLE)
- Never send data to external services without asking

## Planning
- Non-trivial features: plan before coding (docs/plans/YYYY-MM-DD-feature.md)
- Plans use checkboxes: - [ ] pending, - [~] in progress, - [x] done
- Execute in batches of 3 tasks, pause for review between batches

## Fix Discipline
- Before every fix: map blast radius (epicenter → callers → callees → tests)
- After fix: run failing test, then module tests, then caller tests
- First fix fails → structured debug pipeline. No second guesses.

## Evaluation
- After >10 lines changed: run build + lint + type-check + tests
- Security header changes must be tested with a real browser, not curl
- Before reporting done, check for cop-out language in your own output

## Session Management
- Log decisions AND rejected approaches
- End sessions with a continuation prompt for next time
```

Adapt to your stack, add project-specific conventions, and grow it over time.

---

## Security Note

This guide intentionally describes **approaches and patterns** rather than specific implementations. Exact regex patterns, command allow-lists, and scanning rules are not published because:

- Published detection patterns can be studied and evaded
- Published allow-lists reveal which commands are safe injection targets
- Security through obscurity isn't a primary defense, but there's no benefit to publishing the specifics

Build your own scanning patterns, tune your own allow-lists, and treat your `settings.json` like you'd treat a firewall ruleset — configure it per your needs, don't publish it.

If you're accepting contributions to a repo that contains AI instruction files (CLAUDE.md, skills, hooks), treat them like shell scripts during code review: full human review, check for invisible unicode characters, and enable branch protection.

---

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — Use it, adapt it, share it.

## Contributing

Found a pattern that works? Open an issue or PR describing the pattern, what problem it solves, and how you validated it. The best patterns come from real-world usage, not theory.

> **Note:** This repo is maintained for personal use. Issues and PRs are welcome but not guaranteed a response on any timeline.

---

*Built by a solo developer shipping across multiple projects. Every pattern here exists because something went wrong without it.*
