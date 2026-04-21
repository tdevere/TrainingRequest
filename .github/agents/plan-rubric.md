# Plan Rubric

> Scoring rubric applied by the [Critic Agent](critic-agent.md) to every
> drafted plan before it reaches a human reviewer. Each dimension is
> scored `PASS`, `WEAK`, or `FAIL`. A plan reaches `state:plan-approved`
> only after the critic gives overall `PASS` **and** a human reviewer
> confirms.

## Scoring dimensions

For each dimension, list the exact evidence from the plan that supports
the score. No evidence ⇒ automatic `WEAK` or `FAIL`.

### D1 — Outcome verifiability
- `PASS`: Every outcome starts with an externally verifiable verb
  (configure, deploy, roll back, diagnose, explain-and-defend). Each has
  a "how it's demonstrated" column filled in.
- `WEAK`: One outcome uses an internal verb ("understand", "know",
  "be familiar with") OR one demonstration column is vague.
- `FAIL`: Two or more unverifiable outcomes OR any missing demo column.

### D2 — Concrete deployment target (training only)
- `PASS`: A specific platform is named and justified in one line.
- `WEAK`: Platform named but no justification, or marked `n/a` without
  a strong reason.
- `FAIL`: "Replace with your deploy command" or similarly abstract.

### D3 — Artifact discipline
- `PASS`: Plan explicitly teaches build-once, deploy-same-artifact with
  a storage location and an immutable version identifier.
- `WEAK`: Mentions artifacts but uses commit SHA for deploy without
  discussing the trade-off.
- `FAIL`: Rebuild-to-redeploy is presented as rollback.

### D4 — Rollback depth
- `PASS`: Names at least one stateful concern (schema, config, feature
  flags, compatibility window) even if only to scope it out explicitly.
- `WEAK`: Stateless rollback only, but the plan marks stateful concerns
  `out of scope` clearly.
- `FAIL`: Implies rollback is always safe / stateless.

### D5 — Failure realism
- `PASS`: At least one lab scenario is multi-layer OR ambiguous
  (e.g., "intermittent 502s after a recent env-var change, logs show
  DNS + auth hints — diagnose").
- `WEAK`: All scenarios are single-cause but one requires the learner
  to differentiate between similar failure modes.
- `FAIL`: All failures are pre-labeled, single-cause, happy-path breaks.

### D6 — Raw-log interpretation
- `PASS`: Lab includes at least one task with actual log snippets
  (or a link to them) and no inline hints as to the cause.
- `WEAK`: Logs referenced but not shown.
- `FAIL`: No log-reading task, or logs appear only as illustrative
  screenshots with captions that give away the answer.

### D7 — Secrets / security hygiene
- `PASS`: Names a specific secrets mechanism AND calls out at least two
  anti-patterns AND mentions least privilege / token scoping.
- `WEAK`: Covers mechanism but skips anti-patterns or scoping.
- `FAIL`: Secrets are used in examples without commentary.

### D8 — Health validation
- `PASS`: Defines "healthy" beyond HTTP 200 and names an observability
  ceiling (what the training explicitly does NOT teach).
- `WEAK`: "Smoke test" mentioned without definition.
- `FAIL`: No health validation step.

### D9 — Definition of Done
- `PASS`: Includes at least one "explain a decision" item alongside
  task-based items.
- `WEAK`: All items are task-completion; no reflection.
- `FAIL`: Done = "it deployed" or similar single criterion.

### D10 — Scope honesty
- `PASS`: "Out of scope" section is non-empty and specific.
- `WEAK`: Out-of-scope present but generic ("advanced topics").
- `FAIL`: No out-of-scope section, OR the outcomes exceed the stated
  learner time budget.

### D11 — Time budget feasibility
- `PASS`: Plan fits within the stated learner time with a reasonable
  buffer (≤90% packed). A rough per-section minute budget is given.
- `WEAK`: Time budget implied but not broken down.
- `FAIL`: Plan clearly cannot be completed in the stated time.

### D12 — Prerequisites & level match
- `PASS`: Prerequisites named; assumed-not-required listed so learners
  can self-filter. Level matches depth.
- `WEAK`: Prerequisites present but no self-filter list.
- `FAIL`: Plan targets the wrong level (e.g. advanced concepts in a
  beginner plan, or toy exercises in an advanced one).

---

## PDP-specific dimensions

### D13 — Capability observability (PDP)
Same standard as D1, applied to target capabilities.

### D14 — Metrics measurability (PDP)
- `PASS`: Every success metric is time-bound and externally measurable.
- `WEAK`: Measurable but not time-bound.
- `FAIL`: "Feel confident", "become comfortable", etc.

### D15 — Milestone → training linkage (PDP)
- `PASS`: Every milestone names at least one concrete training topic
  to spawn as a sub-request.
- `WEAK`: Generic skill area named without a spawnable topic.
- `FAIL`: No linkage; PDP floats without downstream trainings.

### D16 — Risk register (PDP)
- `PASS`: ≥2 realistic risks with mitigations.
- `WEAK`: 1 risk or mitigations that are aspirational.
- `FAIL`: No risk register or all mitigations = "try harder".

---

## Overall verdict

- **PASS** — zero `FAIL` dimensions AND ≤2 `WEAK` dimensions. Plan may
  proceed to human reviewer. The critic must still list every `WEAK`
  finding so the reviewer sees the residual risk.
- **REVISE** — any other combination. Critic posts revision requests;
  the drafter revises; re-run the rubric.

## Upper bound

After **3 critic rounds**, if still `REVISE`, escalate to the human
reviewer with the full critic history attached and a flag that this
plan may need human redesign rather than further AI iteration.
