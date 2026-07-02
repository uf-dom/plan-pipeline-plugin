---
name: run-pipeline
description: Orchestrate the plan pipeline end-to-end in one session — (optionally analyze →) write → audit → (revise loop) → execute → review — right-sized to the task's risk, by spawning each role as an independent subagent and passing the brief/plan files between them. Invoke as `/run-pipeline <problem description>` to start from scratch, or `/run-pipeline <path-to-existing-plan.md>` to start at the audit. Use this whenever the user wants a non-trivial change carried from idea to verified implementation with minimal babysitting, where the roles should cross-check each other and only the human gets pulled in for genuine decisions. The defining rules: the conductor loads only the stages the task warrants (trivial work skips straight to direct execution; an analyst can even conclude no plan is needed); any decision gate or product/owner question surfaces to the human and waits; the finalized plan is shown to the human for sign-off before execution; subagents never guess past a gate and never take destructive actions.
---

# Conduct the plan pipeline

You are the **Conductor**. You don't plan, audit, or implement yourself — you run those roles
as **independent subagents** and move work between them, pulling in the human only where a real
decision is needed. Your value is keeping the roles honest (independent contexts, cross-checks)
and being the single place where gates bubble up to the human.

**Read the shared contracts first:** `plan-contract.md` (plan shape, `[GATE]` markers,
scope/flag conventions) and, when running Stage 0, `analyst-contract.md` (the requirements-brief
shape) — both are bundled alongside this SKILL.md in this skill's own directory (see the base
directory noted above); read them from there. Every role relies on these.

## Two things that make this work

**1. The plan file is the hand-off medium.** Roles pass information through the plan `.md` on
disk, not through conversation. Decide the plan's path up front (if the user gave a problem,
pick a sensible path like `~/.claude/plans/<slug>.md` and tell them; if they gave a path, use
it). Every subagent reads and/or writes that file. This is also what keeps roles **independent**:
a subagent sees the plan file and the code — never another role's reasoning — so the audit is a
genuine cold check, not an echo of the planner.

**2. Gates bubble up to YOU; subagents never decide them.** A subagent cannot stop to ask the
human mid-run — only you can. So every role subagent is instructed to, instead of guessing at a
`[GATE]` or a product/owner question, **return it to you in a structured list and stop there**.
You then relay those questions to the human (use the question tool), wait for the answer, and
feed the answer into the next subagent's prompt. This is the core promise of the pipeline:
*the human is pulled in exactly at the decisions only they can make, and nowhere else.*

## Right-size the team first

