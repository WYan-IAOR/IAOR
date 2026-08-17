# From Vague Intent to Conflict-Free Parallel Delivery: How a Manager Turns Ambiguity into Organized Work

**Author**: W.Y.
**Date**: 2026-08-12

---

> The previous article gave IAOR a definition. After it ran, I kept going back through the data of that experiment. Going over it, I noticed something interesting: the brief did not even say who should write which file, yet what came out was 25 conflict-free files and a clean build. I want to tell this one story on its own — how ambiguity became order.

## The problem

Here is a scenario that happens constantly in real development. A product person hands a team a description like this:

> "Build a Kanban-style task board. Users can create, edit, and delete tasks. Tasks flow through statuses (todo → in_progress → done, cycling back to todo). There should be filtering, searching, stats, and persistence. Aim for roughly 20-25 source files. `npm run build` must pass with zero errors."

That is a product description, not an engineering plan. It says nothing about:
- Which files to create
- Which engineer owns which file
- Who owns the shared `constants.ts`
- Whether anyone can work in parallel

A single agent receiving this would simply start coding. It would make file ownership decisions implicitly, alone, with no one to conflict with — but also with no coordination, no parallelization, no verification, and no way to catch its own blind spots.

An organization faces a harder test: it must take this vague intent and turn it into clean, parallel, conflict-free work. This article is about how an IAOR does that — and the concrete numbers from a real run.

## The setup: three layers of deliberate ambiguity

To test whether a Manager can genuinely resolve ambiguity rather than merely execute a clear spec, the experiment was deliberately hardened in three ways:

**1. A product brief, not a task list.** The brief (TaskFlow-Project-Brief.md) described the *product* — features, data model, acceptance criteria — but gave no file breakdown.

**2. An ambiguous shared file.** The brief said of `constants.ts`:

> "Shared constants (status labels, priority colors, storage keys) should live in a single `src/constants.ts` file that multiple components import from."

This sentence says `constants.ts` is "shared by multiple components" — but never says who creates it. Logically it belongs to the infrastructure layer (it defines global constants); semantically it sounds tied to the components. The contradiction is deliberate. Which way would the Manager go?

**3. No "avoid overlap" hint.** The Manager's task instructions removed any guidance to "assign files so there is minimal overlap," and softened "who should own each file" into "who should work on what." The Manager had to decide on its own how to handle an ambiguous boundary.

## What the Manager did

The Manager spent 57 seconds understanding the project (reading the brief, the scaffold, confirming `src/` was empty), then made three decisions in a single planning step:

### Decision 1: It resolved the ambiguous file's ownership

The Manager was not fooled by "shared by multiple components." It judged `constants.ts` by technical architecture — it is an infrastructure-layer file (global constants), same layer as `types.ts` and `utils.ts`. So it assigned it to the infrastructure operator (004), with explicit ownership:

> "You own src/types.ts, src/constants.ts, src/utils.ts, src/api.ts, src/store.ts."

And to the frontend operator (003), it gave only import access, never write access:

> "What you import (these files will exist, built by 004): import { STATUS_LABELS, PRIORITY_COLORS, STATUS_ORDER } from '../constants'"

003 never wrote `constants.ts`. **One ambiguous file, resolved by an ownership decision — not by splitting it, not by letting the operators negotiate.**

### Decision 2: It eliminated the serial dependency by prefilling contracts

The frontend operator (003) depends on the infrastructure operator's (004) types and store. A naive Manager would run this serially: 004 finishes, then 003 starts. Instead, the Manager **prefilled 004's interface signatures directly into 003's task description** — the full type definitions and the complete Zustand store API. This let 003 begin coding immediately, in parallel, without waiting.

### Decision 3: It chose a sensible decomposition granularity

The Manager planned 25 files — 5 for infrastructure (004: types/constants/utils/api/store), 20 for frontend (003: entry files + 8 components × 2 files each). This matched the brief's "20-25" range and followed standard React organization (one `.tsx` + one `.css` per component).

## The result: 6:26, 25 files, zero conflicts

Both operators were dispatched in the same LLM turn. The parallel execution began immediately. Full timeline:

