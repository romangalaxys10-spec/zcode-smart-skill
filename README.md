# 🧠 zcode-smart-skill

[![Built with GLM 5.3 Flash](https://img.shields.io/badge/Built%20with-GLM%205.3%20Flash-8A2BE2?style=for-the-badge&logo=z)](https://z.ai/subscribe?ic=ROK78RJKNW)
[![Lead by Roman](https://img.shields.io/badge/Lead%20by-Roman%20%7C%20Rommark.Dev-0A66C2?style=for-the-badge)](https://rommark.dev)
[![Telegram Blog](https://img.shields.io/badge/Telegram%20Blog-@VibeCodePrompterSystem-26A5E4?style=for-the-badge&logo=telegram)](https://t.me/VibeCodePrompterSystem)
[![The Claw Blog](https://img.shields.io/badge/The%20Claw-Blog-FF5733?style=for-the-badge)](https://claw.rommark.dev)
[![Author of Z-Assist Project](https://img.shields.io/badge/Author%20of-Z--Assist%20Project-2EA043?style=for-the-badge)](https://zhelp.space-z.ai/)

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-v2%20—%2010%20enhancements-purple)](#-whats-new-in-v2--10-research-backed-enhancements)
[![Platform](https://img.shields.io/badge/platform-ZCode%20%7C%20Claude--Code--style%20agents-blue)](#install)

**Smart Mode for ZCode** — a skill that ports the [GVS5H](https://github.com/slee-persis/GVS5H) ledger-orchestration technique (the one where 5 orchestrated Qwen3.8-27B instances matched Claude Fable 5 on LiveCodeBench-Hard, 92.4% vs 90.4% pass@1) into a drop-in `/smart` command with auto-triggering — **then goes 10 research-backed enhancements beyond it** (see [What's new in v2](#-whats-new-in-v2--10-research-backed-enhancements)).

> **The whole trick in one sentence:** the same model, invoked in *fresh contexts per role*, coordinating *only through a shared filesystem ledger*, beats a single long-context attempt on hard problems. Disk is the shared brain.

---

## What it does

When you hit a genuinely hard problem (or type `/smart`), your agent stops "just trying hard" and instead runs a structured multi-agent loop:

```
        ┌──────────────────────────────────────────────────────┐
        │                .smart/<hash>/ LEDGER                  │
        │  task.md  plan.md  notes.md  tasks.json  tests_spec   │
        │  solution.*  verify.log  cp-* checkpoints  workflows  │
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

**The loop:**

1. **PLAN** — manager writes a 3–6 sentence strategy + 3–6 concrete tasks, each tagged `easy|medium|hard`.
2. **IDEATE** — a fresh subagent identifies the core difficulty and proposes 3+ *genuinely distinct* approaches (different algorithms/reductions, not variations), with pitfalls. No code.
3. **TEST-SPEC** — an adversarial test-writer subagent writes edge-case + property tests *before* implementation exists.
4. **WORK** — a fresh subagent implements exactly ONE task from a disciplined handoff brief, with a mandatory self-attack pass.
5. **VERIFY** — the manager *actually runs the code* against user criteria AND the adversarial tests. **A failed verify overrides any "done".**
6. **MANAGE** — all green? finish. Otherwise pick the next task, switch approach, race approaches in parallel, or backtrack to a checkpoint.

**The anti-stuck guards (the real secret sauce):**

- After 2 failed attempts on the same approach → **switch** (serially) or **race** (parallel) — never polish a dead idea.
- Same task reissued twice in a row → stop and surface to the user (no infinite loops, no burned quota).
- `done` requires a non-empty artifact **and** green verification. No exceptions.
- Respects your blocked-loop rules: a user-only blocker is a *stop condition*, not a retry.

## Why it works

Single-context attempts on hard problems degrade: context pollution, sunk-cost polishing of a broken approach, unverified "I think this works now". GVS5H shows the fix is architectural, not scale: fresh contexts eliminate pollution, the filesystem ledger carries state across roles, diverse-approach ideation escapes local optima, and *running the code* is the only ground truth. The original paper: 5×Qwen3.8-27B (open weights, single GPU) at **92.4%** vs Claude Fable 5's 90.4% on LiveCodeBench-Hard — see [slee-persis/GVS5H](https://github.com/slee-persis/GVS5H).

## 🎉 What's new in v2 — 10 research-backed enhancements

V1 was a faithful port of GVS5H. **V2 was built by running smart mode on itself**: a research worker surveyed 2024–2026 literature (25+ sources), returned 14 ranked candidates, and the 10 strongest were integrated *into the protocol's structure* — not bolted on as a list.

| # | Enhancement | What changed in the protocol | Research basis |
|---|-------------|------------------------------|----------------|
| **E1** | 🧪 **Adversarial test-writer role** | New worker phase writes edge-case + property tests (`tests_spec.md`) *before* implementation. "Done" now means passing **someone else's** tests — implementers never grade their own homework (GPT-4 CodeContests pass@5 19%→44% in AlphaCodium) | [AlphaCodium](https://arxiv.org/abs/2401.08500) · [TestGenEval](https://testgeneval.github.io/) · [Anthropic PBT](https://www.anthropic.com/research/property-based-testing) |
| **E2** | 🪞 **Structured reflection schema** | Failures append fixed-schema entries to `notes.md` — `ROOT CAUSE / WRONG ASSUMPTION / SIGNAL / DO INSTEAD` — and the next task brief must quote them. Weight-free reinforcement for fresh-context workers | [Reflexion](https://arxiv.org/abs/2303.11366) · SAMULE (EMNLP 2025) |
| **E3** | 🏁 **Parallel approach racing** | On hard tasks / after 2 fails: 2–3 workers implement *different approaches concurrently* in isolated worktree copies; **first verified wins**, loser learnings fold into notes. Wall-clock often *drops* | [LLM Monkeys](https://arxiv.org/abs/2407.21787) · [First Finish Search](https://arxiv.org/abs/2505.18149) |
| **E4** | ⚖️ **Scrutiny: compare-then-verify** | When 2+ candidates exist, a fresh verifier subagent produces a pairwise verdict + loser defect list *before* execution-verify arbitrates. Marginal compute goes to verification, not more sampling | [Sample, Scrutinize and Scale](https://arxiv.org/pdf/2502.01839) (ICML 2025) |
| **E5** | 🎚️ **Difficulty-adaptive budgets** | Tasks tagged `easy/medium/hard`; budgets 2/6/12 iterations replace the flat cap. Easy tasks stop burning tokens; hard ones stop getting strangled. Net cost usually *negative* | [Compute-optimal TTS](https://iclr.cc/virtual/2025/oral/31924) (Snell et al.) · [RouteLLM](https://sky.cs.berkeley.edu/project/routellm/) |
| **E6** | 🎭 **Heterogeneous model-per-role** | Role→model mapping: strongest model for plan+final-verify, cheap/fast for mechanical work, *different model family* for ideation and racing branches (decorrelated errors) | [X-MAS](https://www.emergentmind.com/topics/heterogeneous-multi-agent-systems) · [ModelSwitch AAAI 2025](https://ojs.aaai.org/index.php/AAAI/article/view/39094/43056) |
| **E7** | ↩️ **Checkpointed backtracking + VALUE scores** | Non-destructive snapshots of loop-touched files before each approach; failed switches restore cleanly (debris never pollutes the next attempt); each attempt scored `VALUE: 0–10`; resume highest-value branch — file-based LATS | [LATS](https://arxiv.org/abs/2310.04406) · [SWE-Search](https://arxiv.org/abs/2410.20285) |
| **E8** | 🗡️ **Forced self-attack pass** | Workers must answer *"ATTACK: 3 ways this solution is wrong"* before reporting done — induced self-correction for ~200 tokens | [s1: budget forcing](https://arxiv.org/pdf/2501.19393) |
| **E9** | 🧷 **Cross-session workflow memory** | Global `workflows.md` (project root): after success, distill a 3–6 line reusable workflow; consulted at plan time. The ledger gets *cumulative* across sessions | [Agent Workflow Memory](https://arxiv.org/abs/2409.07429) · [Mem0](https://arxiv.org/abs/2504.19413) |
| **E10** | 📋 **Handoff discipline + calibrated judging** | Every worker brief: `OBJECTIVE / READ / OUTPUT FORMAT / BOUNDARIES / DONE-CRITERIA / TRUST` (anti-prompt-injection). Independent vs sequential declared. LLM judgment only ever *gates* execution, never replaces it; hedged verdicts escalate to running tests | [Anthropic multi-agent engineering](https://www.anthropic.com/engineering/multi-agent-research-system) · [judge-bias research](https://llm-judge-bias.github.io/) |

**❌ Explicitly rejected on evidence:** general inter-worker **debate** stages. 2025 empirical work found debate often fails to beat well-run single agents ([ICLR 2025 multi-framework eval](https://iclr.cc/virtual/2025/poster/31346)) and ensembles degrade past ~10 agents ([NeurIPS 2025](https://arxiv.org/html/2510.12697v1)). Its only defensible niches — value-estimation comparisons (E7) and pairwise scrutiny (E4) — are the ones we kept.

**Security hardening during development:** checkpoint/restore is copy-based and path-scoped (never `git reset --hard`, never commits on the user's branch, no secret-bearing commits); worker briefs carry an explicit trust boundary against ledger-content injection.

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

The agent announces **"Entering smart mode (ledger orchestration)"** so you know the cost profile changed (it spawns subagents — more tokens than a plain answer, by design; difficulty-adaptive budgets keep easy tasks cheap).

After the run, `.smart/<hash>/` stays behind as an **audit log**: read `notes.md` to see every approach tried and exactly why each died, `verify.log` for every verification run.

## Compatibility

Works in any agent harness that has: (1) a skills/prompt-injection mechanism, (2) a subagent/Task tool that starts fresh contexts, (3) file read/write. Tested with ZCode. Adapts trivially to Claude Code, Codex CLI, OpenCode, etc. — the SKILL.md is plain markdown instructions, no code to trust.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | The skill — full v2 protocol with E1–E10 integrated. This is all you need. |
| `AGENTS-snippet.md` | Optional auto-trigger rules for your AGENTS.md / CLAUDE.md |
| `LICENSE` | MIT |

## Credits

Technique from **[GVS5H](https://github.com/slee-persis/GVS5H)** (Persis Capital) — *"Five Qwen3.8-27B Models Match Claude Fable 5 on LiveCodeBench Hard"*. V2 enhancements synthesized from 2024–2026 test-time-compute and multi-agent research (full source links in the v2 table). This repo is an independent port — not affiliated with the original authors.

---

[![Built with GLM 5.3 Flash](https://img.shields.io/badge/Built%20with-GLM%205.3%20Flash-8A2BE2?style=for-the-badge)](https://z.ai/subscribe?ic=ROK78RJKNW)
