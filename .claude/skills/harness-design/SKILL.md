---
name: harness-design
description: >
  Multi-agent harness for long-running application development.
  Orchestrates Brainstorming, Planning, Building, and Two-Stage Review to build
  complete applications from a brief prompt. Tech-stack agnostic.
  Use when building a complete app, adding a large feature, or any multi-sprint task.
  Use this skill whenever the user mentions "harness", asks to build a full application
  from scratch, or requests a structured multi-step development workflow.
---

# Harness Design — Multi-Agent Application Builder

You are orchestrating a structured, multi-agent development process. This methodology
combines Anthropic's harness design research (separating planning, building, and
evaluation into independent agents) with disciplined engineering practices (TDD,
two-stage review, systematic debugging).

## Core Principles

1. **Separation of concerns** — The one who builds must NOT be the one who judges.
   Self-evaluation bias causes agents to "confidently praise work that is obviously mediocre."
2. **File-driven communication** — All state lives in files under `harness/`.
   This enables context resets without information loss.
3. **Sprint contracts** — Builder and Reviewer agree on "definition of done" BEFORE
   coding begins. The contract is the SOLE grading rubric.
4. **Two-stage review** — First verify spec compliance (did you build what was asked?),
   then verify code quality (did you build it well?). Never mix these concerns.
5. **Evidence before claims** — No "it works" without running the command and reading output.
   No "tests pass" without test output. Verification is non-negotiable.
6. **Tech-stack agnostic** — This process works with ANY technology. Detect, don't prescribe.
7. **TDD by default** — No production code without a failing test first. The Generator
   writes the test, watches it fail, then writes minimal code to pass.
8. **Fresh context per role** — Each agent gets a clean context with only the files it needs.
   Never inherit another agent's conversation history.

---

## File Structure

All harness state lives in `harness/` at the project root:

```
harness/
├── context.md                  # Phase 0: Project environment and conventions
├── spec.md                     # Phase 2: Full product spec from Planner
├── contracts/
│   └── sprint-N.md             # Phase 3: Acceptance criteria per sprint
├── handoffs/
│   └── sprint-N.md             # Phase 4: Generator's report (status + decisions)
├── evaluations/
│   └── sprint-N.md             # Phase 5: Two-stage review results
└── summary.md                  # Phase 6: Final summary
```

These files serve dual purposes: **communication between agents** and **recovery anchors
for context resets**. Any agent can be restarted from scratch and recover full state by
reading these files.

---

## Phase 0: Environment Sensing

Before anything else, understand the project.

**Actions:**
1. Detect tech stack from project markers:
   - `package.json` → Node.js ecosystem (React, Vue, Svelte, Next.js, etc.)
   - `requirements.txt` / `pyproject.toml` → Python ecosystem (Django, Flask, FastAPI, etc.)
   - `go.mod` → Go | `Cargo.toml` → Rust | `pom.xml` / `build.gradle` → Java/Kotlin
   - `*.csproj` / `*.sln` → .NET
   - None found → greenfield project
2. Scan existing code: directory layout, naming conventions, patterns
3. Check configs: linter, TypeScript, Docker, CI, CLAUDE.md
4. Identify testing framework and conventions already in use

**If greenfield:** Ask user for tech stack preference.

**Output:** Write to `harness/context.md`:
```markdown
# Project Context

## Tech Stack
[Detected or chosen stack]

## Existing Code
[Summary of current codebase state]

## Conventions
[Detected patterns: naming, file structure, testing approach]

## Testing
[Testing framework, test directory, how to run tests]

## Constraints
[CI requirements, deployment targets, other constraints]
```

---

## Phase 1: Brainstorming

**Do NOT skip this phase.** Before planning, explore the problem space with the user.

**Purpose:** Ensure we build the RIGHT thing, not just build something fast.

**Process:**
1. Ask clarifying questions — **one at a time**, not a wall of questions
2. For each ambiguity, offer **2-3 approaches** with trade-offs and a recommendation
3. Confirm understanding before moving on

**Key questions to explore:**
- What problem are we solving? Who are the users?
- What are the must-have vs nice-to-have features?
- Are there existing products in this space to learn from?
- What are the non-obvious constraints? (Performance, accessibility, offline, etc.)
- Where could AI capabilities add genuine value (not gimmicks)?

**Exit condition:** User confirms the direction. Then proceed to Phase 2.

**For simple/clear requests:** If the user's intent is unambiguous and the scope is small,
this phase can be brief (1-2 questions). Don't over-brainstorm obvious tasks.

---

## Phase 2: Planning

**Spawn a Planner agent** (subagent_type: "general-purpose"):

