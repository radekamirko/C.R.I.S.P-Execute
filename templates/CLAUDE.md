# [Project Name]

> Compile this file from your completed CRISP docs/ folder.
> This is the single source of truth Claude reads at the start of every session.
> Keep it current — update after every sprint.

---

## What this project is
_(From docs/problem-statement.md — one sentence)_


## Why it exists
_(Painkiller: what pain does it solve? For whom?)_


## Desired outcome
_(From docs/success-metrics.md — success targets, baseline, second-order effects)_


## Constraints
_(From docs/problem-statement.md — budget, time, legal, technical)_

- Budget: 
- Timeline: 
- Legal/Compliance: 
- Technical: 

## What we are building
_(Internal tool / External product. Summary of scope from docs/initial-backlog.md)_


## What we are NOT building
_(From docs/initial-backlog.md — out of scope / POST-MVP)_


## Tech stack
_(From Phase 4B tech stack proposal — pinned versions are mandatory)_

| Layer | Tool | Pinned version | Notes |
|---|---|---|---|
| | | | |

**Version rules — Claude must follow these every session:**
- Use only the versions listed above. Do not upgrade silently or assume a newer version.
- If a library's API in your training data differs from the pinned version — trust the pinned version.
- If a version conflict arises, stop and flag it. Do not resolve silently.
- Do not add dependencies not listed here without flagging it first.

## Folder structure
_(Describe key directories and what lives where)_

```
/
├── 
├── 
└── 
```

## Agents / Skills in this project
_(From Phase 4E agent map — name, responsibility, SKILL.md location)_

| Agent/Skill | Responsibility | Location |
|---|---|---|
| | | skills/[name]/SKILL.md |

## Environment Variables

> Never commit these to version control. All keys live in `.env.local` (or equivalent).
> Claude: if a required variable is missing or undefined, stop and flag it — do not proceed or hardcode a fallback.

| Variable | Purpose | Where to get it | Required? | Sprint introduced |
|---|---|---|---|---|
| | | | Yes / No | |

**Rules:**
- All secrets accessed via environment variables — never hardcoded
- `.env.local` (or equivalent) is always in `.gitignore` — verify before first commit
- Server-side secrets (OAuth tokens, service role keys) never sent to client
- If a variable is missing at runtime, fail loudly with a clear error — no silent fallbacks

**`.env.local` template** _(copy this, never commit the filled version)_:
```
# [Service name]
VARIABLE_NAME=
```

## Human-in-the-loop zones
_(From docs/risk-assessment.md — where Claude must NOT act autonomously)_

- 
- 

---

## Security Rules

> These are non-negotiable. Claude must follow them every session, every sprint, no exceptions.
> Source: CRISP security baseline. References: Claude Code Security Guidance, Microsoft Agent Governance Toolkit, NVIDIA SkillSpector.

### Secrets & credentials
- Never log, expose, or hardcode API keys, tokens, passwords, or secrets — anywhere
- All credentials accessed via environment variables only
- Verify `.gitignore` covers `.env*` and secret files before the first commit — never after
- **Never expose credentials client-side.** No `NEXT_PUBLIC_` secrets, no mobile bundle secrets, no frontend env vars with service keys
- All 3rd party API calls requiring credentials must be made server-side. The client calls your server; your server calls the 3rd party
- Server-side secrets (Stripe, OpenAI, OAuth tokens, DB service role keys) must never appear in client bundles

### Input & output handling
- Validate and sanitise all user inputs before they reach the backend — type, length, format
- Never pass raw user input directly to: database queries, shell commands, file paths, or LLM system prompts
- Sanitise all outputs before rendering — prevent XSS and injection in HTML, JSON, and log output
- Never log PII (names, emails, IDs, payment info, health data) — mask or omit in all log statements

### Authentication & authorisation
- Verify auth on every protected route and endpoint — never trust client-side role claims
- Apply least privilege: each role sees only what it needs, nothing more
- Check permissions server-side before every sensitive operation (read, write, delete, export)

### Dependencies
- Check new dependencies for known CVEs before adding: `npm audit` / `pip-audit` / `cargo audit`
- Pin versions — no `latest`, no `*`, no unpinned ranges in production dependencies
- Do not add dependencies not listed in the tech stack table without flagging first

### Static security scanning (Bearer)
- Bearer runs on every PR. Integrated in CI — do not bypass or skip
- 🔴 Critical / High findings: block the PR. Fix before merge, no exceptions
- 🟡 Medium findings: flag in PR comments, require acknowledgement
- 🟢 Low / Informational: log, no block
- Setup (GitHub Actions):
```yaml
- name: Bearer Security Scan
  uses: bearer/bearer-action@v2
  with:
    severity: critical,high
    fail-on-severity: critical,high
```

### Agent & tool governance (if this project has AI agents or tools)
> Reference: Microsoft Agent Governance Toolkit — https://github.com/microsoft/agent-governance-toolkit
> Install: `pip install agent-governance-toolkit[full]`
> Claude Code plugin: `claude --plugin-dir ./agent-governance-claude-code`

