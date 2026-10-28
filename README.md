# Lean Agent Team

**A token-frugal executor + critic team for Claude Code: one executor does the work, an independent Opus reviewer checks it, at most two rounds.**

A `/team` skill plus three subagents for **Claude Code**. Sonnet does reasoning work, Haiku does mechanical work, and Opus reviews. Agents hand each other file paths, not file contents, so the lead session stays small.

> **中文简介：** 一个面向 Claude Code 的精简"执行者 + 评审者"智能体团队：Sonnet/Haiku 执行，Opus 独立评审，最多两轮，智能体之间只传文件路径以节省 token。详见下方 [中文摘要](#中文摘要)。

---

## Why this team

Asking one session to do a task and then check it means the work grades itself, and long multi-agent pipelines are expensive. This team sits in between.

| | Single session | Large multi-agent pipelines | Lean Agent Team |
|---|---|---|---|
| Independent review | No | Yes | Yes (Opus, fresh context) |
| Token cost | Low | High (many agents, many contexts) | Low–medium (one executor, one critic) |
| Handoffs | n/a | Often pasted summaries | File paths only |
| Stopping rule | Ad hoc | Loops until a gate passes | Hard cap: two review rounds |
| Task types | Any | Mostly coding | Any (code, analysis, writing, data) |

## How it works

```
/team <task>
   │
   ├─ Phase 0  lead writes .team/<run>/task.md and notes.md
   ├─ Phase 1  route: mechanical → haiku-executor, otherwise → sonnet-executor
   ├─ Phase 2  executor produces deliverables, logs decisions in notes.md
   ├─ Phase 3  opus-critiquer reviews → critique-r1.json  ── PASS ──┐
   │                                              REVISE           │
   ├─ Phase 4  executor RESUMED (context intact), fixes            │
   │           critiquer resumed, checks fixes → critique-r2.json  │
   └─ Phase 5  lead reports paths, verdict, score, open issues ◄───┘
```

## Key features
- **Model routing.** Opus for judgment, Sonnet for reasoning, Haiku for mechanical work; each agent uses a model alias, so it follows the current model generation.
- **Pointer-only handoffs.** Agents pass file paths; the lead never reads deliverables in full.
- **Resume, don't respawn.** Revisions continue the same executor with its context intact, so it does not re-read everything. If resuming is unavailable, it falls back to a fresh executor that rebuilds context from the run log.
- **Bounded review.** A 1–10 score (process 50%, output 50%), PASS at 7 or above, and no round 3. Round 2 reserves REVISE for real defects, not taste.
- **Cheap critic.** The reviewer spot-checks the claims that matter instead of rebuilding or re-running everything, and it can't edit deliverables.
- **Auditable runs.** Every run leaves `task.md`, `notes.md` and `critique-r<N>.json` in `.team/`.

## Installation (Claude Code)

This team uses Claude Code subagents, so it runs in **Claude Code** only (CLI, IDE extensions, desktop). It is not a Claude.ai or Codex skill.

```bash
git clone https://github.com/Louis-Chenyang-Lu/lean-agent-team.git
mkdir -p ~/.claude/agents ~/.claude/skills
cp lean-agent-team/agents/*.md ~/.claude/agents/
cp -r lean-agent-team/skills/team ~/.claude/skills/
```
Restart Claude Code. For one project only, copy into that project's `.claude/agents/` and `.claude/skills/` instead.

> If another plugin already provides a `/team` command, rename the folder `skills/team` (and the `name:` in its `SKILL.md`), for example to `lean-team`.

## Quick start
```
/team <describe the task and where the result should go>
```
```
/team --haiku convert every CSV in <folder> to JSON, one file each
/team --no-review <a quick task you don't need checked>
```
More: [Usage guide](docs/usage-guide.md) · [Prompt templates](examples/prompts.md) · [Design and comparison](docs/design.md)

## Repository structure
```
lean-agent-team/
├── agents/
│   ├── sonnet-executor.md   # default executor (reasoning, code, writing)
│   ├── haiku-executor.md    # mechanical executor (conversion, extraction)
│   └── opus-critiquer.md    # independent reviewer, writes critique JSON
├── skills/team/SKILL.md     # the /team orchestrator (lead instructions)
├── docs/usage-guide.md      # flags, routing, reading the results, FAQ
├── docs/design.md           # design choices and comparison with similar projects
├── examples/prompts.md      # copy-paste prompt templates
└── LICENSE
```

## Limitations
- **Claude Code only.** Claude.ai and Codex have no equivalent of these subagent files.
- **Resuming depends on your Claude Code version.** Continuing a subagent uses `SendMessage`, which some versions expose only with experimental agent teams enabled. Without it, the team falls back to a fresh executor that rebuilds context from the run log: it still works, but it costs more.
- **Sequential, not parallel.** One executor at a time. For work that splits into independent parallel parts, Claude Code's native agent teams or a fan-out workflow fits better.
- **The critic samples.** It spot-checks rather than re-running everything, so it can miss defects outside the claims it checks. Ask the executor to include tests or checks you care about.
- **Opus review is the costly step.** Use `--no-review` for trivial tasks.

## 中文摘要
**它做什么。** `/team` 把任务交给一个执行者（推理类任务用 Sonnet，机械类任务用 Haiku），再由 Opus 独立评审打分（过程 50%、结果 50%，7 分及以上通过），最多两轮。智能体之间只传文件路径，修改时续接原执行者而非重新启动，从而节省 token。
**安装。** 仅支持 Claude Code：把 `agents/*.md` 复制到 `~/.claude/agents/`，把 `skills/team` 复制到 `~/.claude/skills/`，重启 Claude Code。
**语言。** 智能体以你提问的语言回复；仓库文档为英文。

## Acknowledgements
The planner/executor/critic pattern and model-tier routing are common in the Claude Code community; see [docs/design.md](docs/design.md) for related projects and how this one differs. This project is not affiliated with Anthropic or any of the projects mentioned.

## License
MIT. See [LICENSE](LICENSE).
