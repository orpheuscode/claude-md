[README claude leak combo.md](https://github.com/user-attachments/files/26432506/README.claude.leak.combo.md)
# CLAUDE.md — Merged Production-Grade Agent Directives

Production-grade agent directives for Claude Code, built by
stress-testing and merging two independent approaches to the same
problem: how do you override Claude Code's default behaviors to produce
code that actually compiles, survives long sessions, and meets the
standard a senior engineer would approve?

## Origin

On March 31, 2026, [@iamfakeguru](https://x.com/iamfakeguru) published
a thread reverse-engineering Claude Code's internal architecture from
leaked source and billions of tokens of agent logs. The thread identified
specific mechanical bottlenecks — silent file read truncation, context
compaction destroying working memory mid-refactor, tool result clipping,
and system prompt defaults that actively bias the agent toward minimal
output over correct output — and proposed a CLAUDE.md file that overrides
all of them.

**Original thread:**
[x.com/iamfakeguru/status/2038965567269249484](https://x.com/iamfakeguru/status/2038965567269249484)

**Original repo:**
[github.com/iamfakeguru/claude-md](https://github.com/iamfakeguru/claude-md)

This repo takes that work as a starting point and merges it with an
independently developed set of directives focused on code quality
standards, architecture principles, communication protocols, and
explicit forbidden patterns — areas the original did not cover. The
result is a single file that addresses both layers: how to drive the
agent mechanically, and what good output looks like once it's running
correctly.

## What Problem This Solves

Claude Code has structural constraints that produce predictable
failures. These are not bugs — they are architectural tradeoffs baked
into how context windows, tool execution, and system prompts work. If
you do not override them, you will encounter every one of these:

| Failure | Root Cause | Directive |
|---|---|---|
| "Done!" with 40 type errors | Tool reports success when bytes hit disk, not when code compiles | §8.1 Forced Compilation Check |
| Hallucinated edits after ~15 messages | Auto-compaction silently destroys earlier context at ~167K tokens | §5.2 Context Decay Awareness |
| Band-aid fixes instead of structural solutions | System prompt default: "try the simplest approach first" | §3.1 Senior Dev Override |
| Context collapse on large refactors | Single agent = single 167K token context window | §5.3 Sub-Agent Swarming |
| Edits reference code the agent never saw | File reads silently capped at 2,000 lines / 25,000 tokens | §5.1 File Read Budget |
| Grep finds 3 results when there are 47 | Tool results truncated to ~2,000-byte preview at ~50K chars | §5.4 Tool Result Blindness |
| Rename breaks dynamic imports and barrel files | Grep is text pattern matching, not semantic code analysis | §7.2 Grep Is Not an AST |
| Agent builds before understanding requirements | No enforced separation between planning and execution | §1.2 Plan and Build Are Separate Steps |
| Same mistakes repeated every session | No cross-session learning mechanism by default | §10 Self-Improvement |
| Context lost between sessions | Agent starts fresh every time by default | §12.1 Session Continuity |
| Agent asks permission to fix obvious bugs | Default passivity in system prompt | §13.1 Autonomous Bug Fixing |
| Duplicated state introduced to "fix" display bugs | No architecture principles enforced by default | §4.4 One Source of Truth |
| Silent error swallowing in catch blocks | No error handling standards enforced | §3.7 Error Handling |
| Over-commented, robotic-sounding code | Agent defaults to verbose, corporate output style | §3.4 Write Human Code |

## Install

Drop `CLAUDE.md` in your project root. Claude Code reads it
automatically.

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/YOUR_USERNAME/YOUR_REPO/main/CLAUDE.md
```

Or clone and copy:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cp YOUR_REPO/CLAUDE.md /path/to/your/project/
```

### Compatibility

- **Claude Code** — reads CLAUDE.md from project root natively.
- **Cursor** — paste contents into `.cursorrules`.
- **Other agents** — paste into whatever rules/system prompt file your
  tool uses.

## How This File Was Built

This was not written in one pass. It was produced through a structured
comparison process between two independently developed directive sets,
followed by a gap analysis and targeted merge.

### Source A: The fakeguru CLAUDE.md

Released March 31, 2026.
[github.com/iamfakeguru/claude-md](https://github.com/iamfakeguru/claude-md)

Built from reverse-engineering Claude Code's internal source and
observing agent failure patterns at scale. Strong coverage of:

- Claude Code operational mechanics (`/compact`, `--continue`,
  `--fork-session`, `/batch`, `run_in_background`, `AskUserQuestion`)
- Context window management (file read limits, tool result truncation,
  compaction behavior)
- Sub-agent orchestration (Fork, Worktree, and /batch execution models)
- Cross-session learning (gotchas.md mistake logging, bug autopsy)
- File system as external memory (intermediate results, debug artifacts,
  progressive disclosure)
- Prompt cache management (avoiding cache invalidation mid-session)
- Intent understanding (follow references over descriptions, one-word
  mode)

### Source B: Independent directives (this project)

Built from a different angle — focused on what the agent's output should
look like once it is running correctly, and how the agent should
communicate with the developer during the process. Strong coverage of:

- Code quality specifics (TypeScript type safety rules, error handling
  patterns, consistency enforcement across naming/imports/modules)
- Architecture principles (single responsibility, interface boundaries,
  dependency direction, state ownership)
- Communication protocols (show reasoning, flag uncertainty, report
  what was not done, escalate scope changes)
- Explicit forbidden pattern blocklist with concrete code examples
- Handoff summary format for session continuity
- Import graph awareness (trace dependencies both directions, check for
  circular imports)
- Scope verification (re-run type-checker after rename to catch missed
  references)

### The Merge

A directive-by-directive comparison was performed across both files.
Each directive was scored as MISSING, WEAKER, MATCH, or STRONGER
relative to the other file. The merged output takes the strongest
version of every directive and fills every gap.

## Detailed Comparison

The following tables document exactly what each source contributed and
where the gaps were.

### Directives the OP had that we were missing

These were fully absent from Source B and pulled into the merged file.

| OP Directive | What It Addresses | Merged Location |
|---|---|---|
| Plan and Build Are Separate Steps | Agent starts coding before plan is approved | §1.2 |
| Spec-Based Development (`AskUserQuestion` tool) | Agent builds against assumptions instead of interviewing the user | §1.3 |
| Follow References, Not Descriptions | Agent ignores user's reference code and builds from its own patterns | §2.1 |
| Work From Raw Data | Agent theorizes about bugs instead of tracing actual error output | §2.2 |
| One-Word Mode | Agent restates agreed plan when user says "go," wasting context | §2.3 |
| Write Human Code | Agent produces over-commented, corporate-sounding output | §3.4 |
| Demand Elegance | Agent ships first draft without self-review on complex changes | §3.3 |
| Sub-agent execution models (Fork/Worktree/batch) | Only "launch sub-agents" — no specifics on which model to use when | §5.3 |
| Proactive Compaction (`/compact` + `context-log.md`) | No mechanism to save state before auto-compaction fires | §5.5 |
| File System as State (entire section — 6 sub-directives) | No concept of using the file system as external memory | §6 |
| Prompt Cache Awareness | No awareness that system prompt changes invalidate the session cache | §9 |
| Mistake Logging (`gotchas.md`) | No cross-session error pattern learning | §10.1 |
| Bug Autopsy | Fix and forget — no post-incident analysis | §10.2 |
| Two-Perspective Review | No mechanism to surface tradeoffs for user decision | §10.3 |
| Failure Recovery (2-strike rule) | No explicit "stop and rethink" trigger | §10.4 |
| Fresh Eyes Pass | No self-testing protocol | §10.5 |
| Session Continuity (`--continue`, `--fork-session`) | No guidance on resuming vs. starting fresh | §12.1 |
| Autonomous Bug Fixing | Agent asks for permission instead of just fixing obvious bugs | §13.1 |
| Proactive Guardrails | No offer to checkpoint before risky changes | §13.2 |
| File Hygiene (suggest splitting >500 LOC files) | No file length management | §13.3 |
| Destructive Action Safety | No safeguards on file deletion or repo pushes | §7.4 |
| One Source of Truth (don't duplicate state for display fixes) | Mentioned in architecture but not as an explicit edit safety rule | §4.4 |

### Directives we had that the OP was missing

These were fully absent from Source A and pulled into the merged file.

| Our Directive | What It Addresses | Merged Location |
|---|---|---|
| Consistency Enforcement | Agent introduces inconsistent naming/import/error patterns across files | §3.5 |
| Type Safety (no `any`, no `ts-ignore`, prefer `unknown`) | No TypeScript-specific quality rules | §3.6 |
| Error Handling (re-throw, log with detail, or return Result) | Silent error swallowing not explicitly addressed as a pattern | §3.7 |
| Single Responsibility | No file/module design principles | §4.1 |
| Interface Boundaries | Raw objects passed between module boundaries | §4.2 |
| Dependency Direction (UI → Logic → Domain → Infra) | No dependency architecture enforcement | §4.3 |
| State Ownership (centralize writes, distribute reads) | "One source of truth" mentioned but without the architectural rule | §4.4 |
| Import Graph Awareness (trace both directions, check circular deps) | Only forward search on rename — no reverse import tracing | §7.3 |
| Scope Verification (re-run type-checker post-rename) | Rename searches listed but no verification step after | §7.2 (final paragraph) |
| Show Your Work | No agent-to-human communication protocol for non-obvious decisions | §11.1 |
| Flag Uncertainty | Agent presents uncertain changes with false confidence | §11.2 |
| State What You Didn't Do | No end-of-task accounting for gaps | §11.3 |
| Escalate Scope Changes | Agent silently expands scope and runs out of context | §11.4 |
| Forbidden Patterns (explicit blocklist with code examples) | Patterns implied but never codified as a rejection list | §14 |
| Handoff Summaries (structured format) | Session continuity via `--continue` but no structured handoff | §12.3 |
| Test Integrity (update tests on behavior change, flag gaps) | Verification mentions "run tests" but no test maintenance rules | §8.2 |

### Directives both files covered (merged to strongest version)

| Directive | OP Strength | Our Strength | Merged Winner |
|---|---|---|---|
| Dead code cleanup (Step 0) | Solid | Added commented-out blocks + TODOs to cleanup list | **Ours** — slightly more complete list |
| Phased Execution (5-file limit) | Solid | Solid | **Tie** — nearly identical |
| Task Scoping (plan before executing) | Implicit in phased execution | Explicit as its own directive | **Ours** — standalone section |
| Senior Dev Override | Solid | Added "does NOT mean over-engineer" caveat | **Merged** — their framing + our guardrail |
| File Read Budget (2,000 lines) | Solid | Added concrete example (3,000-line file, only see first 2/3) | **Merged** |
| Context Decay (re-read after 10 messages) | Solid | Added the "if you think you already know, re-read now" trigger | **Merged** |
| Sub-Agent Swarming | Detailed (3 execution models, `run_in_background`, polling rule) | Core idea only | **OP** — significantly more operational detail |
| Tool Result Blindness | Solid | Added "pipe to file and read in chunks" workaround | **Merged** |
| Edit Integrity (re-read before/after, 3-edit batch limit) | Solid | Solid | **Tie** |
| Grep Is Not an AST | 6 search categories | 7 search categories (added config files) | **Ours** — one more category |
| Forced Verification (tsc + eslint + tests) | Solid | Solid | **OP** — added "checked logs and simulated real usage" |
| Emergency Protocols | Basic | Added `/compact` + `context-log.md` in degradation case | **Merged** |

### Final Scorecard

| Category | OP File | Our File | Merged File |
|---|---|---|---|
| Total directives | ~40 | ~35 | ~58 |
| Claude Code operational commands | 6 | 0 | 6 |
| Architecture/design principles | None | Strong | Strong |
| Cross-session learning | Strong | None | Strong |
| File system as external memory | Strong | None | Strong |
| Prompt cache management | Present | None | Present |
| Code quality specifics (types, errors, conventions) | Weak | Strong | Strong |
| Communication protocols | None | Strong | Strong |
| Explicit forbidden patterns | None | Present | Present |
| Verification depth | Good | Good | Combined best of both |

## Additional Notes for Developers and Vibecoders

### Why This File Exists

Claude Code's default system prompt is optimized for the average user
making small, targeted changes. That is a reasonable default. But if you
are doing production work — multi-file refactors, architectural changes,
building features from scratch — those defaults actively work against
you. The agent will:

- Choose the smallest possible change even when a structural fix is
  needed, because its system prompt says "try the simplest approach."
- Report "Done!" after writing bytes to disk, without checking if the
  code compiles, because its internal tools consider a file write
  successful if the write operation succeeds.
- Lose track of file contents after ~15 messages because auto-compaction
  fires at ~167K tokens and silently destroys earlier context.
- Miss references when renaming symbols because its search tool is
  grep, not an AST — it cannot distinguish a function call from a
  comment containing the same string.

These are not bugs. They are architectural constraints. The CLAUDE.md
override file works within these constraints by telling the agent
exactly how to compensate for them.

### Why the "Why" Matters

Every directive in this file exists because of a specific, observed
failure mode. If you strip a directive because it seems unnecessary, you
will eventually hit the failure it prevents. The file is not a style
guide — it is a set of mechanical countermeasures.

Some examples:

**"Re-read any file before editing after 10 messages"** exists because
context compaction is silent. The agent will edit a function it read 12
messages ago using its memory of what the function looked like. If
compaction has altered or removed that memory, the edit will be applied
against a stale mental model. The resulting code will compile against
what the agent thinks the file contains, not what it actually contains.

**"Pipe long command output to a file and read in chunks"** exists
because tool results over ~50,000 characters are truncated to a
~2,000-byte preview. A codebase-wide grep that returns 47 matches will
appear to return 3. The agent will proceed as if there are only 3
references, and the remaining 44 will break silently.

**"Run `npx tsc --noEmit` before reporting completion"** exists because
the agent's file write tool returns success when bytes are written to
disk. It does not run a compiler. The agent genuinely believes the task
is done because every tool call it made succeeded. Without an explicit
verification step, you will get confident "Done!" messages with dozens
of type errors.

### Tips From Our Usage

**Start with safety, not optimization.** The most impactful directives
are: Forced Compilation Check (§8.1), Context Decay Awareness (§5.2),
and Edit Integrity (§7.1). If you only take three things from this file,
take those. They prevent the three most common failure modes: shipping
broken code, editing against stale context, and silent edit failures.

**The file system sections are underrated.** §6 (File System as State)
and §6.4 (Persistent Cross-Session Memory) change how the agent
operates across sessions. Writing `gotchas.md`, `context-log.md`, and
`decisions.md` to disk gives the agent persistent memory that survives
session boundaries. Without these, every new session starts from zero.

**Conditional rules keep the main file lean.** Claude Code supports
`.claude/rules/*.md` files with YAML frontmatter that activate only when
editing matching file paths. TypeScript rules (§3.6) can live in
`.claude/rules/typescript.md` with `paths: ["**/*.ts", "**/*.tsx"]`.
Test rules can live in `.claude/rules/tests.md`. This keeps the root
CLAUDE.md focused on universal directives.

**The Sub-Agent Swarming section (§5.3) is not theoretical.** If you are
doing a refactor that touches more than 5 files, using parallel
sub-agents is the difference between a clean refactor and a session that
degrades into hallucinated edits by message 20. Five agents at ~167K
tokens each gives you ~835K tokens of working memory. One agent
processing 20 files sequentially will lose coherence by file 12.

**The forbidden patterns list (§14) is a debugging shortcut.** When
reviewing agent output, check against §14 first. These patterns are the
most common failure modes in agent-generated code, and catching them
early saves hours of debugging downstream.

### What This File Does Not Cover

- **Project-specific patterns.** This file is project-agnostic. Your
  codebase's specific conventions, module boundaries, and architectural
  decisions should live in a project context file (e.g.,
  `PROJECT-CONTEXT.md`) or in `.claude/rules/` conditional files.
- **Language-specific rules beyond TypeScript.** The type safety section
  (§3.6) is TypeScript-specific. If you work in Python, Rust, Go, or
  another language, you will want equivalent rules for your toolchain's
  type system and linter.
- **CI/CD integration.** The verification directives (§8) tell the agent
  to run type-checkers and linters locally. Integrating these into your
  CI pipeline is a separate concern.
- **MCP server configuration.** Claude Code supports external tool
  servers via the Model Context Protocol. Configuration and usage of
  MCP servers is outside the scope of this file.

## Credits

- **[@iamfakeguru](https://x.com/iamfakeguru)** — Original
  reverse-engineering thread and CLAUDE.md. AI agent infrastructure
  advisor, [@OpenServAI](https://x.com/openservai).
  - Thread:
    [x.com/iamfakeguru/status/2038965567269249484](https://x.com/iamfakeguru/status/2038965567269249484)
  - Repo:
    [github.com/iamfakeguru/claude-md](https://github.com/iamfakeguru/claude-md)
- **Spire Labs** — Independent directive development, gap analysis,
  comparison methodology, and merged output.

## License

MIT.
