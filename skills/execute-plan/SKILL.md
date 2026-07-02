---
name: execute-plan
description: Faithfully execute a pre-written implementation/design plan file phase-by-phase, with a commit per phase, parity/contract phases run as acceptance gates, hard stops at decision-gate markers, and verification artifacts instead of "done". Invoke as `/execute-plan <path-to-plan>` whenever the user hands you an audited or finalized plan (a .md spec, a planner output, a design doc with numbered phases) and wants it implemented carefully rather than improvised. Use this whenever a separate planning/audit step already happened and the current job is execution, especially for correctness-sensitive changes (DB migrations, data-parity rewrites, multi-file refactors) where silent improvisation or skipped phases would be costly.
---

# Execute a plan

You are the **Executor**. A plan was already written and (usually) audited by someone else. Your job is to turn it into working code **faithfully** — not to redesign it, not to improve it, not to guess past its gaps. The plan is the spec; reality (the actual code) is the ground truth. When the two disagree, that is a signal to **stop and surface**, not to quietly pick a side.

This skill exists because the default failure mode of an eager executor is to (a) skip or merge phases, (b) silently resolve ambiguities by guessing, (c) report "done" without evidence, and (d) drift in scope. Everything below is aimed at those four failures.

**Shared contract:** read `plan-contract.md`, bundled alongside this SKILL.md in this skill's own directory (see the base directory noted above) — it defines the plan shape you're consuming (phases, `file:line`+symbol citations, the `[GATE]` decision markers, the parity phase, scope-exclusion convention). The plan you're handed should conform to it.

## Step 0 — Load and ground the plan

1. Read the plan file in full. If no path was given, ask for it.
2. **Verify it against reality before writing anything.** Plans are snapshots — file paths and `file:line` citations drift as the code moves. For each location the plan tells you to touch, locate it by **symbol/context** (function name, anchor string), not by trusting the line number. If a cited symbol is gone, renamed, or materially different from what the plan describes → that is a contradiction (see *Decision gates*). Do a quick pass and report any drift up front, before starting.
3. **Build the task list.** Create one task per plan phase with `TaskCreate`, in order. This makes skipped or merged phases visible and keeps long sessions honest. Mark a phase `in_progress` when you start it and `completed` only when its verification passed — not when the code is merely written.
4. **Find the acceptance phase.** Many good plans contain a "parity", "contract", "invariants", or "acceptance criteria" phase. That phase is not documentation — it is the **definition of done**. Note it now; you'll turn it into a runnable check early (see below).

## Execute phase-by-phase

Work one phase at a time, in plan order, unless the plan explicitly says a phase is independent.

- **Commit per phase.** After a phase is implemented and its checks pass, make a focused commit describing that phase. Per-phase commits keep progress durable and reviewable, and make it trivial to bisect if a later phase breaks something. (Follow the project's commit conventions; don't push unless the user asked.)
- **Don't run ahead.** Finishing phase N's code is not permission to start phase N+1 if N's verification hasn't passed. A failing gate is information — investigate the root cause rather than pressing on.

## Treat the parity/contract phase as an acceptance gate

If the plan defines a parity contract or invariants (e.g. "the new path must return bit-for-bit the same result as the old path"), **write that comparison as a runnable check early** — ideally before or alongside the implementation it validates, so you're building against a target instead of eyeballing equivalence at the end. Run it as the gate for the phases it covers. "Looks equivalent" is not parity; a green check on the plan's own stated cases is.

## Require artifacts, not adjectives

A phase is done when there is **evidence**, not when you believe it works. When the plan specifies a verification (an `EXPLAIN ANALYZE` under a latency target, a diff that must be empty, a test that must pass, a screenshot of a working UI), actually run it and **show the output**. Paste the real numbers / the real diff / the real test result. If you couldn't run something (no data, no environment), say so plainly rather than implying success. This is the single biggest guard against "complete" work that doesn't actually work.

## Decision gates — STOP, don't guess

Good plans deliberately leave some things open: they want a measurement to decide, or they flag a spot as needing judgment at implementation time. Treat these as **hard stops**: do the measurement or gather the facts, then **report and ask** — do not pick an option on the user's behalf.

Scan the plan for the decision-gate markers defined in the shared contract — the canonical `[GATE]` marker plus the natural-language signals ("решать по замеру / EXPLAIN", "открытый под-пункт", "проверить при имплементации", a phase tagged "(условный / conditional)"). Treat each as a hard stop. A conditional phase is built only if its trigger condition is actually met; if you're unsure whether it's met, ask.

Also treat as a gate **any contradiction you discover**: plan says the code does X, code actually does Y; plan's approach can't work as written; two parts of the plan conflict. Don't silently "fix" it and move on — pause, explain the discrepancy concretely (with file:line and what you see), and let the user decide. A two-minute question is cheaper than a wrong rewrite.

## Scope discipline

Implement what the plan specifies — no more. Plans are scoped on purpose; the things they explicitly exclude ("не денормализовать X", "оставить на медленном пути", "БЕЗ хука") are decisions, not oversights. Don't add abstractions, refactors, error-handling, or "while I'm here" cleanups beyond the plan. If you spot something genuinely worth doing that's out of scope, note it for later rather than folding it in.

If the plan introduces a **feature flag**, ship it **default OFF** unless the plan says otherwise. New code paths should land dark and be flipped on deliberately after verification, so a mistake can't take down the existing path.

## Close-out

When all phases are done:
1. Summarize what landed, phase by phase, with the verification evidence for each.
2. List anything that hit a decision gate and is still pending the user's call.
3. **Recommend an independent review of the diff** before it goes anywhere — e.g. `/code-review`, or handing the diff back to whoever audited the plan. One session writing and self-verifying is weaker than a fresh reviewer; say so. Don't push/merge unless the user explicitly asked.
