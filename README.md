# Harness Design

A multi-agent development methodology for AI coding agents. It gives your agent a structured Planner-Generator-Evaluator pipeline so it doesn't just jump into writing code — it brainstorms, plans ambitiously, builds with TDD, and reviews its own work through an independent skeptic.

**No code. No dependencies. Just methodology.**

## How It Works

When your agent picks up a building task, it doesn't start coding immediately. Instead, it follows a disciplined pipeline:

1. **Brainstorming** — Asks you what you're really trying to build. One question at a time, explores alternatives, confirms direction before writing a single line.

2. **Planning** — A Planner agent expands your brief prompt into an ambitious product spec: user stories, data models, file structure, sprint decomposition. You review and approve before anything gets built.

3. **Sprint Contract** — Before each feature, a contract defines exact acceptance criteria. No vague "works well." Every criterion must be independently testable. This contract is the sole grading rubric.

4. **Generation (TDD)** — A Generator agent builds the feature using strict Red-Green-Refactor. It reports structured status (`DONE`, `DONE_WITH_CONCERNS`, `NEEDS_CONTEXT`, `BLOCKED`) instead of vague "I think it's done."

5. **Two-Stage Review** — Two independent reviewers check the work:
   - **Spec Compliance** — Did you build what was asked? Nothing more, nothing less.
   - **Code Quality** — Did you build it well? Clean, tested, maintainable.

   Both are calibrated as strict skeptics. They do NOT trust the Generator's self-report. They run the code, read the output, then judge.

6. **Fix Cycle** — If review fails, the Generator fixes specific issues and resubmits. Max 3 cycles before escalating to you.

Because the skill triggers automatically, you don't need to do anything special. Your coding agent just has a harness.

## Inspiration

This project distills ideas from two sources:

- **[Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps)** by Anthropic — the Generator-Evaluator architecture, Sprint Contracts, context reset strategy, and the insight that separating building from judging eliminates self-evaluation bias.
- **[Superpowers](https://github.com/obra/superpowers)** by Jesse Vincent — TDD as an iron law, two-stage review (spec compliance then code quality), structured agent status protocols, systematic debugging, and the "do not trust the report" principle.

Both projects demonstrate that the most impactful way to improve AI coding agents isn't better models — it's better process. This project combines their core ideas into a single, opinionated workflow.

## Installation

**Note:** Installation differs by platform.

### Claude Code

Clone the repo and copy the skill directory into your skills folder:

```bash
git clone https://github.com/Alex-wuhu/Harness-design.git

# Global install (available in all projects)
cp -r Harness-design ~/.claude/skills/harness-design

# Or project-level install (ships with your repo)
mkdir -p .claude/skills
cp -r Harness-design .claude/skills/harness-design
```

### Cursor

Copy the skill directory into your Cursor rules or agent config:

```bash
git clone https://github.com/Alex-wuhu/Harness-design.git
cp -r Harness-design ~/.cursor/skills/harness-design
```

Or reference the `SKILL.md` content directly in your `.cursorrules` file.

### Codex

Tell Codex:

```
Fetch and follow instructions from https://raw.githubusercontent.com/Alex-wuhu/Harness-design/main/SKILL.md
```

### Gemini CLI

Copy the skill content into your Gemini agent configuration, or reference it as an extension:

```bash
git clone https://github.com/Alex-wuhu/Harness-design.git
# Add SKILL.md content to your GEMINI.md or agent instructions
```

### Other Agents

This is pure markdown. Any AI coding agent that supports custom instructions can use it:

1. Clone this repo
2. Feed `SKILL.md` as system/custom instructions to your agent
3. Make the `references/` files available for the agent to read when spawning sub-agents

### Verify Installation

Start a new session and ask your agent to build something substantial:

```
Build a browser-based DAW using the Web Audio API
```

The agent should automatically enter the harness workflow: brainstorm first, then plan, then build with sprint contracts and two-stage review.

## What's Inside

```
harness-design/
├── SKILL.md                  # Main orchestrator — 6-phase pipeline
├── references/
│   ├── planner.md            # Planner agent prompt — product vision + spec generation
│   ├── generator.md          # Generator agent prompt — TDD + status protocol + debugging
│   └── evaluator.md          # Evaluator agent prompt — two-stage independent review
├── README.md
└── LICENSE
```

**Runtime output** (created in your project during execution):

```
harness/
├── context.md                # Detected project environment
├── spec.md                   # Full product spec from Planner
├── contracts/
│   └── sprint-N.md           # Acceptance criteria per sprint
├── handoffs/
│   └── sprint-N.md           # Generator's status report per sprint
├── evaluations/
│   ├── sprint-N-spec.md      # Spec compliance review
│   └── sprint-N-quality.md   # Code quality review
└── summary.md                # Final summary
```

## Key Design Decisions

**Pure markdown.** The skill files are structured prompts, not code. Any agent that reads markdown can use them. Anyone can read, modify, or extend them.

**Tech-stack agnostic.** The methodology doesn't prescribe React, FastAPI, or anything else. It detects what your project uses and adapts. Greenfield projects get asked for their preference.

**File-driven state.** All inter-agent communication goes through `harness/` files. These serve double duty as context reset anchors — any agent can be killed and restarted without losing state.

**Separation of building and judging.** The Generator never sees the Evaluator's prompt. The Evaluator never inherits the Generator's context. Self-evaluation bias is real, and the fix is architectural.

## Customization

These are markdown files. Edit them.

- **Adjust TDD strictness** — If your project doesn't need TDD, modify `references/generator.md`
- **Change review dimensions** — Add security, accessibility, or performance reviews in `references/evaluator.md`
- **Add domain expertise** — Insert domain-specific guidance into `references/planner.md` (e.g., "this is a fintech app, compliance matters")
- **Tune the skeptic** — Add your own calibration examples to `references/evaluator.md` based on failure modes you've seen

## Philosophy

- **Encode the thinking, not the tooling** — These skills describe HOW to think about building software, not what tools to use
- **Separation of concerns is architectural** — The one who builds must not be the one who judges
- **Evidence before claims** — No "it works" without running the command and reading the output
- **Files are the API** — Observable, debuggable, resilient to context resets
- **Adaptive, not rigid** — Full pipeline for complex tasks, subset for simple ones

## Contributing

1. Fork the repository
2. Create a branch for your changes
3. Edit the skill files
4. Submit a PR

The skill files are the product. If you have ideas for better prompts, sharper calibration examples, or new review dimensions — contributions are welcome.

## License

MIT
