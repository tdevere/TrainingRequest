# Proposal: Bring the AI-Assisted Training Request system in-house

**To:** Readiness global partners (regional and discipline leads)
**From:** Tony Devere
**Date:** 2026-04-21
**Status:** Draft for feedback
**Repo (working prototype, public):** <https://github.com/tdevere/TrainingRequest>

---

## TL;DR

I built and tested a public prototype of a **personalized training
planner** for our field. It turns a free-form training request into a
structured, peer-reviewable training plan in a few hours, with most of
the quality gating done by two specialized AI agents before anything
hits a human reviewer. I'd like to **pilot it inside Readiness** with
one region and one discipline, measure the before/after, and decide in
the open whether to scale it.

**Ask:** one sponsor, one discipline lead, one SME champion, `[fill N]`
hours of engineering time over a `[fill weeks]` pilot.

---

## Why now

Three things we already know inside Readiness:

- **Training requests scatter.** Today they arrive as emails, Teams
  pings, ServiceNow items, and SharePoint forms — `[fill: estimate of
  monthly request volume across regions]`. There is no single queue and
  no standard intake shape.
- **Plans vary wildly by author.** Two people taking the same intake
  produce training plans with very different verifiability, scope
  honesty, and rollback depth — because there is no shared rubric and
  no peer-review step before build.
- **Personalization is manual.** Tailoring a plan to a specific
  engineer's level and context is currently the SME's job, done in
  their head, rarely captured.

The cost is plans that ship weak, content that overlaps across regions,
and SMEs burned out re-answering the same intake conversations.

## What I built (it's real, not a slide)

A public GitHub repo that runs the full request → plan → asset
workflow on labeled issues, with four stages:

1. **Intake (AI):** turns a requestor issue into a structured
   **Plan Acceptance Form** — 12 required fields including concrete
   deploy target, artifact discipline, rollback treatment, failure
   realism, secrets hygiene, health ceiling, and a Definition of Done
   that requires the learner to *explain a decision*.
2. **Critic (AI):** scores the plan against a 12-dimension rubric with
   evidence; PASS/REVISE/ESCALATE; cap 3 rounds; never self-approves a
   human gate.
3. **Human Reviewer gate:** approves the plan with traceable labels.
4. **SME Build + Verify:** training asset is built from the approved
   form into a template, PR'd, and reviewed before it ships.

End-to-end demonstration run: a beginner "first deployment" training
request went **intake → critic REVISE (10 rubric fails flagged with
numbered fixes) → intake round 2 (full form) → critic PASS → human
approve → asset built → merged.** The full audit trail is in the repo
history.

### What's different from existing training intake

| Existing pattern | This prototype |
|---|---|
| Free-text ticket → SME writes back with questions | Issue form → AI extracts structure automatically |
| Plan review is 1 SME's taste | Plan review is a published rubric scored by an AI critic first, then a human |
| "Understands X" outcomes | Every outcome must have a demo column; "explain a decision" is required |
| Content drifts from intake context | Plan fields are traced into the asset 1-to-1; traceability table required in PR |
| Rollback taught as redeploy | Rubric forces stateful concerns to be named, even if out of scope |
| Scope creep | Mandatory "out of scope" section protects the time budget |

## Pilot proposal

**Scope:** one region × one discipline (my recommendation:
`[fill: e.g. EMEA × DevOps]`). One real backlog of 10–15 training
requests run through the system over `[fill weeks]`.

**Success metrics (set up-front, scored by an external reviewer):**

1. **Time from request → approved plan.** Baseline vs pilot.
2. **Plan quality.** Independent scorer applies the same 12-dimension
   rubric to pilot plans *and* to a matched set of historical plans;
   delta is the signal.
3. **SME hours per training.** Intake + plan revision hours, isolated
   from content-build hours.
4. **Requestor satisfaction.** Short survey, 3 questions, baseline vs
   pilot.
5. **Reuse.** Of the N plans produced, how many were adopted by a
   second requestor without re-authoring.

**Team:** 1 sponsor (signs off on scope + decision rights), 1 discipline
lead (owns the SME bench), 1 engineering partner (me + `[fill]` hours),
2–3 SME volunteers for content verification.

**Decision point at end of pilot:** ship to a second region, scope up
to a platform investment, or stop with a written postmortem. All
artifacts public inside the company.

## Risks and honest caveats

- **The critic is only as good as the rubric.** The current rubric is
  biased toward deployment/operations topics because those were the
  test personas. Rubric needs a second pass from an instructional-design
  SME before production use.
- **Copilot agent latency is per-round.** The current repo requires a
  human to re-kick the agent per round. A production version would
  auto-invoke, which is a modest engineering lift.
- **AI-drafted plans can be fluent but wrong.** The two-stage rubric
  catches structural failures, not factual ones. The human SME gate
  exists for that reason and is non-negotiable.
- **Adoption risk.** Field SMEs already have tools and templates they
  like. The pilot is the cheapest way to learn whether this adds value
  or just adds process.
- **Public repo today, internal repo for pilot.** The prototype lives
  on public GitHub for transparency during the build. Pilot data would
  move to an internal repo; nothing customer-specific is in the public
  prototype.

## Open questions I'd like the group to weigh in on

1. **Where does this live if it's real?** A platform team effort, a
   Readiness-owned tool, or a GitHub-native workflow we tell regions
   to clone?
2. **Relationship to Microsoft Learn / Viva Learning.** Is a published
   training asset a new MS Learn module, a pointer to one, or an
   internal-only artifact that feeds back into MS Learn over time?
3. **Pilot region + discipline.** Where is the pain loudest?

---

## Research to complete before sending

Before this goes to the group, I should fill in:

- [ ] Current monthly request volume (total and by region).
- [ ] Current median time from request → usable training.
- [ ] Average SME hours per training today (build vs intake vs review).
- [ ] Existing intake tools in each region and their pain points.
- [ ] A shortlist of 2–3 candidate pilot regions + disciplines with a
      named sponsor already interested.
- [ ] Rough cost: engineering hours for a production-grade version of
      the agent invocation loop (auto-polling, not manual re-kick).
- [ ] Legal/IP: public-repo-of-prototype posture; internal-repo plan
      for pilot.

---

## Appendix A — Try the demo in 5 minutes

The fastest way to evaluate this is to watch it work:

1. Visit the repo: <https://github.com/tdevere/TrainingRequest>
2. Read [`docs/PROJECT-BRIEF.md`](../PROJECT-BRIEF.md) for the
   one-page design.
3. Read [`docs/PLAN-ACCEPTANCE-FORM.md`](../PLAN-ACCEPTANCE-FORM.md) —
   this is the structure every plan must hit.
4. Read [`.github/agents/plan-rubric.md`](../../.github/agents/plan-rubric.md) —
   this is the scoring rubric the critic AI applies.
5. Ask me for a live walkthrough of an end-to-end run on a real
   request from your backlog.

## Appendix B — What I am *not* proposing

- A replacement for Microsoft Learn.
- A replacement for SME judgment.
- A new front-end / portal at this stage — intake is a GitHub Issue
  form, reviews are GitHub comments. Ship the workflow, not the UI.
- A mandatory process. Pilot is opt-in per region.
