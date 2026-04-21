# Testing Mode

While the repo has only one maintainer / reviewer (`@tdevere`), the normal
rule "the Reviewer cannot be the Requestor" cannot be honored. This doc
defines a **sanctioned bypass** for bootstrap and dogfooding — and the
exact moment that bypass must end.

## When test mode is allowed

- During initial workflow validation.
- When adding or refactoring an issue template, workflow, or agent prompt
  and you need to exercise the full state machine.
- When seeding example content so new contributors can see a worked
  example in each catalog.

Test mode is **not** allowed for any training or PDP that will be presented
to a real learner as production content. Those must follow the normal
two-human-gate flow.

## How to activate test mode on an issue

1. File the request normally (Training Request or PDP).
2. Immediately add two labels:
   - `test-mode`
   - `persona:fixture`
3. Add a top comment that starts with `TEST MODE:` and names the persona
   you are role-playing (see [docs/TEST-PERSONAS.md](TEST-PERSONAS.md)).

## Rules under test mode

1. **One human can play multiple roles**, but each role must be declared
   explicitly in a comment:
   - `ROLE: Reviewer — approving plan` when applying `state:plan-approved`.
   - `ROLE: SME — verifying content` when approving the content PR.
   - `ROLE: Requestor — feedback` when filing the feedback issue.
2. **Gate labels still require a comment** declaring the role. Don't just
   flip a label silently; the audit trail is the point.
3. **The AI agent behavior does not change.** Copilot still runs intake,
   drafts plans, opens PRs, and stops at gates. Only the human side is
   relaxed.
4. Test-mode assets live in the catalog like any other training/PDP, but
   carry `status: fixture` in front-matter (not `published`) and are
   excluded from any "recommend to a learner" flow.
5. Test-mode issues and assets must be labeled `test-mode` so they can be
   filtered or cleaned up in bulk.

## Exiting test mode

Once a second trusted reviewer is onboarded:

1. Update [.github/CODEOWNERS](../.github/CODEOWNERS) to list both.
2. Update [docs/ROLES.md](ROLES.md) and the "Gate #1" assignee rotation.
3. New requests no longer use `test-mode`.
4. Existing fixture assets can stay (they're useful as examples) but do
   not need to be re-reviewed.

## Cleanup query

```text
is:issue label:test-mode
is:pr label:test-mode
```

These filter to every fixture for easy triage or bulk close when you're
done testing.