- Declare every tool's allowed action scope in `docs/agent-security.md` before the sprint that builds it
- Wrap governed tools with AGT policy enforcement — every tool call is checked against the declared policy, logged, and denied if out of scope
- Policy YAML lives at `governance/policy.yaml` — define `default_action: allow` + explicit deny rules for destructive operations
- Destructive operations (delete, drop, truncate, send, publish, deploy) require `require_approval` or explicit human confirmation — never autonomous
- All agent actions are logged with: agent identity, action type, policy decision, timestamp — tamper-evident
- In multi-agent systems: every agent has a declared identity; API keys are not shared between agents
- Circuit breaker enabled for downstream calls — cascading failures must not propagate silently
- If an agent call fails or returns unexpected output: fail loudly, log the failure, do not act on partial results
- OWASP Agentic Top 10 (ASI01–ASI11) coverage is enforced via AGT middleware — do not bypass

**Minimum policy.yaml for any project with agents:**
```yaml
apiVersion: governance.toolkit/v1
name: [project]-policy
default_action: allow
rules:
  - name: block-destructive
    condition: "action.type in ['drop', 'delete', 'truncate']"
    action: deny
    description: "Destructive operations require human approval"
  - name: require-approval-send
    condition: "action.type in ['send_email', 'send_message', 'publish', 'deploy']"
    action: require_approval
    approvers: ["human-owner"]
```

### Skill & prompt security (if this project ships AI skills or prompts)
> Reference: NVIDIA SkillSpector — https://github.com/nvidia/skillspector
> Scans for 64 vulnerability patterns across 16 categories including prompt injection, data exfiltration, privilege escalation, and supply chain risks.

- Run SkillSpector on every SKILL.md and prompt file before shipping: `skillspector scan ./skills/`
- Risk score 0–100: scores above 50 require review before merge; scores above 75 block merge
- No raw user input injected directly into system prompts — sanitise and bound all user-controlled content
- Prompt boundaries must be clearly defined — user input cannot escape its declared context
- Skills must not request more permissions than they need for their stated purpose (least privilege)
- Never include hardcoded credentials, webhook URLs, or internal endpoints in skill files

**CI step (add alongside Bearer):**
```yaml
- name: SkillSpector Scan
  run: |
    pip install skillspector
    skillspector scan ./skills/ --format sarif --output skillspector.sarif
  env:
    SKILLSPECTOR_PROVIDER: anthropic
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

## Current sprint
_(Update this at the start of each sprint)_

**Sprint:** 
**Goal:** 
**AI Spec:** `docs/ai-spec-[name].md`

## Open questions
_(Unresolved — Claude should flag these, not assume answers)_

- 

---

## CRISP Output Manifest

> All project discovery and spec documents live in `docs/`. 
> If a file is marked ✅ it exists and is current. If ❌ it is missing or incomplete — flag before proceeding with any work that depends on it.

### Phase 1 — Clarify
| File | Status | Notes |
|---|---|---|
| `docs/problem-statement.md` | ✅ / ❌ | |
| `docs/buy-vs-build-matrix.md` | ✅ / ❌ | |
| `docs/market-research.md` | ✅ / ❌ / N/A (internal) | |
| `docs/value-proposition-canvas.md` | ✅ / ❌ / N/A (internal) | |
| `docs/swot.md` | ✅ / ❌ / N/A (internal) | |

### Phase 2 — Results
| File | Status | Notes |
|---|---|---|
| `docs/stakeholder-register.md` | ✅ / ❌ | Impacted parties, HITL zones |
| `docs/success-metrics.md` | ✅ / ❌ | Baseline measurements, success targets, second-order effects |

### Phase 3 — Investigate
| File | Status | Notes |
|---|---|---|
| `docs/process-flow.md` | ✅ / ❌ | |
| `docs/user-journey-map.md` | ✅ / ❌ | |
| `docs/project-goals.md` | ✅ / ❌ | |
| `docs/ux-discovery.md` | ✅ / ❌ / N/A (non-UI) | |

### Phase 4 — Spec
| File | Status | Notes |
|---|---|---|
| `docs/design-system.md` | ✅ / ❌ / N/A (non-UI) | |
| `docs/ux-spec.md` | ✅ / ❌ / N/A (non-UI) | |
| `docs/initial-backlog.md` | ✅ / ❌ | MVP tags applied |
| `docs/assumptions-log.md` | ✅ / ❌ | |
| `docs/risk-assessment.md` | ✅ / ❌ | |
| `docs/mvp-prioritization.md` | ✅ / ❌ | HVLE scores + MVP line |
| `docs/sprint-plan.md` | ✅ / ❌ | |
| `docs/ai-spec-[sprint/feature].md` | ✅ / ❌ | One per sprint — list all below |
| `docs/ai-spec-[integration].md` | ✅ / ❌ / N/A | One per 3rd party service — list all below |
| `docs/agent-security.md` | ✅ / ❌ / N/A (no agents) | Required if any AI agents in scope |
| `governance/policy.yaml` | ✅ / ❌ / N/A (no agents) | AGT policy — required if agents in scope |

**AI Specs written:**
- [ ] `docs/ai-spec-` 
- [ ] `docs/ai-spec-` 

**Integration specs written:**
- [ ] `docs/ai-spec-` _(service name)_
- [ ] `docs/ai-spec-` _(service name)_
