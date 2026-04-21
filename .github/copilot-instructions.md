# Copilot instructions for this repo

This file is read automatically by GitHub Copilot (Chat and the coding agent)
when operating against this repo. Keep it short and action-oriented.

## What this repo is

A coordination repo for **Training Requests** and **Personal Development
Plans (PDPs)**. Most work happens in issues; assets live as markdown under
`/trainings/` and `/pdps/`.

## When acting on an issue

1. Check the issue's labels to determine kind and state:
   - `training-request` vs `pdp-request`
   - `state:*` labels drive the workflow; read them before acting.
2. If the issue is in `state:intake` **or** `state:plan-revising`, follow
   [.github/agents/intake-agent.md](.github/agents/intake-agent.md). That
   file is your system prompt as the Intake Agent. In revision rounds,
   address the numbered items from the latest Critic Agent comment and
   re-post the full Plan Acceptance Form.
3. If the issue is in `state:ai-reviewing`, follow
   [.github/agents/critic-agent.md](.github/agents/critic-agent.md) as
   the Critic Agent. Score against
   [.github/agents/plan-rubric.md](.github/agents/plan-rubric.md) with
   evidence, and either PASS or REVISE.
4. Every draft plan must copy the full **Plan Acceptance Form** from
   [docs/PLAN-ACCEPTANCE-FORM.md](docs/PLAN-ACCEPTANCE-FORM.md). Blank
   fields will be rejected by the Critic Agent.
5. If the issue is in `state:plan-approved` and you are asked to build the
   asset:
   - Branch: `training/<slug>` or `pdp/<slug>`.
   - Use [trainings/_template.md](trainings/_template.md) or
     [pdps/_template.md](pdps/_template.md). Copy, don't edit in place.
   - Set `request_issue:` in front-matter to the issue number.
   - Open a PR and request review from the assigned SME.
6. Never move an issue to `state:plan-approved`, `state:content-verified`,
   or `state:complete` yourself. Those are human gates.

## Content quality bar

- Learning outcomes must be **observable** ("can deploy X") not internal
  ("understands X").
- Include at least one hands-on element when the requestor asked for `Hands-on`.
- Cite sources in the **References** section.
- No hallucinated SMEs, handles, or links. Use `tbd` if unknown.

## Conventions

- Markdown, 2-space indent in YAML, LF line endings.
- PR titles: `feat(training): <slug>` or `feat(pdp): <slug>`.
- Commit messages: conventional-ish (`feat:`, `fix:`, `docs:`, `ci:`).
- Don't modify `trainings/INDEX.md` or `pdps/INDEX.md` by hand — the index
  workflow regenerates them.

## When in doubt

Ask in an issue comment, don't guess. A human reviewer or SME can unblock.
