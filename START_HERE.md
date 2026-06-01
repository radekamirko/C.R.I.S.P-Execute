# START HERE

## Prerequisites

Before using CRISP Execute, you need:

1. **CRISP Phase S complete** — `docs/sprint-plan.md` exists and is locked
2. **AI Specs locked** — one per sprint, no open questions remaining
3. **`crisp-state.json`** — exists in project root with `handoffs.ready_for_execute: true`

If any of these are missing, go back to [C.R.I.S.P](https://github.com/radekamirko/C.R.I.S.P) and complete Phase S first.

---

## How to start

Open your project in Claude Code and say:

> **"Start Sprint 1"**

CRISP Execute will:
- Read `crisp-state.json` and `docs/sprint-plan.md`
- Check pre-conditions
- Lock the sprint scope
- Begin the build loop

---

## Mid-sprint: new requirement comes in

Say it out loud — CRISP Execute will run it through intake:

> **"New requirement: [describe it]"**

It will classify it, elicit the details, and slot it to the right sprint. The current sprint stays locked.

---

## Sprint close

When Claude Code signals the sprint is done, say:

> **"Close Sprint [N]"**

CRISP Execute will run the product gate, security gate, generate the delta doc and report, and ask for your approval before moving to the next sprint.

---

## After all sprints

When all MVP sprints are complete, CRISP Execute hands off to Phase P (Prove) — validation against your Phase R success metrics.
