# Changelog

## v2.0.0

**Fable decides and reviews; Haiku, Sonnet and Opus execute.**

- **Lead on Fable.** The `/team` skill now sets `model: fable`, so the routing and escalation decisions are made by the strongest model.
- **Reviewer on Fable.** `opus-critiquer` is replaced by `fable-critiquer`.
- **New `opus-executor`** for hard tasks, and a new `--opus` flag. Routing is now three tiers: Haiku (mechanical), Sonnet (default), Opus (hard).
- **One-step escalation.** After a round-1 REVISE, the lead can hand the task up one tier when the critique sets `"escalate": true` or the approach itself failed. At most once per run; never when the executor was forced with a flag.
- **Critique format.** `critique-r<N>.json` gains an `escalate` boolean.
- **Run log.** The lead records its routing and round-2 decisions in `notes.md`.
- Sonnet and Opus tasks are reviewed by default; Haiku tasks are not.

**Breaking changes**
- Delete the old reviewer after upgrading: `rm -f ~/.claude/agents/opus-critiquer.md`.
- Requires access to Fable and Claude Code v2.1.170 or later. Without Fable, change `model: fable` to `model: opus` in `fable-critiquer.md` and `skills/team/SKILL.md`.

## v1.0.0

Initial release: `/team` skill, `sonnet-executor`, `haiku-executor`, `opus-critiquer`; pointer-only handoffs, resumed revisions, two-round cap.
