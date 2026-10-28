# Usage guide

## When to use it
- A task where a mistake would cost you more than one Opus review: code changes, analyses, reports, data transformations you will rely on.
- Mechanical batch work you want done cheaply by Haiku (`--haiku`), with or without a check.

Skip it for quick questions; a normal Claude Code turn is cheaper.

## Command
```
/team [--haiku | --sonnet] [--review | --no-review] <task>
```
| Flag | Effect |
|---|---|
| (none) | The lead picks the executor; Sonnet tasks are reviewed, Haiku tasks are not. |
| `--sonnet` / `--haiku` | Force the executor. |
| `--review` | Review even a Haiku task. |
| `--no-review` | Skip review entirely. |

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
| `notes.md` | The executor's approach, assumptions, checks, per round |
| `critique-r1.json`, `critique-r2.json` | Verdict, score, issues, suggestions |
| `out/` | Deliverables, if the task gave no location |

The lead's final message gives the deliverable paths, the verdict and score per round, any open issues (if round 2 still failed) and remaining suggestions.

## Reading the critique
- `output_issues`: defects in the deliverable. If any remain after round 2, fix them or rerun with a sharper task.
- `process_issues`: the executor skipped a check, hid an assumption, or didn't follow the task.
- `suggestions`: polish. Optional.

## FAQ
**Revisions seem to restart from scratch.** Your Claude Code version may not expose `SendMessage` for resuming subagents (some versions need experimental agent teams enabled). The fallback starts a fresh executor that reads `notes.md`; it works, but costs more.

**Another plugin also has `/team`.** Rename `skills/team` and its `name:` field, for example to `lean-team`.

**Should `.team/` be committed?** Usually not. Add `.team/` to `.gitignore`, or keep runs you want as an audit trail.

**Can I change the models?** Yes. Edit `model:` in the agent files (`opus`, `sonnet`, `haiku`, `inherit`, or a full model ID).
