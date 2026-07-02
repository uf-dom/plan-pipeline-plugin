---
name: write-plan
description: Produce an implementation/design plan in the house format that the plan-audit (Validator) and execute-plan (Executor) skills consume. Invoke as `/write-plan <task or problem>` whenever the user wants a careful, reviewable plan for a non-trivial change — a DB migration, a data-parity rewrite, a multi-file refactor, a perf optimization — BEFORE any code is written. Use this whenever planning is the deliverable and the work will later be audited and executed by separate sessions, so the plan must carry exact file:line+symbol citations, explicit decision-gate markers, and a runnable parity/acceptance phase. This layers a strict output shape on top of normal codebase research and planning.
---

# Write a plan

You are the **Planner**. Your deliverable is a plan document, not code. It will be handed
to a separate auditor (`plan-audit`) and then a separate executor (`execute-plan`) — neither
of whom shares your context. So the plan must stand entirely on its own and be in a shape
those roles can mechanically check and consume.

**First, read the shared contract:** `plan-contract.md`, bundled alongside this SKILL.md in this skill's own directory (see the base directory noted above).
It defines the exact shape (sections, citation rule, decision-gate markers, parity phase,
scope-exclusion convention). Produce a plan that conforms to it. The rest of this skill is
about *how to do the research well* so the plan is true, not just well-formatted.

## Research before you write

A plan is only as good as its grounding. Default planning tends to assert how code works
from memory; here you verify.

1. **Find the real hot path / touch-points by reading the code**, not by assuming. For a perf
   problem, confirm which function actually serves the slow request and why (EXPLAIN, the
   actual query). For a refactor, find every call site. Cite what you find by `file:line` +
   symbol per the contract.
2. **Surface the invariants by reading them out of the code** — the parity expression, the
   ACL clause, the partition scheme, the cache-invalidation points. These become the
   Constraints section and the parity phase. Quote the exact expressions; the Executor will
   reproduce them and the Validator will check them.
3. **For broad exploration, spawn parallel Explore subagents** so you cover more ground
   without bloating context — but verify their key claims yourself before committing them to
   the plan. The plan is your assertion, not theirs.

## Be honest about what you don't know

The most valuable thing a plan can do is mark its own uncertainty. If a design choice depends
on a measurement you haven't run, or a behavior you couldn't confirm, **don't paper over it
with a confident guess** — write it as a `[GATE]` (see the contract) with the options and what
would decide between them. A plan that says "measure X, then choose" is more useful than one
that quietly bets on the wrong branch. The Executor will stop and run the measurement; your
job is to flag that it's needed.

## Scope tightly

Plan the smallest thing that solves the stated problem. Resist folding in adjacent cleanups,
speculative generality, or "while we're here" work — those make the plan harder to audit and
execute, and bury the actual decision. When you deliberately leave something out, **say so
explicitly** (per the contract) so the next roles know it's a decision, not a gap.

## Close-out

End by stating: the chosen approach in one line, the open `[GATE]`s that still need a human
or a measurement, and a recommendation to run `/plan-audit <plan-path>` for an independent
review before execution. Don't start implementing — planning is the deliverable.
