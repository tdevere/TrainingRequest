# Operator Guide — How to drive a request through the workflow

> If you ever wonder "what do I do next?" — this is the file. Bookmark it.

There are exactly **five human decision points** in the lifecycle of a
training request or PDP. Everything else is automated. This guide tells you
how to find what's pending and how to advance each stage.

---

## How to see what's waiting on you (the dashboard)

Open these saved searches — bookmark them.

### "Things waiting on a human"

| What | Search link |
| --- | --- |
| Plans waiting for Reviewer approval | [`is:open label:"state:plan-drafted"`](https://github.com/tdevere/TrainingRequest/issues?q=is%3Aopen+label%3A%22state%3Aplan-drafted%22) |
| Content PRs waiting for SME verification | [`is:open is:pr label:"state:content-building"`](https://github.com/tdevere/TrainingRequest/pulls?q=is%3Aopen+is%3Apr+label%3A%22state%3Acontent-building%22) |
| Delivered, awaiting feedback | [`is:open label:"state:delivered"`](https://github.com/tdevere/TrainingRequest/issues?q=is%3Aopen+label%3A%22state%3Adelivered%22) |
| Feedback collected, awaiting verdict | [`is:open label:"state:feedback-collected"`](https://github.com/tdevere/TrainingRequest/issues?q=is%3Aopen+label%3A%22state%3Afeedback-collected%22) |
| Stuck in intake (AI not picking up) | [`is:open label:"state:intake"`](https://github.com/tdevere/TrainingRequest/issues?q=is%3Aopen+label%3A%22state%3Aintake%22) |

### Quick CLI version

```pwsh
gh issue list --repo tdevere/TrainingRequest --state open --json number,title,labels |
  ConvertFrom-Json |
  ForEach-Object {
    $st = ($_.labels.name | Where-Object { $_ -like 'state:*' }) -join ','
    "#$($_.number) [$st] - $($_.title)"
  }
```

If you see `state:plan-drafted` → you owe a plan approval.
If you see `state:feedback-collected` → you owe a verdict.
If you see only `state:intake` for a long time → the AI hasn't started; see
the "Stage 1" section below.

---

## The 5 human decision points

### Stage 1 — Kick off the AI intake agent

**When:** A new issue is in `state:intake` and nothing is happening.

**Why this step exists:** The intake workflow only @-mentions Copilot. The
Copilot coding agent typically only starts work when **assigned** to the
issue.

**Do this:**

1. Open the issue in the browser.
2. In the right sidebar → **Assignees** → click the gear icon.
3. Type `Copilot` and select the **GitHub Copilot** bot account.
4. Done. Copilot will read [.github/agents/intake-agent.md](../.github/agents/intake-agent.md)
   and either ask clarifying questions or post a draft plan.

> Tip: You can also just type `@github-copilot` as a fresh comment asking
> it to draft the plan — but assigning it is the canonical way and gives
> you a "Copilot session" panel you can monitor.

---

### Stage 2 — Approve (or revise) the training plan ← **Reviewer gate**

**When:** Issue is `state:plan-drafted` + `needs:reviewer`. A **Critic
Agent has already scored the plan** and posted a `🟢 Critic Agent — PASS`
or `🚨 ESCALATION` comment directly above. Read that comment first — it
tells you which dimensions are weak and what residual risks remain.

> While the issue is in `state:ai-reviewing` or `state:plan-revising`,
> **don't act**. The AI critic loop is running. It caps at 3 rounds and
> will hand the issue to you automatically with `needs:reviewer` when it
> either PASSes or escalates.

**What to check** before approving:
- The Critic Agent's evidence table — do the scores look fair?
- Any `WEAK` dimensions the critic flagged — acceptable, or block merge?
- Are the learning outcomes observable and aligned with the request?
- Does the Plan Acceptance Form fit the stated learner time budget?
- Does the proposed SME make sense? (Use `tbd` if no one fits.)
- Any "Open questions for reviewer" the agent flagged?

**Do this (test mode, since you're solo for now):**

1. Comment on the issue:
   ```
   ROLE: Reviewer — approving plan (test mode).
   Approved as drafted.
   ```
   (Or: "Approved with edits: <list>" and edit the plan in the comment thread.)
2. Apply label `state:plan-approved`. Remove `state:plan-drafted` and `needs:reviewer`.
3. The agent will pick this up and open a PR adding the asset file.

**Do this (normal mode, with two reviewers):**

Same as above, but the Reviewer cannot be the Requestor.

---

### Stage 3 — Verify the built content ← **SME gate**

**When:** A PR is open with title `feat(training): <slug>` or
`feat(pdp): <slug>`, labeled `state:content-building` + `needs:sme`.

**What to check** in the PR diff:
- Front-matter complete? (`title`, `slug`, `request_issue`, `level`,
  `format`, `duration`, `owner_sme`, `status`, `created`, `last_updated`)
- `request_issue:` matches the originating issue number?
- Learning outcomes match the approved plan exactly?
- For each outcome, there's actually content covering it?
- Code samples / commands run as written?
- References cited (no hand-wavey "see the docs")?
- No hallucinated SMEs, handles, links, or tools?
- If `Hands-on lab` was requested, there's a real exercise (not just
  "now try it yourself")?

**Do this (test mode):**

1. PR review comment:
   ```
   ROLE: SME — verifying content (test mode).
   Verified. Ready to merge.
   ```
2. Click **Approve** in the PR review UI.
3. Apply label `state:content-verified`. Remove `state:content-building`
   and `needs:sme`.
4. Merge the PR (squash-merge is fine).
5. Apply `state:delivered` to the issue. Comment on the issue with the
   merged file's path so the requestor has a direct link.

**Do this (normal mode):**

The SME approves the PR. The Maintainer merges and applies `state:delivered`.

---

### Stage 4 — Collect feedback ← **Requestor's turn**

**When:** Issue is `state:delivered`.

**Do this:**

1. Notify the requestor (comment + GitHub mention or external channel)
   that the training is ready, with a link to the merged file.
2. Wait. The requestor takes the training and files a
   [Training Feedback](https://github.com/tdevere/TrainingRequest/issues/new?template=training-feedback.yml)
   issue. That issue links back to the original request and includes a
   verdict: `Finalize | Iterate | Retire`.
3. The original request issue gets `state:feedback-collected` (apply
   manually or wait for a future automation).

---

### Stage 5 — Decide finalize / iterate / retire ← **Reviewer gate #2**

**When:** Issue is `state:feedback-collected` and a feedback issue exists.

**Do this:**

1. Read the feedback issue. Look at the verdict field.
2. Apply one of the following labels to the **request** issue:

| Verdict | Label | Then |
| --- | --- | --- |
| Finalize | `state:complete` | Confirm the asset's front-matter `status:` field is `published`. Close the request issue. The catalog index will refresh. |
| Iterate | `state:iterating` | Open a new branch, edit the existing asset file, bump `last_updated`, open a revision PR. Loop back to Stage 3 (SME verification). Increment an `iteration-N` label if you want. |
| Retire | `state:retired` | Either delete the asset or set its front-matter `status:` to `retired`. Close the request issue. |

---

## "Help, I'm stuck"

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Issue sat in `state:intake` for hours, no comments from Copilot | Coding agent not assigned | Stage 1 — assign `Copilot` as an assignee |
| Coding agent posted but didn't change labels | Agent skipped the label-update step | Apply `state:plan-drafted` + `needs:reviewer` manually |
| You can't find a label | It wasn't synced yet | Run `gh workflow run sync-labels.yml --repo tdevere/TrainingRequest --ref main` |
| PR opened but no `state:content-building` label | Agent forgot to label | Add the label manually; the SME gate still applies |
| You're the only human and feel weird approving your own plan | That's what test mode is for | See [TESTING.md](TESTING.md). Add `ROLE: Reviewer …` and `ROLE: SME …` comments so the audit trail is clear. Recruit a second reviewer when you can. |
| Catalog index didn't update | Index workflow only runs on push to `trainings/**.md` or `pdps/**.md` | Trigger manually: `gh workflow run index-trainings.yml --ref main` |

---

## TL;DR — the 30-second mental model

```
File issue
   ↓
Assign Copilot         ← (you, once)
   ↓
AI drafts plan
   ↓
Approve plan           ← (you, comment + label)
   ↓
AI opens PR
   ↓
Approve PR             ← (you, GitHub PR review + label + merge)
   ↓
Mark delivered         ← (you, label)
   ↓
Requestor files feedback
   ↓
Apply verdict label    ← (you, label + close)
```

Five clicks total per training. Most of them are just labels.
