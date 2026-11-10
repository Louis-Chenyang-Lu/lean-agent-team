# Usage guide

## When to use it
- A task where a mistake would cost you more than one Fable review: code changes, analyses, reports, data transformations you will rely on.
- Mechanical batch work you want done cheaply by Haiku (`--haiku`), with or without a check.

Skip it for quick questions; a normal Claude Code turn is cheaper.

## Command
```
/team [--haiku | --sonnet | --opus] [--review | --no-review] <task>
```
| Flag | Effect |
|---|---|
| (none) | The Fable lead picks the executor; Sonnet and Opus tasks are reviewed, Haiku tasks are not. |
| `--haiku` / `--sonnet` / `--opus` | Force the executor. A forced executor is never escalated. |
| `--review` | Review even a Haiku task. |
| `--no-review` | Skip review entirely. |

## How the lead routes
| Executor | Chosen when |
|---|---|
| Haiku | The task is purely mechanical and the result is easy to check. |
| Sonnet | Default: scoped work with a clear definition of done. |
| Opus | The difficulty is in the thinking: no obvious approach, interacting parts, high stakes, or errors that would be hard to spot. Size alone does not qualify. |

When unsure, the lead takes the lower tier. If round 1 then shows the approach itself failed, it escalates one tier for round 2 (once per run). The routing and any escalation are logged in `notes.md` and named in the final report.

## Writing a good task
1. **The outcome.** What should exist when it's done.
2. **Where it goes.** A path or folder; otherwise deliverables land in `.team/<run>/out/`.
3. **What "correct" means.** Tests that must pass, a format to match, numbers to reconcile. The critic checks against this.
4. **Constraints.** Files not to touch, style, length.

See [../examples/prompts.md](../examples/prompts.md).

## What you get
Each run creates `.team/<YYYYMMDD-HHMM>-<slug>/`:

| File | Contents |
|---|---|
| `task.md` | Your task, verbatim |
| `notes.md` | The lead's routing decisions, and the executor's approach, assumptions and checks per round |
| `critique-r1.json`, `critique-r2.json` | Verdict, score, issues, suggestions, escalation flag |
| `out/` | Deliverables, if the task gave no location |

The lead's final message gives the deliverable paths, which executor did the work, the verdict and score per round, any open issues (if round 2 still failed) and remaining suggestions.

## Reading the critique
- `output_issues`: defects in the deliverable. If any remain after round 2, fix them or rerun with a sharper task.
- `process_issues`: the executor skipped a check, hid an assumption, or didn't follow the task.
- `suggestions`: polish. Optional.
- `escalate`: `true` when the reviewer judged that the approach was wrong or the task was beyond the executor.

## FAQ
**Revisions seem to restart from scratch.** Your Claude Code version may not expose `SendMessage` for resuming subagents (some versions need experimental agent teams enabled). The fallback starts a fresh executor that reads `notes.md`; it works, but costs more.

**Another plugin also has `/team`.** Rename `skills/team` and its `name:` field, for example to `lean-team`.

**Should `.team/` be committed?** Usually not. Add `.team/` to `.gitignore`, or keep runs you want as an audit trail.

**I don't have Fable.** Change `model: fable` to `model: opus` in `fable-critiquer.md` and `skills/team/SKILL.md`.

**The reviewer doesn't seem to run on Fable.** Check that `CLAUDE_CODE_SUBAGENT_MODEL` is not set (it overrides every agent file), that `/model` lists Fable for your account, and that you removed v1's `opus-critiquer.md`. Requests flagged by Claude Code's safety classifiers are also re-run on an Opus model.

**How do I keep Fable costs down?** Use `--no-review` for trivial tasks, write a precise task (fewer revision rounds), and say what "correct" means so the review can be a targeted check.

**Can I change the models?** Yes. Edit `model:` in the agent files and the skill (`fable`, `opus`, `sonnet`, `haiku`, `inherit`, or a full model ID).
