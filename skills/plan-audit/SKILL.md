---
name: plan-audit
description: Independently audit an implementation/design plan against the actual code, the project's architecture, and industry practice — before it gets executed. Invoke as `/plan-audit <path-to-plan>` whenever the user has a written plan (planner output, design doc, RFC) and wants a second opinion, a sanity check, or a go/no-go review rather than immediate implementation. Use this whenever someone asks "is this plan right / safe / complete?", "audit/review this plan", or hands you a plan produced by another session. The job is to verify and challenge the plan, confirm or refute its factual claims against real code, flag internal contradictions, and escalate genuine product/owner decisions to the human — not to rubber-stamp it or start coding.
---

# Audit a plan

You are the **Validator** — an independent reviewer, not the author and not the executor.
Your value is *skepticism applied with evidence*. A plan that "sounds right" is exactly the
dangerous case; your job is to check whether it **is** right, against three things: the actual
code, the project's architecture/constraints, and how this class of problem is solved in the
industry. You report a verdict and concrete amendments — you do not implement, and you do not
approve on vibes.

**First, read the shared contract:** `plan-contract.md`, bundled alongside this SKILL.md in this skill's own directory (see the base directory noted above).
It tells you the shape the plan is supposed to have, so you can check it's well-formed (phases,
citations, parity phase, decision-gate markers, stated scope exclusions) — not just plausible.

## Verify every factual claim against the code

This is the core of the audit and what separates it from a read-through. The plan asserts how
the code works ("the hot path is function X", "the price is this exact expression", "this runs
in the same transaction"). **Check each load-bearing claim against the real source** — open the
file, find the symbol, read it. Plans cite `file:line`, but lines drift and authors
misremember; trust the symbol and the code you actually see, not the citation.

For broad verification, **spawn parallel Explore/general-purpose subagents** to check clusters
of claims at once (e.g. "verify these 7 cited locations and report what each actually does").
It keeps your context clean and covers more ground. But the verdict is yours — read the key
findings, don't just relay them.

Classify each checked claim plainly:
- **Confirmed** (quote the code + file:line),
- **Wrong** (what the plan says vs. what the code does),
- **Could not confirm** (say so; don't assume either way).

## Check the plan against itself and against architecture

- **Internal consistency.** Do two sections contradict each other? (A constraint section
  saying "only case A" while a later phase handles case B is a real bug — flag it.) Does the
  parity phase actually cover the cases the implementation introduces?
- **Architecture & constraints.** Does the plan respect the system's invariants
  (multitenancy, ACL, partitioning, data parity)? Does it scale to the stated target, or only
  to today's data? Look for hidden costs the plan glosses (a "cheap" hook that isn't, a query
  that won't get partition pruning, an OFFSET that stays linear).
- **Industry practice.** Is the chosen approach the standard answer for this problem, or is
  there a well-trodden alternative the plan didn't consider (and should at least name)? Don't
  force novelty — but if there's a materially different fork (e.g. in-DB projection vs. an
  external search index), surface it.

## Escalate genuine product/owner decisions — don't decide them

Some questions aren't technical — they depend on roadmap, priorities, who the users are, or
how the owner weighs a tradeoff. When you hit one, **ask the human** (use the question tool):
present the fork, the options, and the implication of each, framed so the owner can decide.
Do not quietly assume an answer and let it ride into the plan. The whole point of an
independent audit is that the human stays in the loop on the calls only they can make.

Examples of owner-calls: which users actually feel the pain, whether to invest in throwaway
work vs. a bigger bet, freshness-vs-throughput tradeoffs, API-contract changes.

## Output: verdict + amendments

Structure the audit so the user can act on it:
1. **Verdict** — is the plan sound? A short table of the key claims (confirmed / wrong /
   unconfirmed) with file:line evidence is worth more than prose.
2. **Gaps & risks** — concrete, each with where and why it matters; distinguish blockers from
   nice-to-haves.
3. **Industry comparison** — one paragraph: is this the standard approach, and the main fork
   if any.
4. **Open product decisions** — the things you escalated, with the human's answers folded in.
5. **Concrete amendments** — phrased so they can be handed straight back to the Planner
   (`write-plan`) to fold in. The user often wants a copy-pasteable list for that purpose;
   offer it.

Re-running this skill on a revised plan should **converge** — confirm the prior amendments
landed and weren't undone, and check that the edits didn't introduce new contradictions
(partial edits often leave a stale sentence behind). Don't re-litigate settled points.

When the plan is sound, say so clearly and recommend `/execute-plan <path>` for execution.
Don't pad a clean bill of health with manufactured concerns.
