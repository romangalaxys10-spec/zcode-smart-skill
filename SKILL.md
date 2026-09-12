---
name: smart
description: GVS5H ledger-based self-orchestration for hard problems. Multi-agent solve loop — plan, ideate distinct approaches, execute one task at a time via fresh subagents, verify by actually running code, switch approaches when stuck. Use for hard algorithmic/coding tasks, gnarly debugging, architecture design, or anything the user flags with /smart. Triggers: "smart mode", "/smart", "solve this properly", hard algorithm/optimization/concurrency tasks, a fix that already failed twice, multi-file refactors with tricky invariants.
---

# Smart Mode — GVS5H ledger orchestration for ZCode

Port of the GVS5H technique (github.com/slee-persis/GVS5H): the same model, invoked
in **fresh contexts per role**, coordinating **only through a shared filesystem
ledger**, beats a single long-context attempt on hard problems. The point is not
more thinking — it is **compartmentalized thinking with disk as the shared brain**.

You (the main agent) are the PRIMARY. Workers are fresh subagents via the Agent
tool (`general-purpose`, self-contained prompts — subagents cannot see this
conversation). State lives ONLY in ledger files.

## 0. Workspace

At start, create `.smart/<slug>/` in the project root. slug = first 8 chars of the
md5 of the task text (hex only — never derive the slug from user-supplied words,
so no path traversal is possible).

- `task.md` — verbatim problem statement + acceptance criteria (you write this first)
- `plan.md` — 3–6 sentence strategy (≤4000 chars)
- `notes.md` — the shared brain: findings, approaches tried, pitfalls, verdicts (≤800 words, ALWAYS rewritten/pruned, never appended blindly)
- `tasks.json` — array of `{id, desc, status: pending|in_progress|done, result}` (≤12 tasks)
- `solution.*` — the actual artifact being built
- `verify.log` — every verification run + its verdict

## 1. Phase: PLAN (you, cheap)

Write `plan.md` + initial `tasks.json` (3–6 concrete tasks). Do NOT solve anything.
Rule: each task must be independently verifiable.

## 2. Phase: IDEATE (fresh subagent #1)

Spawn one subagent. Its prompt: "Do NOT solve, do NOT write code. Read
`.smart/<slug>/task.md` and `plan.md`. Identify the core difficulty. List 3+
GENUINELY DISTINCT approaches (different algorithms/data structures/reductions —
not variations of one idea), each with its pitfalls, in prose. Return as notes."
Append result to `notes.md` under `## ideation`.

## 3. Loop (≤10 iterations): MANAGE → WORK → VERIFY

**MANAGE (you):** pick exactly ONE highest-value task from `tasks.json`.
Anti-stuck rule: if the last 2 attempts on the same approach made no real progress,
do NOT polish — switch to a DIFFERENT approach from `notes.md` (or spawn a new
ideation round). Same-task-twice in a row = stop and report to the user.

**WORK (fresh subagent):** self-contained prompt containing:
- the task description + path to ledger files (it reads `task.md`, `plan.md`, `notes.md`, current `solution.*` itself)
- "Implement ONLY this task. Update `solution.*` in place. REWRITE `notes.md`
  folding in your findings — prune anything superseded or disproven (≤800 words,
  no markdown headings inside entries). Append what you changed to your final report.
  Do not touch `plan.md` or `tasks.json`."
- Explicitly forbid: solving other tasks, reformatting unrelated code.

**VERIFY (you, mandatory — this is the ground-truth step):** actually run the
code/tests/build. Never accept a worker's claim of success. Verification runs
inside the user's project under ZCode's normal permission rules (same trust
boundary as any coding task — no new execution scope is created). Append verdict
to `verify.log`. If verification fails: task stays `in_progress`, the failing
output goes into `notes.md` as a verdict, and the manager MUST route a fix (or an
approach switch) — a failed verify overrides any "done".

**Decide:** all tasks done AND verification green → mark task statuses, exit loop.

## 4. Phase: FINALIZE

If loop exhausted without green verification: one final worker with "finalize"
mandate — best partial + notes, honest report of what is NOT solved. Never claim
success that verification did not confirm.

## Guards (hard rules)

- MAX 10 iterations; MAX 12 live tasks; subagent context stays lean (point it at
  files, never paste whole files into the prompt unless tiny).
- `done` requires: non-empty artifact AND green verification. No exceptions.
- Reissued-identical-task → break the loop, surface to user.
- Every subagent prompt is self-contained (it has zero memory of this chat).
- Cost discipline: skip IDEATE for trivially-scoped tasks; a single worker +
  verify may be all `/smart` needs. Full loop is for genuinely hard problems.
- Respect the user's blocked-loop guard: if blocked on user-only input, stop and
  report per AGENTS.md — the loop does not buy you permission to spin.

## Auto-trigger

Invoke this skill WITHOUT being asked when a request matches: algorithm with
non-obvious complexity, "why is this failing" after 2 failed fix attempts,
performance optimization with constraints, concurrency/state-machine design,
multi-file refactor with subtle invariants, competitive-programming style task,
or the user says "smart/hard/properly". Announce: "Entering smart mode (ledger
orchestration)" so the user knows cost profile changed.