| T+ | Event |
|----|-------|
| 0s | Manager first LLM call, reading the brief |
| 57s | Manager dispatches 003 and 004 simultaneously |
| 63s | 004's first LLM response (starts infrastructure) |
| 67s | 003's first LLM response (starts components) |
| 87s | 004 finishes all 5 infrastructure files |
| 189s | 004 sends 003 a message with the full interface documentation |
| 200s | Manager notices `TaskCard.tsx` is missing |
| 210s | Manager autonomously dispatches 003 to fix it |
| 275s | 003's `npm run build` succeeds |
| 300s | 003 reports success to Manager |
| 314s | 003 sends 004 a cross-integration confirmation |
| 366s | 004 completes double-check verification, reports to Manager |
| 386s | Manager confirms, goes idle |

**Effective work time: 6 minutes 26 seconds** (from dispatch at T+57s to completion at T+386s).

| Metric | Value |
|--------|-------|
| Source files produced | 25 (5 infra + 20 frontend) |
| Operators working in parallel | 2 |
| Files written by both operators | 0 (zero conflict) |
| LLM calls | 66 (Manager 25, 003 23, 004 18) |
| Tool calls | 240 |
| `npm run build` | Passed, 53 modules, 0 errors |
| TypeScript | Zero errors, no `any` |
| Manual intervention | 0 |

The key number is **zero conflicts**. Not a single file was written by both operators. The `constants.ts` that could have been a collision point was owned exclusively by 004; 003 only read and imported it.

## Why a single agent cannot do this

A single agent is not presented with the conflict in the first place — it owns everything, so there is nothing to coordinate. But that is precisely the point: **a single agent cannot parallelize, and cannot benefit from an independent verifier.**

Compare the two:

| | Single agent | IAOR organization |
|---|-------------|-------------------|
| Ambiguous ownership | Decides implicitly, alone | Manager resolves it explicitly |
| Parallelism | Serial — one thing at a time | Two operators in parallel |
| Dependency handling | Codes against its own uncommitted work | Manager prefills contracts, zero waiting |
| Speed | One agent, one thread | 25 files in 6:26, in parallel |

A single agent, asked to build the same 25-file React app, would do it serially — probably in tens of minutes, with no one to catch its blind spots, and no way to divide the work.

<figure>
  <img src="article-images/fig3-parallel.png" alt="Both operators worked in parallel">
  <figcaption><b>Figure 3 — Parallel execution.</b> The Manager's timeline summary makes the parallelism explicit: <i>"Both operators worked in parallel: 004 built the type/store foundation while 003 built components against those interfaces."</i> Each Operator advances against its own contract.</figcaption>
</figure>

## The deeper point: contracts are how organizations parallelize

The thing that made 6:26 possible was not raw speed. It was **contracts**. The Manager turned an ambiguous whole into two decoupled workstreams joined by a precise interface contract:

- 004 builds the infrastructure against a clear ownership list
- 003 builds the frontend against a prefilled type/store contract
- The two never touch the same file because the contract defines exactly who owns what

This is how real organizations parallelize: not by "working faster," but by **defining contracts that make parallel work safe**. The contract is the enabling artifact — and in IAOR, the Manager is what produces and distributes it.

## A note on honesty

This experiment was not flawless. Two things are worth stating plainly:

1. **The task's ambiguity was deliberately manufactured.** The brief was written as a product description rather than an engineering plan, `constants.ts` ownership was left ambiguous, and the Manager's instructions deliberately dropped any "avoid overlap" hint — these difficulties were added on purpose to observe how the Manager resolves ambiguity on its own. In real settings, ambiguity can be messier, or scarcer.
2. **One operator struggled to type-check only its own files.** `tsc` checks the whole project, so the infrastructure operator could not cleanly verify just its 5 files before the frontend existed. This is an engine-level gap (partial type-checking) I am addressing.

## Conclusion

A vague, ambiguous, potentially conflicting high-level intent can be turned into clean parallel work — 25 files, zero conflicts, 6 minutes 26 seconds — if there is a layer that resolves ambiguity, defines contracts, and dispatches in parallel. That layer is what an IAOR's Manager provides.

A single agent cannot do this, because a single agent has nothing to organize.

---

*W.Y. · 2026-08-12*
