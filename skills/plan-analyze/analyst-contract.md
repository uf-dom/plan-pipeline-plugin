# Analyst brief contract (shared)

Single source of truth for the **shape of a requirements/assumptions brief** — the artifact the
Analyst produces and the Planner consumes. The analyst's job is to **define the problem and surface
the real business/domain facts**, NOT to design a solution. Keep the brief small and inspectable.

Referenced by: `plan-analyze` (produces this shape), `run-pipeline` Stage 0 (consumes + human-approves),
`write-plan` (reads it as input), `plan-audit` (may check plan-vs-requirements).

## A well-formed brief has

1. **Problem** — what is actually wanted and **why** (the underlying need), stated independently of
   any presumed solution. A reader should understand the goal, not a foregone implementation.
2. **Business / domain facts** — the real-world constraints that bound the solution: scale/volumes,
   growth horizon, usage patterns, client expectations, money/SLA/compliance. **Tag each fact as
   `[confirmed]`** (the human/owner stated it) **or `[assumed]`** (taken as given, must be challengeable).
   Facts about the *current system* (row counts, query patterns, what the code supports) must be
   **evidence-backed** — a real measurement or a code reference, not a guess.
3. **Assumptions** — everything taken as given so the Planner/Auditor can attack them.
4. **Open owner questions** — the **business/product** decisions only the human can make (distinct
   from a plan's technical `[GATE]`s). Each with options + the implication of each. These bubble up
   to the human via the conductor before/while the plan is written.
5. **Success criteria** — the measurable definition of "this solved the problem" (so the plan's
   acceptance phase and the review can be checked against it).
6. **Scope boundaries** — explicitly in vs. out; deliberate exclusions are decisions, not gaps.

## Hard rule: no solution

The brief defines the problem space. It must **not** prescribe schema, algorithms, phases, or file
edits — that is the Planner's job, in an independent context. If the analyst finds the obvious
approach, it may note it as an `[assumed direction]` fact for the planner to confirm or reject — but
it does not design it.

## The brief can conclude "no plan needed"

A legitimate analyst outcome is: the task is trivial, already solved, ill-posed, or not worth doing.
Say so plainly with the evidence — the conductor then collapses the pipeline (skip planning) instead
of manufacturing work. Defining the problem out of existence is a success, not a failure.
