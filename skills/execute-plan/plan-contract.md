# Plan contract (shared)

This is the single source of truth for the **shape** of an implementation plan in the
Planner → Validator → Executor pipeline. All three roles read this file so the artifact
one produces is exactly what the next can check and consume. Keep it stable; if it changes,
all three skills inherit the change automatically (they reference this file by path).

Referenced by: `write-plan` (produces this shape), `plan-audit` (checks this shape),
`execute-plan` (consumes this shape).

## A well-formed plan has

1. **Context** — the problem, the measured symptom (numbers if available), the chosen
   approach, and **why** that approach over the alternatives. A reader who never saw the
   problem should understand the bet being made.
2. **Constraints** — invariants that must not break (data parity, multitenancy, ACL,
   partitioning, backward-compat). State them explicitly; they bound every phase.
3. **Numbered phases** — each phase is a coherent, independently committable unit of work,
   in execution order. Small enough to verify on its own. If a phase depends on another,
   say so.
4. **A parity / contract / acceptance phase** — the **runnable definition of done**. If the
   change must preserve behavior (e.g. a new fast path equals the old path bit-for-bit),
   spell out the exact expressions/cases to reproduce so it can be turned into a check, not
   eyeballed. This is usually Phase 0 or the final verification phase.
5. **Risks** — what can go wrong, and the mitigation. Includes the deliberately-excluded
   scope (see below).
6. **Critical files** — the touch-points, each cited per the rule below.

## Code-location citations

Cite every touch-point as **`file:line` AND by symbol/anchor** (function name, a unique
string near the edit). Line numbers are a snapshot and drift as code moves; the symbol is
what survives. The Executor locates work by symbol and only uses the line as a hint, and the
Validator verifies the symbol still exists and still does what the plan claims.

## Decision-gate markers (load-bearing)

A plan should be honest about what it does **not** yet decide. Mark every such spot so the
Executor hard-stops there instead of guessing. Canonical inline marker:

> **`[GATE]`** — followed by what must be measured/confirmed and the options.

These natural-language phrasings are also treated as gates (so older plans still work):
- "решать по замеру / по EXPLAIN" · "decide by measurement / benchmark"
- "открытый под-пункт" · "open question / sub-point / TBD"
- "проверить при имплементации" · "verify during implementation / confirm whether"
- a phase tagged "(условный / conditional)" — built only if its trigger condition is met.

A gate means: gather the facts (run the measurement), then **report and ask the human** —
do not pick an option on their behalf.

## Scope exclusions are decisions

Things the plan explicitly excludes ("НЕ делаем X", "оставить на медленном пути", "БЕЗ
хука", "не денормализовать Y") are deliberate decisions, not oversights. The Executor must
not "helpfully" add them back; the Validator should confirm they're intentional, not gaps.

## Feature flags default OFF

If the plan introduces a flag, new code paths land **dark** (flag off) and are flipped on
deliberately after verification, so a mistake can't take down the existing path.
