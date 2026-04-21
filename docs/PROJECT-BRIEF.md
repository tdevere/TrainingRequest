# Project Brief — for AI agents

> A machine-readable summary of this repository's purpose, structure, and
> operating rules. Another AI can ingest this file alone to understand what
> the repo does and how to act inside it.

---

## 1. Identity

- **Name:** TrainingRequest
- **Owner:** `@tdevere`
- **Repo:** https://github.com/tdevere/TrainingRequest
- **Visibility:** public
- **Primary language:** Markdown (GitHub-flavored) + GitHub Actions YAML
- **License:** not yet set
- **Status:** active

## 2. Purpose (one paragraph)

A coordination system for requesting, designing, building, delivering, and
cataloguing internal learning assets. It supports two request kinds:
**Training Requests** (single, scoped trainings) and **Personal Development
Plans / PDPs** (longer-horizon growth roadmaps that may spawn multiple
Training Requests). The system pairs an AI intake agent with two explicit
human approval gates so AI can accelerate drafting while humans retain
control over plan approval and content verification.

## 3. Core features

1. **Structured intake forms** (GitHub Issue Forms) for training and PDP
   requests, plus a feedback form.
2. **AI intake agent** with a committed system prompt that clarifies asks
   and drafts plans.
3. **Labeled state machine** that every request traverses:
   `new → intake → plan-drafted → plan-approved → content-building →
   content-verified → delivered → feedback-collected → complete` (with
   `iterating` and `retired` as alternates).
4. **Two human gates:** Reviewer approves the *plan*; SME verifies the
   *built content*.
5. **Markdown asset catalog** under `/trainings/` and `/pdps/` with YAML
   front-matter.
6. **Auto-generated indexes** (`trainings/INDEX.md`, `pdps/INDEX.md`) built
   by a GitHub Actions workflow on every push to `main`.
7. **Structured feedback** with a verdict field (`finalize | iterate |
   retire`) that deterministically drives the next state.
8. **Definition of Done** tied to artifacts (file merged, index updated,
   feedback logged, issue closed) rather than vibes.

## 4. Roles