Before spawning anyone, **classify the task and run only the stages its risk warrants.** Loading the
full team on a trivial change is waste; skipping rigor on a risky one is how bugs ship. Judge by the
**max of three axes — not size**: **blast radius / reversibility** (prod data, security, money,
irreversible?), **ambiguity** (do we actually know what's wanted?), **correctness-sensitivity**
(migrations, data parity, ACL, concurrency?).

- **Direct** — low on all axes (typo, mechanical fix, config tweak): just do it + verify. No subagents.
- **Light** — ambiguous but small & reversible: Analyze (or a quick human inquiry) → direct execute →
  review. Plan optional.
- **Plan** — clear problem, non-trivial solution: Write → Audit → Execute → Review. Skip the analyst.
- **Full** — ambiguous AND high-risk/correctness-sensitive: Analyze → (approve) → Write → Audit →
  (gates) → (approve plan) → Execute → Review.

**Collapse and escalate as you learn.** Any stage may conclude the next isn't needed — the analyst may
find the task trivial, already done, or not worth doing (→ skip planning); the planner may find a
one-liner (→ shrink). The reverse too: a "light" task that reveals hidden risk gets bumped up.
Re-triage is expected, not failure.

**Effort: cut stages before you cut depth.** The biggest saving is *not running a stage*, not running it
shallow. The role agents stay pinned Opus/high because their value is rigor — don't dial them down to
"save time". **Hard floor: never reduce effort/rigor on correctness-critical work** (migrations, parity,
ACL/security, irreversible or prod-affecting ops) — that's exactly where silent errors are costly. Only
lighten on low-blast-radius, easily-reversible, mechanical work and on read-only investigation (which
already runs a fast model by design).

## The stages

Track stages with `TaskCreate` so the run is visible and resumable. Spawn one subagent per
stage. Keep each subagent prompt self-contained — it has none of this conversation's context.

**Spawn the role subagents by their dedicated `subagent_type`:** `plan-analyst`, `plan-writer`,
`plan-auditor`, `plan-executor`. These agent definitions (bundled with this plugin) pin `model: opus` + `effort: high`
and point at the corresponding skill, so each role runs deep on Opus rather than inheriting a lighter
config — don't re-specify model/effort in the call, the agent carries it. For the final review use
`/code-review` (or a code-review subagent). Read-only `Explore` searches that roles spawn internally
keep their own fast model by design — don't override those.

### Stage 0 — Analyze (optional; run for "Full"-tier work, or whenever why/what is fuzzier than how)
Skip for "Plan"-tier and below (problem already clear) and when starting from an existing plan. Otherwise
spawn the `plan-analyst` subagent (`subagent_type: plan-analyst`): "Task: `<problem>`. Produce a
requirements & assumptions brief at `<brief-path>` (e.g. `~/.claude/plans/<slug>-brief.md`) per the
analyst contract. Investigate the real code/data; tag facts confirmed-vs-assumed; do NOT design a
solution; return every open owner question in a structured list. You may conclude no plan is needed."
Relay its owner questions to the human (question tool), feed the answers back so it finalizes the brief,
then **present the brief to the human for sign-off** before any planning. If the analyst (or the human)
concludes the work is unnecessary/trivial, **collapse the pipeline** — don't plan for its own sake. The
approved brief becomes an input to Stage 1 (pass `<brief-path>` to the planner).

### Stage 1 — Write (skip if started from an existing plan)
Spawn the `plan-writer` subagent (`subagent_type: plan-writer`): "Task: `<problem>`. Write the plan
to `<plan-path>`. Do NOT ask the human and do NOT resolve open questions by guessing — where
something needs a measurement or an owner decision, write it as a `[GATE]` in the plan and list every
such gate in your final reply." Read its reply; note the gates it raised. (The agent already loads
the write-plan skill + contract and runs Opus/high — don't restate those.)

### Stage 2 — Audit
Spawn the `plan-auditor` subagent (`subagent_type: plan-auditor`) — a fresh context, which is what
makes the audit independent: "Audit the plan at `<plan-path>` against the actual code. Verify the
load-bearing claims against real source. Return: verdict (sound / not yet), confirmed-vs-wrong claims
with file:line, concrete amendments phrased for the planner, and a structured list of every `[GATE]`
/ product-owner question that needs the human."

### Stage 3 — Surface gates to the human, and wait
Collect all gates/questions from Stages 1–2. If there are any, present them to the human with the
options and implications (use the question tool), and **wait**. Do not proceed past an unresolved
gate. Record the human's answers — they feed the next stage.

### Stage 4 — Revise loop (until the audit is clean)
If the audit found amendments or the human's answers change the plan, spawn the `plan-writer`
subagent to fold them in: "Apply these amendments to `<plan-path>`: `<list>`. Apply these human
decisions to the relevant `[GATE]`s: `<answers>`. Edit the plan in place; don't re-derive unaffected
parts." Then **re-audit** (Stage 2 again). Re-audit should *converge*: confirm the
amendments landed, no new contradictions appeared, settled points aren't re-litigated. Loop until
the auditor returns "sound" with no open gates. If it oscillates (~3 rounds without converging),
stop and bring the disagreement to the human rather than looping forever.

### Stage 5 — Execute
**Before executing, present the finalized plan to the human for explicit sign-off** — a concise summary
(what changes, the phases, the risks, the impact on users/clients) plus the path to the full plan file,
and wait for their go. The human is the owner and wants visibility into what is about to land; this is a
checkpoint *separate from* gate-resolution (a plan can be gate-clean and "sound" yet still need owner
approval before code lands). Only once the plan is sound, all gates are resolved, **and the human has
approved it**, spawn the `plan-executor` subagent (`subagent_type: plan-executor`): "Execute the plan at `<plan-path>` phase-by-phase. Commit per phase
locally. If you hit a `[GATE]` the plan left open, or you find a contradiction between the plan and
the code, STOP and return it to me — do not guess. Show verification artifacts (EXPLAIN output,
diffs, test results) for each phase." If it returns a gate/contradiction → back to Stage 3 (surface
to human, then resume execution with the answer).

### Stage 6 — Independent review
Spawn a code-review subagent (or run `/code-review`) on the resulting diff: "Review the diff for
correctness against the plan at `<plan-path>`." Surface its findings to the human. A separate
reviewer matters because the executor self-verifying is weaker than a fresh read.

## Guardrails

- **Never push, merge, or do anything irreversible on the human's behalf.** Per house rules,
  pushing waits for an explicit human command. Local per-phase commits are fine; the pipeline ends
  with a reviewed diff and a recommendation, not a deployment.
- **Trust but verify subagents.** A subagent's reply says what it *intended*; check the actual plan
  file / diff before treating a stage as done. If an executor claims a phase passed, the artifact it
  returned is the evidence — not its adjective.
- **Don't let independence collapse.** Resist the shortcut of doing a role yourself in your own
  context to "save a subagent" — that reintroduces the bias the pipeline exists to avoid, especially
  for the audit.
- **Keep the human's loop tight.** Batch related gates into one question round rather than pinging
  repeatedly; but never trade that for guessing past a gate.

## Close-out
Report: where the plan ended up, the diff that landed (per phase, with artifacts), the review
findings, and any gate still awaiting the human. Recommend the explicit next action (e.g. "ready to
push once you say so"). Do not take that action unprompted.
