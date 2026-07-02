# Plan Pipeline (Claude Code plugin)

A right-sized **analyze → write → audit → execute → review** pipeline for non-trivial changes.
The conductor runs each role as an **independent subagent**, passes the brief/plan `.md` file
between them (so the audit is a genuine cold check, not an echo of the planner), and pulls the
human in **only at real decision gates**.

## What's inside

**Skills** (invoked as `/plan-pipeline:<name>` once installed):

| Skill | Role |
|-------|------|
| `run-pipeline` | Conductor — orchestrates the whole pipeline end-to-end |
| `plan-analyze` | Analyst — produces a requirements & assumptions brief |
| `write-plan`   | Planner — produces the implementation plan |
| `plan-audit`   | Validator — independently audits the plan against real code |
| `execute-plan` | Executor — implements the plan phase-by-phase |

**Subagents** (spawned by the conductor via `subagent_type`; not namespaced):
`plan-analyst`, `plan-writer`, `plan-auditor`, `plan-executor` — each pins `model: opus` +
`effort: high`.

**Shared contracts** (`plan-contract.md`, `analyst-contract.md`) are bundled inside each skill
that reads them, so the pipeline is fully self-contained — no dependency on any file in your
`~/.claude/`.

## Install

From a Claude Code session:

```
/plugin marketplace add uf-dom/plan-pipeline-plugin
/plugin install plan-pipeline@plan-pipeline-marketplace
```

(If the repo is on GitLab or another host, use the full git URL instead of the `user/repo`
shorthand: `/plugin marketplace add https://gitlab.com/<team>/plan-pipeline-plugin.git`.)

Then start it with:

```
/plan-pipeline:run-pipeline <problem description>
```

or, from an existing plan file:

```
/plan-pipeline:run-pipeline <path-to-existing-plan.md>
```

## Publishing (for the author)

This repo is **both** the plugin (`.claude-plugin/plugin.json`) and a single-plugin marketplace
(`.claude-plugin/marketplace.json`, `source: "./"`). To share it:

```
cd ~/plan-pipeline-plugin
git init && git add -A && git commit -m "Plan Pipeline plugin v1.0.0"
# push to a git host, then share the repo URL
```

Bump `version` in `.claude-plugin/plugin.json` on each release so installed copies pick up
updates. (Omit `version` if you'd rather have every commit be a new version.)

## Notes on portability

- Skills reference their bundled contract by **bare filename**, resolved against the skill's
  base directory that Claude Code announces at invoke time — works regardless of where the
  plugin cache lives, and on Windows (no symlinks).
- Subagents reference their skill by **namespaced name** (`plan-pipeline:<skill>`), not by an
  absolute path — so nothing points at `~/.claude/skills/...`. If you rename the plugin, update
  those references in `agents/*.md`.
