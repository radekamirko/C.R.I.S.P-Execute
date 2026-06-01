# C.R.I.S.P Execute

> **Sprint execution loop for Claude Code projects.**  
> Build → Gate → Report → Repeat.

Part of the [CRISP](https://github.com/radekamirko/C.R.I.S.P) system. Takes over where Phase S (Spec) ends.

---

## What it does

CRISP Execute runs the build phase sprint by sprint:

- Locks sprint scope before Claude Code starts
- Manages change requests mid-sprint (queue, don't absorb)
- Runs product and security gates at sprint close
- Generates a delta doc and progress report per sprint
- Hands off to Phase P (Prove) when all sprints are done

---

## Quick start

1. Complete CRISP Phase S — `docs/sprint-plan.md` and AI Specs must be locked
2. Open this repo in Claude Code
3. Say: **"Start Sprint 1"**

CRISP Execute reads `crisp-state.json` and `docs/sprint-plan.md`, confirms pre-conditions, and begins the loop.

---

## The execute loop

```
Sprint N opens
  → Confirm pre-conditions
  → Claude Code builds
  → [mid-sprint: change requests → intake → queue]
  → Product gate (acceptance criteria)
  → Security gate (Bearer + checklist + agent governance)
  → Sprint delta doc
  → Progress report (technical + stakeholder)
  → Human approves
  → Sprint N+1
```

---

## Security gates

Every sprint close runs four layers:

| Layer | What it checks |
|---|---|
| Bearer scan | CVEs, secret leaks, OWASP — Critical/High blocks sprint close |
| Claude Code security checklist | Secrets, auth, input validation, HTTPS, dependency CVEs |
| Agent governance | Permissions, audit trail, failure modes (if agents built) |
| Prompt injection | Skill/prompt boundaries (if skills shipped) |

---

## Change request rule

**The current sprint is locked.** No scope changes mid-flight.

New requirements hit an intake gate — classified, elicited, slotted to a future sprint, confirmed with the human. The queue prevents loss. The lock prevents chaos.

---

## Outputs (per sprint)

| File | Contents |
|---|---|
| `docs/sprint-[N]-delta.md` | Specced vs shipped, slipped scope, change requests, gate results |
| `docs/sprint-[N]-report.md` | Technical snapshot + stakeholder report (plain language) |
| `crisp-state.json` | Updated with sprint status, gate results, change request queue |

---

## Repo structure

```
.claude/
  skills/
    crisp-execute/
      SKILL.md          ← the execute loop skill
templates/
  crisp-state.json      ← schema with execute fields
  sprint-delta.md       ← delta doc template
  sprint-report.md      ← report template (technical + stakeholder)
```

---

## Part of the CRISP system

```
C.R.I.S.P              → plan (discovery + spec)
C.R.I.S.P-Archeology   → enter from existing code
C.R.I.S.P-Execute      → build + gate + report  ← you are here
```

All three share `crisp-state.json` as the spine.

---

## License

MIT
