# Roles

| Role | Responsibilities | Gate |
| --- | --- | --- |
| **Requestor** | Files the request, answers intake questions, takes the training, submits feedback. | Closes the loop (feedback). |
| **AI Agent** (Copilot / Actions) | Intake Q&A, plan drafting, content drafting (PR), catalog indexing, label transitions. | None — always human-reviewed. |
| **Reviewer** (training lead) | Approves or revises the training *plan*. Assigns an SME. | Gate #1 — plan approval. |
| **SME** (subject matter expert) | Verifies the built *content* is accurate and complete. | Gate #2 — content verification. |
| **Maintainer** | Merges PRs, runs label/index automation, closes requests, tags catalog releases. | Custodial. |

## Assignment

- **Reviewer** is assigned via `CODEOWNERS` on `/trainings/**` plan comments
  (or a rotating on-call in the repo README).
- **SME** is proposed by the AI in the plan, confirmed by the Reviewer, and
  added as an assignee on the PR.

## Approval conventions

- Plan approval: Reviewer comments `/approve-plan` on the request issue, or
  applies label `state:plan-approved` directly.
- Content approval: standard GitHub PR review by the assigned SME.
