# Harness Design

A multi-agent development methodology for AI coding agents. Drop these skill files into your Claude Code setup and get a structured Planner-Generator-Evaluator pipeline for building complete applications from a brief prompt.

**No code. No dependencies. Just methodology.**

## Inspiration

This project distills ideas from two sources:

- **[Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps)** by Anthropic — the Generator-Evaluator architecture, Sprint Contracts, context reset strategy, and the insight that separating building from judging eliminates self-evaluation bias.
- **[Superpowers](https://github.com/obra/superpowers)** by Jesse Vincent — TDD as an iron law, two-stage review (spec compliance then code quality), structured agent status protocols, systematic debugging, and the "do not trust the report" principle.

Both projects demonstrate that the most impactful way to improve AI coding agents isn't better models — it's better process. This project combines their core ideas into a single, opinionated workflow.

## What It Does

When you invoke `/harness`, the skill orchestrates this pipeline:

```
Brainstorming → Planning → [Sprint Contract → Generator (TDD) → Two-Stage Review → Fix Cycle] × N → Summary
                            └──────────── repeats per feature ────────────────────────────────┘
```

**Phase 0 — Environment Sensing:** Auto-detects your project's tech stack, conventions, and testing setup. Works with any language or framework.

**Phase 1 — Brainstorming:** Socratic exploration of the problem space before writing any code. Clarifies intent, explores approaches, confirms direction.

**Phase 2 — Planning:** A Planner agent expands your brief prompt into an ambitious product spec with sprint decomposition, user stories, and data models.

**Phase 3 — Sprint Contract:** Before each feature, a contract defines exact acceptance criteria — the sole grading rubric. No vague "works well." Every criterion must be independently testable.

**Phase 4 — Generation:** A Generator agent builds the feature using TDD (Red-Green-Refactor). Reports structured status: `DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, or `BLOCKED`.

**Phase 5 — Two-Stage Review:**
1. **Spec Compliance** — Did you build what was asked? Nothing more, nothing less.
2. **Code Quality** — Did you build it well? Clean, tested, maintainable.

Both reviewers are calibrated as strict skeptics. They do NOT trust the Generator's self-report. They run the code, read the output, then judge.

**Phase 6 — Completion:** Summary of what was built, sprint history, known limitations.

## Key Design Decisions

**Pure markdown.** No Python scripts, no orchestration code, no dependencies. The skill files are structured prompts that Claude Code's existing tools execute. Anyone can read, modify, or extend them.

**Tech-stack agnostic.** The methodology doesn't prescribe React, FastAPI, or anything else. It detects what your project uses and adapts. Greenfield projects get asked for their preference.

**File-driven state.** All inter-agent communication goes through `harness/` files (spec, contracts, handoffs, evaluations). These serve double duty as context reset anchors — any agent can be killed and restarted without losing state.

**Separation of building and judging.** The Generator never sees the Evaluator's prompt. The Evaluator never inherits the Generator's context. This is the core insight from Anthropic's research: self-evaluation bias is real, and the fix is architectural.

## Installation

### Claude Code

Copy the skill files into your project or global skills directory:

```bash
# Project-level (recommended — ships with your repo)
mkdir -p .claude/skills
cp path/to/harness-design/.claude/skills/harness-*.md .claude/skills/

# Or global (available in all projects)
mkdir -p ~/.claude/skills
cp path/to/harness-design/.claude/skills/harness-*.md ~/.claude/skills/
```

That's it. No build step. No config. Claude Code will discover the skills automatically.

### Verify installation

In Claude Code, type `/harness` followed by what you want to build:

```
/harness Build a browser-based DAW using the Web Audio API
```

## File Structure

```
.claude/skills/
├── harness-design.md       # Main orchestrator — 6-phase pipeline
├── harness-planner.md      # Planner agent — product vision + spec generation
├── harness-generator.md    # Generator agent — TDD + status protocol + debugging
└── harness-evaluator.md    # Evaluator agent — two-stage independent review
```

Runtime output (created during execution):

```
harness/
├── context.md              # Detected project environment
├── spec.md                 # Full product spec from Planner
├── contracts/
│   └── sprint-N.md         # Acceptance criteria per sprint
├── handoffs/
│   └── sprint-N.md         # Generator's status report per sprint
├── evaluations/
│   ├── sprint-N-spec.md    # Spec compliance review
│   └── sprint-N-quality.md # Code quality review
└── summary.md              # Final summary
```

## The Four Skill Files

### `harness-design.md` — The Orchestrator

Defines the 6-phase pipeline and how agents hand off to each other. Includes adaptive complexity (skip phases for simple tasks), error recovery, and context reset strategy.

### `harness-planner.md` — The Visionary

Turns "build me a todo app" into an ambitious, well-structured product spec. Prompted to think big but stay grounded. Outputs user stories, data models, file structure, and sprint decomposition.

### `harness-generator.md` — The Builder

Implements features using strict TDD (Red-Green-Refactor). Reports structured status instead of free-form text. Follows systematic debugging (root cause before fix) when stuck. Self-reviews before handoff but knows an independent reviewer is coming.

### `harness-evaluator.md` — The Skeptic

Two independent review passes:
1. **Spec Compliance** — Binary pass/fail against each acceptance criterion
2. **Code Quality** — Scored across dimensions (code quality, testing, UX, design)

Calibrated with concrete examples of what PASS and FAIL look like. The explicit instruction "Do NOT trust the Generator's report" prevents rubber-stamping.

## Customization

These are markdown files. Edit them.

- **Adjust TDD strictness** — If your project doesn't need TDD, modify the Generator prompt
- **Change review dimensions** — Add security review, accessibility, performance as dimensions in the Evaluator
- **Add domain expertise** — Insert domain-specific guidance into the Planner (e.g., "this is a fintech app, compliance matters")
- **Tune the skeptic** — Add your own calibration examples to the Evaluator based on failure modes you've seen

## Principles

1. **Encode the thinking, not the tooling.** These skills work because they describe HOW to think about building software, not what specific tools to use.
2. **Separation of concerns is architectural.** The one who builds must not be the one who judges. This isn't a suggestion — it's the structural foundation.
3. **Evidence before claims.** No "it works" without running the command. No "tests pass" without test output. Verification is non-negotiable.
4. **Files are the API.** Agent communication happens through files, not conversation context. This makes the process observable, debuggable, and resilient to context resets.
5. **Adaptive, not rigid.** The full pipeline exists for complex tasks. Simple tasks use a subset. The skill adapts to the work.

## License

MIT
