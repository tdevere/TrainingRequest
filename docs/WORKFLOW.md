# Workflow

## State machine

```mermaid
stateDiagram-v2
    [*] --> New: Requestor files issue
    New --> Intake: AI picks up
    Intake --> PlanDrafted: AI posts plan
    PlanDrafted --> PlanApproved: Reviewer approves
    PlanDrafted --> Intake: Reviewer requests changes
    PlanApproved --> ContentBuilding: AI starts build
    ContentBuilding --> ContentVerified: SME verifies
    ContentBuilding --> ContentBuilding: SME requests fixes
    ContentVerified --> Delivered: Shared with requestor
    Delivered --> FeedbackCollected: Requestor submits feedback
    FeedbackCollected --> Complete: Verdict = finalize
    FeedbackCollected --> Iterating: Verdict = iterate
    FeedbackCollected --> Retired: Verdict = retire
    Iterating --> ContentBuilding: Next revision
    Complete --> [*]
    Retired --> [*]
```

## Stage details

### 1. New — Request filed
- **Who**: Requestor
- **How**: `Training Request` issue form
- **Label applied**: `state:new`

### 2. Intake — AI clarifies
- **Who**: AI Agent (via Copilot / Actions bot)
- **Does**: reads the form, asks 1–5 clarifying questions as an issue comment,
  tags `needs:requestor` if answers are missing.
- **Exit criterion**: enough context to draft a plan.
- **Label**: `state:intake`

### 3. Plan drafted — AI proposes a training plan
- AI posts a plan comment containing:
  - Outline (modules / sections)
  - Learning objectives mapped to requested outcomes
  - Format(s) chosen
  - Estimated effort to build
  - Prerequisites
  - Proposed SME (if known)
- **Label**: `state:plan-drafted`, `needs:reviewer`

### 4. Plan approved — Human reviewer gate #1
- **Who**: Reviewer (training lead)
- **Does**: approves via comment `/approve-plan` or edits and re-approves.
- **Label**: `state:plan-approved`

### 5. Content building — AI drafts the asset
- AI produces a PR that adds a file under `/trainings/` using
  `trainings/_template.md`. Front-matter links back to the request issue.
- **Label**: `state:content-building`, `needs:sme`

### 6. Content verified — Human SME gate #2
- **Who**: SME
- **Does**: reviews the PR for accuracy, approves or requests changes.
- **Exit**: PR approved by SME.
- **Label**: `state:content-verified`

### 7. Delivered
- Maintainer merges the PR to `main`. Automation links the training asset
  back into the request issue and notifies the requestor.
- **Label**: `state:delivered`

### 8. Feedback collected
- Requestor opens a `Training Feedback` issue (linked to the request).
- **Label on request**: `state:feedback-collected`

### 9. Complete / Iterate / Retire
- Based on feedback verdict:
  - **Finalize** → `state:complete`, catalog entry confirmed, request closed.
  - **Iterate** → `state:iterating`, a new PR revises `/trainings/<file>`,
    loop back to stage 5.
  - **Retire** → `state:retired`, file removed or marked archived, request closed.

## Definition of Done

A training is **Complete** only when all are true:
- Training file exists in `/trainings/` and is approved by SME.
- `trainings/INDEX.md` lists it (auto-generated).
- A `Training Feedback` issue exists with verdict = Finalize.
- Original request issue is closed with `state:complete`.
