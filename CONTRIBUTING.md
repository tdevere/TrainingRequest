# Contributing

Thanks for your interest in contributing. This repo coordinates training
requests and personal development plans (PDPs). Most "contribution" happens
through issues, not code — but process changes are welcome too.

## Ways to contribute

### 1. File a request
- **Training Request** — a single, scoped training you want built.
  [New issue](../../issues/new?template=training-request.yml)
- **Personal Development Plan (PDP)** — a longer-horizon growth roadmap.
  [New issue](../../issues/new?template=personal-development-plan.yml)

### 2. Give feedback on a training you took
- Use the [Training Feedback](../../issues/new?template=training-feedback.yml)
  form. Link back to the original training request and the asset file.

### 3. Review a plan or verify content
- If you're assigned as a **Reviewer**, approve the plan by commenting
  `/approve-plan` or applying the `state:plan-approved` label.
- If you're assigned as an **SME**, review the PR that adds the training
  asset. Approve the PR to signal content verification.

### 4. Author or improve a training asset
1. Claim a request issue labeled `state:plan-approved`.
2. Branch: `training/<slug>`.
3. Copy [trainings/_template.md](trainings/_template.md) → `trainings/<slug>.md`.
4. Fill in front-matter. Set `request_issue:` to the request issue number.
5. Open a PR; request review from the assigned SME.
6. On merge, the catalog index refreshes automatically.

### 5. Improve the workflow itself
- Process changes land in `docs/` and `.github/`.
- Open a PR; `@tdevere` reviews.

## Conventions

- **Branches**: `training/<slug>`, `pdp/<slug>`, `docs/<topic>`, `ci/<topic>`.
- **Commits**: conventional-ish (`feat:`, `fix:`, `docs:`, `chore:`, `ci:`).
- **Labels**: set by automation; don't fight the state machine. If a state
  transition is wrong, say so in a comment and a maintainer will adjust.
- **PR size**: one training per PR. One doc change per PR.

## Code of Conduct

Be kind. Assume good intent. This repo is about helping people learn.
Abusive behavior in issues, PRs, or feedback forms is not tolerated and
will be removed.

## Questions

- Workflow questions → see [docs/WORKFLOW.md](docs/WORKFLOW.md) and
  [docs/PDP-WORKFLOW.md](docs/PDP-WORKFLOW.md).
- Roles → see [docs/ROLES.md](docs/ROLES.md).
- Maintainer setup → see [docs/MAINTAINERS.md](docs/MAINTAINERS.md).
- Anything else → open an issue and label it `question`.
