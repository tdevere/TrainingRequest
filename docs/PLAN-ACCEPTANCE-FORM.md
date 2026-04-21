# Plan Acceptance Form

> A required, structured artifact that every drafted training plan and PDP
> roadmap must include. The Intake Agent fills it in, the Critic Agent
> scores it, and the Reviewer uses it to approve without guessing.
>
> This form bakes in the evaluation dimensions that have historically
> caused plans to ship weak (see the "Why these fields" section below).
> **Do not skip fields.** Use `n/a` with a one-line justification if a
> field truly doesn't apply; the Critic Agent will reject blank fields.

---

## Required fields (training plan)

Copy this block into the plan comment. Every line must be filled.

```markdown
### Plan Acceptance Form — Training

**Topic:** <string>
**Slug:** <kebab-case>
**Level:** <beginner | intermediate | advanced>
**Learner time:** <minutes>
**Build effort:** <S | M | L>

#### 1. Concrete deployment target
- Platform: <Azure App Service | Azure Container Apps | AKS | local Docker | n/a with reason>
- Why this target (1 line):

#### 2. Observable learning outcomes (2–5)
Each outcome MUST start with a verb that is externally verifiable.
| # | Outcome | How it's demonstrated |
|---|---------|----------------------|
| 1 |         |                      |
| 2 |         |                      |

#### 3. Artifact discipline
- Build-once, deploy-same artifact: <yes | no | n/a + reason>
- Artifact storage location: <e.g. GHCR, ACR, releases>
- Version identifier: <immutable tag | SHA | semver>

#### 4. Rollback treatment
- Rollback mechanism taught: <redeploy prior artifact | DB-aware | feature flag | n/a + reason>
- Stateful concerns covered: <yes | no>
  - If no, explicitly name what's out of scope:

#### 5. Failure realism
Describe the hands-on failure scenarios. At least ONE must be multi-layer
or ambiguous (not a single-cause, pre-labeled break).
- Scenario A (simple):
- Scenario B (ambiguous / multi-cause):
- Raw-log interpretation task included: <yes | no>

#### 6. Secrets & security hygiene
- Secrets mechanism taught: <GH Actions secrets | Key Vault | OIDC | n/a + reason>
- Anti-patterns explicitly called out: <e.g. hardcoded tokens, over-scoped PATs>
- Token scoping / least privilege mentioned: <yes | no>

#### 7. Health validation
- "Healthy" is defined as: <>
- Beyond HTTP 200, the lab checks: <e.g. smoke test, log line, metric>
- Observability ceiling (what we explicitly don't teach here):

#### 8. Definition of Done (learner-side)
A learner is done when they can, unaided:
- [ ] <observable task 1>
- [ ] <observable task 2>
- [ ] Explain (in writing or verbally) <one decision they made and why>

#### 9. Prerequisites
- Required:
- Assumed-not-required (listed so learner self-filters):

#### 10. Out of scope (protect the time budget)
-
-

#### 11. Proposed SME
- Handle: <@handle or `tbd`>
- Rationale:

#### 12. Open questions for the human Reviewer
- <If none, write "None.">
```

---

## Required fields (PDP roadmap)

```markdown
### Plan Acceptance Form — PDP

**Goal:** <one sentence>
**Horizon:** <months>
**Weekly time budget:** <hours>

#### 1. Target capabilities (3–7, observable)
| # | Capability | How demonstrated |
|---|-----------|-----------------|
| 1 |           |                 |
| 2 |           |                 |

#### 2. Success metrics (measurable, time-bound)
- [ ] <metric 1>
- [ ] <metric 2>

#### 3. Milestones (3–5)
Each milestone must name at least one linked training to spawn.
- **M1 (weeks 1–N):** <objective>
  - Training to spawn: <topic>
  - Checkpoint activity:
- **M2:** …
- **M3:** …

#### 4. Mentor recommendation
- <@handle or "self-directed + rationale">

#### 5. Checkpoint cadence
- Every <N> weeks with <role>
- Artifacts the learner brings to each checkpoint:

#### 6. Risk register
- <risk → mitigation>
- <risk → mitigation>

#### 7. Out of scope
-

#### 8. Open questions for the human Reviewer
- <or "None.">
```

---

## Why these fields exist

Each field maps to a real failure mode observed in past plans:

| Field | Failure mode it prevents |
|---|---|
| Concrete deployment target | Vague "replace with your deploy command" hand-waving |
| Observable outcomes with demo column | Unfalsifiable outcomes like "understands X" |
| Artifact discipline | Teaching redeploy-by-rebuild as if it were rollback |
| Rollback stateful concerns | Treating rollback as stateless when it rarely is |
| Failure realism (ambiguous scenario) | Only pre-labeled, single-cause lab failures |
| Raw-log interpretation task | Superficial "log-reading checklist" without real logs |
| Secrets anti-patterns | Embedding hardcoded tokens, over-scoped PATs early |
| Observability ceiling | Overpromising; reveals scope honestly |
| Definition of Done (explain a decision) | "It deployed" as the only pass criterion |
| Out of scope | Time-budget overrun; scope creep |
| Risk register (PDP) | Silent goal collapse when life happens |

---

## How the form is validated

The [Critic Agent](../.github/agents/critic-agent.md) scores every draft
against the rubric in [.github/agents/plan-rubric.md](../.github/agents/plan-rubric.md).
A plan cannot reach a human reviewer until the critic gives a **PASS**
verdict. Expected flow: 1–3 critic rounds per plan.
