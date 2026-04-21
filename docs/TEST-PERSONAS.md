# Test Personas

Synthetic requestors used during bootstrap testing. Each scenario exercises
a different slice of the workflow: different levels, formats, and a PDP.

All are DevOps-focused.

---

## P1 — "Alex" · Beginner developer

- **Background:** Application developer (C#/Node). 3 years experience.
  Has used Git for daily commits but never owned a deployment.
- **Current pain:** Manager asked them to "own the deploy" for a small
  internal service. They don't know where to start.
- **Time available:** 3–5 hours this week, then 1–2 hours/week.
- **Preferred format:** Written guide + one hands-on lab.
- **Stretch goal:** Not certification; just competence.
- **Requests:** Training — *"From commit to running service: your first
  deployment"* (level: beginner).

## P2 — "Priya" · Intermediate SRE-curious engineer

- **Background:** Backend engineer, 5 years. Writes YAML daily. Has
  deployed via someone else's pipelines but never authored one end-to-end.
- **Current pain:** Team has inconsistent CI. Wants to standardize on
  GitHub Actions with reusable workflows.
- **Time available:** 6–10 hours/week for 2 weeks.
- **Preferred format:** Hands-on lab, with a reference cheatsheet to keep.
- **Requests:** Training — *"Reusable GitHub Actions: build a shared
  CI library for your org"* (level: intermediate).

## P3 — "Jordan" · Advanced platform engineer

- **Background:** Platform engineer, 8+ years. Comfortable with
  Kubernetes basics. Runs a small AKS cluster for staging.
- **Current pain:** Moving to a production-grade multi-tenant cluster.
  Needs depth on security boundaries, quota, and policy.
- **Time available:** Half-day intensives.
- **Preferred format:** Live session or recorded walkthrough + deep lab.
- **Requests:** Training — *"Production-grade AKS: tenancy, quotas, and
  policy"* (level: advanced).

## P4 — "Sam" · Junior developer wanting to grow into DevOps

- **Background:** Junior full-stack dev, 18 months in industry.
- **Current pain:** Wants a career path — doesn't want to stay in feature
  work forever. Attracted to platform/DevOps roles.
- **Time available:** 3–5 hours/week sustainably.
- **Horizon:** 6 months.
- **Preferred format:** Mix. Strong preference for mentorship.
- **Requests:** **PDP** — *"Junior developer → DevOps engineer in 6 months"*
  with a mentor.

---

## What each persona tests

| Persona | Kind | Level | Format | Notable test |
| --- | --- | --- | --- | --- |
| P1 Alex | Training | beginner | written + lab | Simplest happy path |
| P2 Priya | Training | intermediate | lab + reference | Multi-format handling |
| P3 Jordan | Training | advanced | live + lab | Complex content, SME load |
| P4 Sam | PDP | — | mixed + mentor | PDP → spawns sub-training-requests |

Together they exercise: all three levels, multiple formats, a PDP with
linked trainings, and the iteration/feedback loop (when one persona
returns with feedback).
