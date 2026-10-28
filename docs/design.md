# Design and comparison

## Design choices

**One executor, one critic.** Most of the value of a second agent comes from an independent reviewer with a fresh context: the work is not grading itself. Adding planners, auditors and parallel workers multiplies cost. This team keeps the reviewer and drops the rest.

**Pointers, not content.** Each agent re-reads files it needs. The lead passes paths and reads only the small critique JSON, so its context does not grow with the size of the deliverables.

**Resume instead of respawn.** A respawned executor has to re-read the task, the files and its own reasoning. Resuming it keeps that context. Because resuming is not available in every Claude Code configuration, every decision is also written to `notes.md`, so a fresh executor can pick up from the log.

**Bounded review.** A numeric gate (PASS at 7 or above) and a hard limit of two rounds stop endless polishing. In round 2 the critic is told to reserve REVISE for correctness or completeness defects. If the work still fails, the lead reports the open issues and the user decides.

**Process counts.** The score is half process (did the executor state assumptions, verify, log honestly?) and half output. This rewards work that is easy to check.

**Model aliases.** Agents use `opus`, `sonnet` and `haiku` rather than pinned model IDs, so they follow the current generation without edits.

## How it compares

The executor + critic pattern with model-tier routing is not new. As of October 2026, related public projects include:

| Project | Shape | Main difference from this team |
|---|---|---|
| Claude Code native **agent teams** (experimental) | A lead plus several independent teammates with a shared task list and direct peer messaging | Built for parallel work and debate between teammates; uses much more tokens. This team is sequential and has a fixed reviewer. |
| [oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode) `/team` | N coordinated workers on native team tools, staged plan → exec → verify → fix loop | Parallel and coding-focused, with a larger agent roster. Note that it also uses the `/team` name. |
| [Fableathome](https://github.com/Jessomadic/Fableathome) `/build` | Opus spec → Sonnet build → Opus critique, part of a larger harness | Adds a planning phase and lives inside a full harness; this team is three agent files and one skill. |
| [mastermind](https://github.com/xcrft/mastermind) | Planner → critic → executor → auditor around a spec template | Spec-driven and code-specific; the executor follows a spec and never improvises. |
| [Quoin](https://github.com/FourthWiz/quoin) | Plan → critic → revise convergence loop with per-skill model tiers | The critic loop is applied to plans before code is written, not to finished deliverables. |

**What is distinctive here:** a task-agnostic, deliberately small team whose contract is about cost: pointer-only handoffs, a resumed executor, a critic told not to re-run everything, Haiku for single-shot mechanical work, and a hard two-round cap with a numeric gate.

**What it does not try to do:** parallel fan-out, planning before execution, or code-specific quality gates. Use one of the projects above, or Claude Code's native features, when you need those.