| Role | Responsibility | Human? |
| --- | --- | --- |
| Requestor | Files request, takes training, gives feedback | Yes |
| AI Agent (Copilot coding agent) | Intake, plan drafting, content drafting, catalog indexing | No |
| Reviewer | Approves the training/PDP plan (Gate #1) | Yes |
| SME | Verifies built content for accuracy (Gate #2) | Yes |
| Maintainer (`@tdevere`) | Merges, closes requests, runs setup | Yes |

## 5. Repo map (authoritative)

```
README.md                                     # overview
CONTRIBUTING.md                               # how to contribute
CODE_OF_CONDUCT.md                            # behavior rules
.github/
  copilot-instructions.md                     # auto-loaded rules for Copilot
  CODEOWNERS                                  # @tdevere owns everything
  pull_request_template.md
  labels.yml                                  # canonical label set
  agents/
    intake-agent.md                           # SYSTEM PROMPT for intake
  ISSUE_TEMPLATE/
    config.yml
    training-request.yml                      # training intake form
    personal-development-plan.yml             # PDP intake form
    training-feedback.yml                     # feedback form
  workflows/
    training-intake.yml                       # posts welcome + advances labels
    index-trainings.yml                       # rebuilds both INDEX.md files
    sync-labels.yml                           # syncs labels.yml -> repo labels
docs/
  WORKFLOW.md                                 # training state machine
  PDP-WORKFLOW.md                             # PDP state machine (same states)
  ROLES.md                                    # roles + approval conventions
  MAINTAINERS.md                              # one-time + recurring tasks
trainings/
  _template.md                                # template for a new training
  INDEX.md                                    # AUTO-GENERATED catalog
pdps/
  _template.md                                # template for a new PDP
  INDEX.md                                    # AUTO-GENERATED catalog
```

## 6. Label schema (used by automation)

- **Kinds:** `training-request`, `pdp-request`, `training-feedback`
- **States:** `state:new`, `state:intake`, `state:plan-drafted`,
  `state:plan-approved`, `state:content-building`,
  `state:content-verified`, `state:delivered`,
  `state:feedback-collected`, `state:iterating`, `state:complete`,
  `state:retired`
- **Metadata:** `level:beginner|intermediate|advanced`,
  `needs:reviewer`, `needs:sme`, `needs:requestor`

State labels are mutually exclusive within a single issue at any moment.

## 7. Operating rules for an AI agent

Read these before taking any action in the repo.

1. **Detect context first.** Read issue labels to know (a) kind and (b)
   current state. Never act on state alone; kind determines the template.
2. **Never self-approve a human gate.** Do not apply `state:plan-approved`,
   `state:content-verified`, or `state:complete`. Those transitions are
   reserved for humans.
3. **Use the intake system prompt.** When an issue is in `state:intake`,
   follow [`.github/agents/intake-agent.md`](../.github/agents/intake-agent.md)
   verbatim. It defines when to ask questions, how many (max 5, one
   consolidated comment), and the plan comment templates.
4. **Check the catalog for overlap** before drafting a new plan. If a close
   match exists in `trainings/INDEX.md` or `pdps/INDEX.md`, recommend it
   first.
5. **Build in a branch, never on `main`.** Use `training/<slug>` or
   `pdp/<slug>`. Open a PR. Set `request_issue:` in front-matter.
6. **Copy templates — don't edit them.** `_template.md` files are
   read-only from the agent's perspective.
7. **Don't edit `INDEX.md` files.** The indexing workflow owns them.
8. **No hallucinations.** Unknown SMEs/links/handles → `tbd`. Cite sources
   in the `References` section.
9. **Outcomes must be observable.** Prefer "can deploy X" over
   "understands X".
10. **Stop at gates.** After posting a plan, set
    `state:plan-drafted` + `needs:reviewer` and stop. After opening a
    content PR, set `state:content-building` + `needs:sme` and stop.

## 8. State transitions (who can make them)

| From | To | Trigger | Authorized by |
| --- | --- | --- | --- |
| `new` | `intake` | New issue with kind label | Automation |
| `intake` | `plan-drafted` | Plan comment posted | AI agent |
| `plan-drafted` | `plan-approved` | `/approve-plan` or label | Reviewer |
| `plan-approved` | `content-building` | PR opened with asset file | AI agent |
| `content-building` | `content-verified` | PR approved by SME | SME |
| `content-verified` | `delivered` | PR merged | Maintainer |
| `delivered` | `feedback-collected` | Feedback issue opened | Requestor |
| `feedback-collected` | `complete` | Verdict = Finalize | Reviewer/Maintainer |
| `feedback-collected` | `iterating` | Verdict = Iterate | Reviewer |
| `feedback-collected` | `retired` | Verdict = Retire | Reviewer |
| `iterating` | `content-building` | New revision PR | AI agent |

## 9. Artifacts schema

### Training asset (`trainings/<slug>.md`)

Required YAML front-matter keys:
`title`, `slug`, `request_issue`, `level`, `format`, `duration`,
`owner_sme`, `status`, `created`, `last_updated`.

### PDP (`pdps/<slug>.md`)

Required YAML front-matter keys:
`title`, `slug`, `request_issue`, `owner`, `reviewer`, `mentor`,
`horizon`, `start_date`, `target_date`, `status`, `created`,
`last_updated`.

Both are parsed by `.github/workflows/index-trainings.yml` to build catalog
tables. Keep keys stable; changing them requires a workflow update.

## 10. Definition of Done

A request reaches `state:complete` only when **all** are true:

- Asset file exists under `/trainings/` or `/pdps/` with `status: published`.
- The matching catalog `INDEX.md` lists it.
- A feedback issue exists with verdict = Finalize and is linked.
- The original request issue is closed with `state:complete`.

## 11. Non-goals

- Not an LMS. No SCORM, no completion tracking beyond feedback issues.
- Not a credentialing system. No certificates or scoring.
- Not a content hosting platform for video/binary. Link out to those.
- Not a substitute for performance management. PDPs are voluntary growth
  plans, not performance reviews.

## 12. Entry points for another AI

- **To understand the process:** start with this file, then
  [`docs/WORKFLOW.md`](WORKFLOW.md) and [`docs/PDP-WORKFLOW.md`](PDP-WORKFLOW.md).
- **To act as the intake agent:** load
  [`.github/agents/intake-agent.md`](../.github/agents/intake-agent.md) as
  the system prompt and the triggering issue as user input.
- **To build a training asset:** load
  [`trainings/_template.md`](../trainings/_template.md) and the approved
  plan comment from the issue thread.
- **To build a PDP asset:** load
  [`pdps/_template.md`](../pdps/_template.md) and the approved roadmap
  comment from the issue thread.
- **To answer "does this already exist?":** read
  [`trainings/INDEX.md`](../trainings/INDEX.md) and
  [`pdps/INDEX.md`](../pdps/INDEX.md).

## 13. Known limits / open items

- No automated SME assignment yet — Reviewer proposes, assigns manually.
- No notification to requestor on delivery beyond the GitHub issue/PR
  link. An email/Slack bridge is out of scope for now.
- No license file. If the repo is used outside this org, add one.
- No CONTRIBUTING enforcement in CI (e.g. front-matter validation).
  Candidate follow-up: a Pylance/yamllint step on PRs touching
  `trainings/*.md` or `pdps/*.md`.