> Read `references/planner.md` (relative to this skill's directory) for your role instructions.
> Read `harness/context.md` for project context.
> The user's request is: [USER_REQUEST]
> Brainstorming conclusions: [SUMMARY_OF_PHASE_1]
>
> Generate a complete product spec and write it to `harness/spec.md`.

**After the Planner completes:**
1. Present spec summary to user — show sprints, features, scope
2. Ask: "Want to adjust scope, add/remove features, or proceed?"
3. Apply feedback by editing `harness/spec.md`
4. User confirms → proceed to Phase 3

---

## Phase 3: Sprint Planning (Contract)

Before each sprint, establish a **Sprint Contract** — the sole grading rubric.

Write to `harness/contracts/sprint-N.md`:
```markdown
# Sprint N Contract: [Feature Name]

## Scope
[What will be built]

## Acceptance Criteria
- [ ] [Specific, testable criterion — "user can click X and see Y"]
- [ ] [Each criterion independently verifiable]
- [ ] [Include both happy path and key error cases]

## Out of Scope
[Explicitly NOT part of this sprint]

## Files to Create/Modify
[Exact file paths planned for this sprint]

## How to Verify
[Commands to run, endpoints to test, UI flows to check]
```

**Rules:**
- No vague criteria ("works well", "handles errors gracefully")
- Every criterion must have a verification method
- The contract is the ONLY thing reviewers grade against

---

## Phase 4: Generation (Building)

**Spawn a Generator agent** (subagent_type: "general-purpose"):

> Read `references/generator.md` (relative to this skill's directory) for your role instructions.
>
> Read these files for context:
> - `harness/context.md` — project tech stack and conventions
> - `harness/spec.md` — full product spec
> - `harness/contracts/sprint-N.md` — THIS sprint's contract
> - Previous evaluations in `harness/evaluations/` — learn from past feedback
>
> Implement the sprint. Follow TDD. Report your status when done.

### Generator Status Protocol

The Generator reports one of four statuses:

| Status | Meaning | Your Action |
|--------|---------|-------------|
| **DONE** | All criteria implemented, tests pass | Proceed to Phase 5 (Review) |
| **DONE_WITH_CONCERNS** | Completed but has doubts | Read concerns, address if needed, then review |
| **NEEDS_CONTEXT** | Missing information to proceed | Provide context, re-dispatch |
| **BLOCKED** | Cannot complete the task | Assess: simplify scope, provide more context, or escalate to user |

**Never** ignore a BLOCKED or NEEDS_CONTEXT status. Something must change before retry.

### Context Reset Strategy

If the Generator's context becomes too long, spawn a fresh agent.
Recovery: read `harness/context.md` + `harness/spec.md` + `harness/contracts/sprint-N.md`
+ `harness/handoffs/` + explore current codebase.

This is a **full context reset** (clean slate), not compaction (summary in place).

---

## Phase 5: Two-Stage Review

Review is split into two independent passes. **Never combine them.**

### Stage 1: Spec Compliance Review

**Spawn a Spec Reviewer agent** (subagent_type: "general-purpose"):

> Read `references/evaluator.md` (relative to this skill's directory), section "Stage 1: Spec Compliance".
>
> Sprint Contract: `harness/contracts/sprint-N.md`
> Generator's handoff: `harness/handoffs/sprint-N.md`
>
> Verify: did the Generator build exactly what was specified? Nothing more, nothing less.
> CRITICAL: Do NOT trust the Generator's report. Verify everything by reading actual code.

**If spec review fails** → Generator fixes spec gaps → re-review spec compliance.
**If spec review passes** → proceed to Stage 2.

### Stage 2: Code Quality Review

**Spawn a Code Quality Reviewer agent** (subagent_type: "general-purpose"):

> Read `references/evaluator.md` (relative to this skill's directory), section "Stage 2: Code Quality".
>
> Sprint Contract: `harness/contracts/sprint-N.md`
> Project Context: `harness/context.md`
>
> Review: is the implementation well-built? Clean, tested, maintainable, secure?

**If quality review fails** → Generator fixes quality issues → re-review quality only.
**If quality review passes** → sprint complete, proceed to next sprint.

### Verification Gate

Before ANY reviewer declares PASS, they MUST:
1. **IDENTIFY** what command proves the claim
2. **RUN** the command
3. **READ** the full output
4. **VERIFY** the output confirms the claim
5. **ONLY THEN** declare PASS

"Should work" / "looks correct" / "probably passes" = automatic FAIL.

### Fix Cycle

Write evaluation to `harness/evaluations/sprint-N.md`. If issues found:

1. Spawn Generator to fix specific issues (provide evaluation file)
2. Re-run the failing review stage only
3. Max 3 fix cycles per sprint. After 3: ask user how to proceed

---

## Phase 6: Completion

When all sprints are done:

1. Write `harness/summary.md`:
```markdown
# Harness Run Summary

## Original Request
[User's original prompt]

## What Was Built
[Overview of all features]

## Sprint History
| Sprint | Feature | Review Rounds | Status |
|--------|---------|---------------|--------|
| 1      | ...     | 2             | Passed |
| 2      | ...     | 1             | Passed |

## Known Limitations
[Any accepted compromises]

## Recommendations
[Future improvements]
```

2. Present summary to user

---

## Adaptive Complexity

Not every task needs full ceremony:

| Complexity | Phases to Use |
|-----------|---------------|
| **Small** (< 1 sprint) | Skip Brainstorming & Planner. Write contract directly → Generator → Review |
| **Medium** (2-5 sprints) | Full flow |
| **Large** (5+ sprints) | Full flow + user checkpoint every 2-3 sprints |

When in doubt, ask: "This seems [simple/complex]. Want the full harness or a lighter version?"

---

## Error Recovery

| Situation | Action |
|-----------|--------|
| Agent produces garbage | Context reset: spawn fresh agent with file-based recovery |
| Agent stuck in loop | Simplify sprint scope, re-dispatch |
| Build/test failures | Generator invokes systematic debugging (root cause before fix) |
| Persistent failure (3+ attempts) | Escalate to user with full context |

**Never** retry the same action with the same context and expect different results.

---

## Orchestration Tools

- `Agent` tool → spawn each role as sub-agent (fresh context per role)
- `TaskCreate`/`TaskUpdate` → track sprint progress visibly
- `harness/` directory → persistent state and inter-agent communication
- Git commits → checkpoint after each sprint for safe rollback
