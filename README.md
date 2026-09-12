# 🧠 zcode-smart-skill

[![made by GLM 5.3 Flash](https://img.shields.io/badge/made%20by-GLM%205.3%20Flash-8A2BE2?style=for-the-badge&logo=z)](https://z.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-ZCode%20%7C%20Claude--Code--style%20agents-blue)](#install)

**Smart Mode for ZCode** — a skill that ports the [GVS5H](https://github.com/slee-persis/GVS5H) ledger-orchestration technique (the one where 5 orchestrated Qwen3.8-27B instances matched Claude Fable 5 on LiveCodeBench-Hard, 92.4% vs 90.4% pass@1) into a drop-in `/smart` command with auto-triggering.

> **The whole trick in one sentence:** the same model, invoked in *fresh contexts per role*, coordinating *only through a shared filesystem ledger*, beats a single long-context attempt on hard problems. Disk is the shared brain.

---

## What it does

When you hit a genuinely hard problem (or type `/smart`), your agent stops "just trying hard" and instead runs a structured multi-agent loop:

```
        ┌──────────────────────────────────────────────────────┐
        │                .smart/<hash>/ LEDGER                  │
        │  task.md   plan.md   notes.md   tasks.json            │
        │  solution.*   verify.log                              │
        └──────────────────────────────────────────────────────┘
              ▲ read/write          ▲ read/write
              │                     │
   ┌──────────┴─────────┐   ┌──────┴───────────────┐
   │  PRIMARY (main     │   │  WORKERS (fresh      │
   │  agent = manager)  │   │  subagents, zero     │
   │  plan · curate ·   │   │  memory of the chat; │
   │  verify · decide   │   │  read ledger, do ONE │
   └────────────────────┘   │  task, update notes) │
                            └──────────────────────┘
```

**The loop (max 10 iterations):**

1. **PLAN** — manager writes a 3–6 sentence strategy + 3–6 concrete tasks. Solves nothing.
2. **IDEATE** — a fresh subagent identifies the core difficulty and proposes 3+ *genuinely distinct* approaches (different algorithms/reductions, not variations), with pitfalls. No code.
3. **WORK** — a fresh subagent implements exactly ONE task, updating the artifact and rewriting `notes.md` (pruning superseded findings — the notes never bloat).
4. **VERIFY** — the manager *actually runs the code/tests/build*. A worker claiming success means nothing. **A failed verify overrides any "done".**
5. **MANAGE** — all green? finish. Otherwise pick the next single highest-value task.

**The anti-stuck guards (the real secret sauce):**

- After 2 failed attempts on the same approach → **switch to a different approach**, don't polish a dead idea.
- Same task reissued twice in a row → stop and surface to the user (no infinite loops, no burned quota).
- Caps: 10 iterations, 12 live tasks, notes ≤ 800 words.
- `done` requires a non-empty artifact **and** green verification. No exceptions.
- Respects your blocked-loop rules: a user-only blocker is a *stop condition*, not a retry.

## Why it works

Single-context attempts on hard problems degrade: context pollution, sunk-cost polishing of a broken approach, unverified "I think this works now". GVS5H shows the fix is architectural, not scale: fresh contexts eliminate pollution, the filesystem ledger carries state across roles, diverse-approach ideation escapes local optima, and *running the code* is the only ground truth. The original paper: 5×Qwen3.8-27B (open weights, single GPU) at **92.4%** vs Claude Fable 5's 90.4% on LiveCodeBench-Hard — see [slee-persis/GVS5H](https://github.com/slee-persis/GVS5H).

## Install

**Option A — shared skills root (ZCode, recommended):**

```bash
git clone https://github.com/romangalaxys10-spec/zcode-smart-skill.git
mkdir -p ~/.agents/skills
cp -R zcode-smart-skill/SKILL.md ~/.agents/skills/smart/SKILL.md
```

**Option B — ZCode user skills:**

```bash
mkdir -p ~/.zcode/skills/smart
cp zcode-smart-skill/SKILL.md ~/.zcode/skills/smart/SKILL.md
```

Then restart your agent once — `smart` appears in the skill list and `/smart` becomes callable.

**Enable auto-triggering (optional but recommended):** append [`AGENTS-snippet.md`](AGENTS-snippet.md) to your `AGENTS.md` (or `CLAUDE.md`):

```bash
cat AGENTS-snippet.md >> ~/AGENTS.md
```

## Usage

```
/smart solve this scheduling bug — it already survived two fix attempts
```

Or just ask naturally; with the snippet installed, these auto-fire smart mode:

- algorithms/optimization with non-obvious complexity
- "why is this failing?" after 2 failed fixes
- concurrency / state-machine design
- multi-file refactors with subtle invariants
- competitive-programming-style tasks
- the words *smart*, *hard*, or *properly*

The agent announces **"Entering smart mode (ledger orchestration)"** so you know the cost profile changed (it spawns subagents — more tokens than a plain answer, by design).

After the run, `.smart/<hash>/` stays behind as an **audit log**: read `notes.md` to see every approach tried and exactly why each died.

## Compatibility

Works in any agent harness that has: (1) a skills/prompt-injection mechanism, (2) a subagent/Task tool that starts fresh contexts, (3) file read/write. Tested with ZCode. Adapts trivially to Claude Code, Codex CLI, OpenCode, etc. — the SKILL.md is plain markdown instructions, no code to trust.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | The skill itself — protocol + guards. This is all you need. |
| `AGENTS-snippet.md` | Optional auto-trigger rules for your AGENTS.md / CLAUDE.md |
| `LICENSE` | MIT |

## Credits

Technique from **[GVS5H](https://github.com/slee-persis/GVS5H)** (Persis Capital) — *"Five Qwen3.8-27B Models Match Claude Fable 5 on LiveCodeBench Hard"*. This repo is an independent port of that methodology to ZCode's skill/subagent model. Not affiliated with the original authors.

---

[![made by GLM 5.3 Flash](https://img.shields.io/badge/made%20by-GLM%205.3%20Flash-8A2BE2?style=for-the-badge)](https://z.ai)
