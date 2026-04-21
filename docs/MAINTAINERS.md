# Maintainer Setup

One-time and recurring tasks for repo maintainers (currently: `@tdevere`).

## One-time setup (do this once after first push)

### 1. Enable the GitHub Copilot coding agent

The intake workflow @-mentions `@github-copilot` on new issues. For Copilot
to actually pick up and act on those mentions, the coding agent must be
enabled for this repo.

Steps:

1. Navigate to **Settings → Copilot → Coding agent** on this repo:
   https://github.com/tdevere/TrainingRequest/settings/copilot/coding_agent
2. Toggle **Enable Copilot coding agent** for the repository.
3. (Optional) Configure which branches the agent is allowed to open PRs
   against. Recommended: allow `main`.
4. Verify under **Settings → Actions → General** that:
   - **Workflow permissions** = "Read and write permissions".
   - **Allow GitHub Actions to create and approve pull requests** = enabled.
     (Required so the coding agent can open PRs for training assets.)

Requirements:
- The account enabling this must have a Copilot Pro, Copilot Business, or
  Copilot Enterprise plan with coding agent access.
- The repo owner must have accepted the Copilot coding agent preview terms
  once at the account level.

If the coding agent is unavailable, the workflow still works — a human
reviewer can paste [.github/agents/intake-agent.md](../.github/agents/intake-agent.md)
into a Copilot Chat session against the issue to drive intake manually.

### 2. Run label sync once

If you didn't already:

```pwsh
gh workflow run sync-labels.yml --ref main --repo tdevere/TrainingRequest
```

Verify labels in the UI: https://github.com/tdevere/TrainingRequest/labels

### 3. Protect `main`

Recommended branch protection on `main`:

- Require PR review before merging (1 approval).
- Require status checks to pass: the indexing workflow is non-blocking and
  commits back to main itself, so exclude it from required checks if you
  add stricter rules.
- Require conversation resolution before merging.
- Disallow force pushes.

Quick setup:

```pwsh
gh api -X PUT repos/tdevere/TrainingRequest/branches/main/protection `
  -f required_pull_request_reviews.required_approving_review_count=1 `
  -F required_status_checks= `
  -F enforce_admins=false `
  -F restrictions= `
  -F required_conversation_resolution=true
```

Or use the UI: Settings → Branches → Add rule.

### 4. Enable Discussions (optional)

If you want a place for general Q&A that isn't a training request:

```pwsh
gh repo edit tdevere/TrainingRequest --enable-discussions
```

## Recurring tasks

### Triage queue

Query for things needing your attention:

- **Plans awaiting review**: `is:open label:"state:plan-drafted" label:"needs:reviewer"`
  https://github.com/tdevere/TrainingRequest/issues?q=is%3Aopen+label%3A%22state%3Aplan-drafted%22
- **Content awaiting SME**: `is:open is:pr label:"state:content-building" label:"needs:sme"`
- **Feedback collected**: `is:open label:"state:feedback-collected"`
- **Stuck in intake > 5 days**: `is:open label:"state:intake" updated:<YYYY-MM-DD`

### Closing the loop

When a training request hits `state:complete`:

1. Confirm the asset exists under `/trainings/<slug>.md` with `status: published`.
2. Confirm a feedback issue with verdict = Finalize exists and is linked.
3. Close the request issue.
4. The catalog index refreshes automatically on the next push.

### Releases (optional)

If you want to tag catalog snapshots:

```pwsh
git tag -a catalog-2026.Q2 -m "Catalog snapshot 2026 Q2"
git push origin catalog-2026.Q2
```

## Rotation

If maintainership rotates, update:

- [.github/CODEOWNERS](../.github/CODEOWNERS)
- [README.md](../README.md) ("Roles" section)
- The default assignee in [.github/workflows/training-intake.yml](../.github/workflows/training-intake.yml)
  (look for `assignees: ['tdevere']`).
