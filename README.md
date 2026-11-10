# Lean Agent Team

**A token-frugal agent team for Claude Code: Fable decides and reviews; Haiku, Sonnet or Opus does the work; at most two rounds.**

A `/team` skill plus four subagents for **Claude Code**. The strongest model (Fable) is used only where judgment matters most: routing the task and independently reviewing the result. The work itself goes to the cheapest executor that can do it well. Agents hand each other file paths, not file contents, so the lead session stays small.

> **中文简介：** 一个面向 Claude Code 的精简智能体团队：Fable 负责决策与独立评审，Haiku / Sonnet / Opus 按任务难度执行，最多两轮，智能体之间只传文件路径以节省 token。详见下方 [中文摘要](#中文摘要)。

**Version 2.0.0.** New in v2: Fable as lead and critiquer, Opus added as an executor, one-step escalation. See the [changelog](CHANGELOG.md).

---

## Why this team

Asking one session to do a task and then check it means the work grades itself, and long multi-agent pipelines are expensive. Running everything on the strongest model is expensive too. This team puts the strongest model on the two decisions that determine quality, and cheaper models on the work.

| | Single session | Large multi-agent pipelines | Lean Agent Team |
|---|---|---|---|
| Independent review | No | Yes | Yes (Fable, fresh context) |
| Where the strongest model is spent | Everything, or nothing | Varies | Routing and review only |
| Token cost | Low | High (many agents, many contexts) | Low–medium (one executor, one critic) |
| Handoffs | n/a | Often pasted summaries | File paths only |
| Stopping rule | Ad hoc | Loops until a gate passes | Hard cap: two review rounds |
| Task types | Any | Mostly coding | Any (code, analysis, writing, data) |

## The team

| Role | Agent | Model | Job |
|---|---|---|---|
| Decision maker | the `/team` lead | Fable | Chooses the executor, decides on review, decides whether to escalate, reports |
| Executor | `haiku-executor` | Haiku | Mechanical, rule-based work with a checkable result |
| Executor | `sonnet-executor` | Sonnet | Default: well-scoped reasoning, coding, analysis, writing |
| Executor | `opus-executor` | Opus | Hard tasks: deep reasoning, design decisions, subtle debugging |
| Critiquer | `fable-critiquer` | Fable | Independent review, score and verdict |

## How it works

```
/team <task>
   │
   ├─ Phase 0  lead (Fable) writes .team/<run>/task.md and notes.md
   ├─ Phase 1  lead decides: mechanical → Haiku · standard → Sonnet · hard → Opus
   ├─ Phase 2  executor produces deliverables, logs decisions in notes.md
   ├─ Phase 3  fable-critiquer reviews → critique-r1.json  ── PASS ──┐
   │                                              REVISE            │
   ├─ Phase 4  lead decides: resume the same executor (default),    │
   │           or escalate one tier if the approach itself failed   │
   │           critiquer resumed, checks fixes → critique-r2.json   │
   └─ Phase 5  lead reports paths, executor, verdict, open issues ◄─┘
```

## Key features
- **Strongest model where it counts.** Fable routes and reviews; it never writes the deliverable. Each agent uses a model alias (`fable`, `opus`, `sonnet`, `haiku`), so it follows the current model generation.
- **Three executor tiers.** The lead picks the cheapest executor that fits, and takes the lower tier when unsure, because the review catches an under-powered choice.
- **One-step escalation.** If round 1 shows the approach itself is wrong, the lead can hand the task up one tier (Haiku → Sonnet, Sonnet → Opus) instead of asking the same executor to patch it. At most once per run, and never when you forced the executor with a flag.
- **Pointer-only handoffs.** Agents pass file paths; the lead never reads deliverables in full.
- **Resume, don't respawn.** Revisions continue the same executor with its context intact. If resuming is unavailable, a fresh executor rebuilds context from the run log.
- **Bounded review.** A 1–10 score (process 50%, output 50%), PASS at 7 or above, and no round 3. Round 2 reserves REVISE for real defects, not taste.
- **Cheap critic.** The reviewer spot-checks the claims that matter instead of re-running everything, and it can't edit deliverables.
- **Auditable runs.** Every run leaves `task.md`, `notes.md` (including the lead's routing decisions) and `critique-r<N>.json` in `.team/`.

## Requirements
- **Claude Code** v2.1.170 or later (the `fable` alias). This team uses Claude Code subagents, so it is not a Claude.ai or Codex skill.
- **Access to Fable** on your account. Fable is not the default model on any plan and may be billed to usage credits; check `/model` in Claude Code. No access? See [No Fable access](#no-fable-access).

## Installation (Claude Code)

```bash
git clone https://github.com/Louis-Chenyang-Lu/lean-agent-team.git
mkdir -p ~/.claude/agents ~/.claude/skills
cp lean-agent-team/agents/*.md ~/.claude/agents/
cp -r lean-agent-team/skills/team ~/.claude/skills/
```
Restart Claude Code. For one project only, copy into that project's `.claude/agents/` and `.claude/skills/` instead.

### Upgrading from v1
v1's reviewer was `opus-critiquer`; v2 replaces it with `fable-critiquer`. After copying the new files, remove the old one:
```bash
rm -f ~/.claude/agents/opus-critiquer.md
```

### No Fable access
Change `model: fable` to `model: opus` in two files: `~/.claude/agents/fable-critiquer.md` and `~/.claude/skills/team/SKILL.md`. The team then runs with an Opus lead and reviewer, as in v1, plus the Opus executor.

> If another plugin already provides a `/team` command, rename the folder `skills/team` (and the `name:` in its `SKILL.md`).

## Quick start
```
/team <describe the task and where the result should go>
```
```
/team --opus <a hard task: design decision, subtle bug, high-stakes analysis>
/team --haiku convert every CSV in <folder> to JSON, one file each
/team --no-review <a quick task you don't need checked>
```
More: [Usage guide](docs/usage-guide.md) · [Prompt templates](examples/prompts.md) · [Design and comparison](docs/design.md)

## Repository structure
```
lean-agent-team/
├── agents/
│   ├── haiku-executor.md    # mechanical executor (conversion, extraction)
│   ├── sonnet-executor.md   # default executor (reasoning, code, writing)
│   ├── opus-executor.md     # heavy-duty executor (hard tasks)
│   └── fable-critiquer.md   # independent reviewer, writes critique JSON
├── skills/team/SKILL.md     # the /team orchestrator (Fable lead instructions)
├── docs/usage-guide.md      # flags, routing, reading the results, FAQ
├── docs/design.md           # design choices and comparison with similar projects
├── examples/prompts.md      # copy-paste prompt templates
├── CHANGELOG.md
└── LICENSE
```

## Limitations
- **Claude Code only.** Claude.ai and Codex have no equivalent of these subagent files.
- **Fable costs more.** It is priced above Opus and uses plan limits faster. The design keeps its share small (short routing decisions, spot-check reviews), but a reviewed run costs more than in v1. Use `--no-review` for trivial tasks.
- **Fable may hand a request to Opus.** Claude Code re-runs requests flagged by its safety classifiers (mainly cybersecurity and biology topics) on an Opus model, so for such tasks the lead or reviewer may not be Fable.
- **A global subagent override wins.** If the `CLAUDE_CODE_SUBAGENT_MODEL` environment variable is set, every subagent runs on that model regardless of these files.
- **Resuming depends on your Claude Code version.** Continuing a subagent uses `SendMessage`, which some versions expose only with experimental agent teams enabled. Without it, the team falls back to a fresh executor that rebuilds context from the run log: it still works, but it costs more.
- **Sequential, not parallel.** One executor at a time. For work that splits into independent parallel parts, Claude Code's native agent teams or a fan-out workflow fits better.
- **The critic samples.** It spot-checks rather than re-running everything, so it can miss defects outside the claims it checks. State the tests or checks you care about in the task.

## 中文摘要
**它做什么。** `/team` 由 Fable 担任决策者：选择执行者（机械类任务用 Haiku，常规任务用 Sonnet，高难度任务用 Opus），决定是否评审、是否升级。执行完成后由 Fable 独立评审打分（过程 50%、结果 50%，7 分及以上通过），最多两轮。若第一轮表明方法本身有问题，可向上升级一档执行者（仅一次）。智能体之间只传文件路径，修改时续接原执行者，从而节省 token。
**安装。** 仅支持 Claude Code（v2.1.170 及以上，且账号可使用 Fable）：把 `agents/*.md` 复制到 `~/.claude/agents/`，把 `skills/team` 复制到 `~/.claude/skills/`，重启 Claude Code。从 v1 升级需删除 `~/.claude/agents/opus-critiquer.md`。没有 Fable 权限时，把两处 `model: fable` 改为 `model: opus`。
**语言。** 智能体以你提问的语言回复；仓库文档为英文。

## Acknowledgements
The planner/executor/critic pattern and model-tier routing are common in the Claude Code community; see [docs/design.md](docs/design.md) for related projects and how this one differs. This project is not affiliated with Anthropic or any of the projects mentioned.

## License
MIT. See [LICENSE](LICENSE).
