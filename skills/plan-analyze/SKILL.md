---
name: plan-analyze
description: Analyst role for the plan pipeline. Produces a requirements & assumptions BRIEF (not a plan) — defines the problem, extracts the real business/domain facts, surfaces the owner decisions, and sets success criteria — so the Planner starts from a grounded, human-approved problem definition. Invoke as `/plan-analyze <task or problem>` when the problem is ambiguous, the business constraints aren't pinned down, or you want to right-size (or even cancel) a piece of work before any plan is written. Use this BEFORE write-plan whenever "what/why" is fuzzier than "how". Explicitly may conclude that no plan is needed.
---

# Analyze: produce a requirements brief (not a solution)

Your deliverable is a **brief** on disk in the shape defined by `analyst-contract.md`,
bundled alongside this SKILL.md in this skill's own directory (see the base directory noted
above) — read it first. Your goal is to make the
**problem and its real-world constraints** crisp, not to design the solution.

## What you do

1. **Investigate reality.** Ground the brief in facts, not assumptions: read the code to see what the
   system actually does/supports; where it matters, get real data (a row count, a measurement, an
   existing query pattern). Delegate this legwork to read-only research (Explore / general-purpose)
   so it doesn't bloat the brief. Cite evidence for every current-system claim.
2. **Separate confirmed from assumed.** Tag each business/domain fact `[confirmed]` (the owner said
   it) vs `[assumed]`. You cannot invent domain facts — where a fact is load-bearing and unknown,
   make it an **open owner question** rather than guessing.
3. **Surface owner decisions.** List the business/product questions only the human can answer, each
   with options and implications. In the pipeline these go to the human via the conductor; standalone,
   ask the human directly.
4. **Set success criteria and scope.** Measurable "done", and explicit in/out boundaries.

## What you do NOT do

- **No solution.** No schema, algorithms, phases, or file edits — that's the Planner's job, in an
  independent context. The most you may add is an `[assumed direction]` note for the planner to
  confirm/reject.
- **No guessing past an owner decision.** Unknown + load-bearing → open question, not an assumption
  silently baked in.

## You may conclude "no plan needed"

If the task is trivial, already done, ill-posed, or not worth it, say so with the evidence. Killing or
shrinking unnecessary work is a valid, valuable outcome — better here than after a plan is built.
