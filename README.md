# Training & Development Request System

An AI-assisted, human-reviewed workflow for requesting, building, delivering,
and capturing internal **trainings** and **personal development plans (PDPs)**
as reusable assets.

## What this repo does

Two request types, one workflow:

- **Training Request** — a single, scoped training on a topic.
- **Personal Development Plan (PDP)** — a longer-horizon roadmap (3–12 months)
  that typically spawns multiple linked Training Requests.

For both:

1. Anyone files a GitHub Issue using the matching structured form.
2. An **AI intake agent** asks clarifying questions and drafts a plan / roadmap.
3. A **human Reviewer** approves (or edits) the plan.
4. The AI builds the asset (training content, or PDP file with linked sub-items).
5. A **human SME / Reviewer** verifies the content.
6. The asset is delivered to the requestor.
7. Requestor takes the training / works the plan and submits structured feedback.
8. Based on feedback, the asset is **finalized**, **iterated**, or **retired**.

## Repo layout

| Path | Purpose |
| --- | --- |
| `.github/ISSUE_TEMPLATE/training-request.yml` | Structured intake form for a single training |
| `.github/ISSUE_TEMPLATE/personal-development-plan.yml` | Structured intake form for a PDP |
| `.github/ISSUE_TEMPLATE/training-feedback.yml` | Structured feedback form |
| `.github/agents/intake-agent.md` | System prompt for the AI intake agent |
| `.github/workflows/` | Automation for intake, labeling, catalog indexing |
| `.github/labels.yml` | Canonical label set (states + metadata) |
| `docs/WORKFLOW.md` | Training workflow (states + roles) |
| `docs/PDP-WORKFLOW.md` | PDP workflow (states + roles) |
| `docs/ROLES.md` | Who does what |
| `trainings/` | Completed training assets |
| `trainings/_template.md` | Template for a new training |
| `trainings/INDEX.md` | Auto-generated training catalog |
| `pdps/` | Active and completed PDPs |
| `pdps/_template.md` | Template for a new PDP |
| `pdps/INDEX.md` | Auto-generated PDP catalog |

## Quick start

- **Request a single training**: [open a Training Request](../../issues/new?template=training-request.yml).
- **Request a development plan**: [open a PDP](../../issues/new?template=personal-development-plan.yml).
- **Browse existing trainings**: [trainings/INDEX.md](trainings/INDEX.md).
- **Browse existing PDPs**: [pdps/INDEX.md](pdps/INDEX.md).
- **Understand the process**: [docs/WORKFLOW.md](docs/WORKFLOW.md), [docs/PDP-WORKFLOW.md](docs/PDP-WORKFLOW.md).

## Roles

See [docs/ROLES.md](docs/ROLES.md). Short version:

- **Requestor** — files the request, takes the training, gives feedback.
- **AI Agent** — intake, plan drafting, content drafting, catalog indexing.
- **Reviewer** — approves the training plan.
- **SME** — verifies built content for accuracy.
- **Maintainer** — merges, tags releases, closes the loop.
