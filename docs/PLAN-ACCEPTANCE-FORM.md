# Plan Acceptance Form

> Used by the **Reviewer** (Gate #1) when a training or PDP plan reaches
> `state:plan-drafted`. Fill in this form in an issue comment, then apply
> the appropriate state label.

---

## How to use

1. Read the proposed plan comment on the issue.
2. Copy the form block below into a **new comment** on the same issue.
3. Fill in every field.
4. Apply the label matching your decision:
   - Approve → `state:plan-approved` (remove `state:plan-drafted`, `needs:reviewer`)
   - Request changes → keep `state:plan-drafted`, keep `needs:reviewer`

---

## Plan Acceptance Form

```
ROLE: Reviewer — Gate #1 plan review

**Issue:** #<number>
**Plan comment:** <!-- link to the plan comment -->
**Reviewed by:** @<your-handle>
**Review date:** YYYY-MM-DD

### Decision
- [ ] ✅ Approved as-is
- [ ] 🔄 Approved with minor edits (list below)
- [ ] ❌ Changes required before approval (list below)

### Scope check
- [ ] Learning outcomes are observable (measurable behaviors, not internal states).
- [ ] Level matches requestor's stated background.
- [ ] Duration is realistic for the format chosen.
- [ ] At least one hands-on element is present (required when requestor asked for lab/exercises).
- [ ] No existing catalog asset already covers this request.

### Outcome alignment
<!-- For each outcome requested, confirm it is addressed in the plan.
     Cross out any that are missing; mark additions if you're adding any. -->
- [ ] <outcome 1>
- [ ] <outcome 2>
- [ ] <outcome 3>

### Edits / Required changes
<!-- List any changes needed. Be specific (section, wording, addition/removal). -->
1.

### SME assignment
**Proposed SME:** @<handle or "tbd">
<!-- If "tbd", name who will identify the SME and by when. -->

### Notes for the AI content builder
<!-- Optional: constraints, tone, depth, style, platform choices. -->

---
<!-- Reviewer: after filling in the form, apply the matching label and, if
     approved, reply with /approve-plan so the AI agent picks up the signal. -->
```

---

## Quick reference: what "approved" means

Approving the plan authorises the AI agent to:

1. Open a PR that adds a file under `/trainings/` (or `/pdps/`) populated from
   the template.
2. Assign the nominated SME as a PR reviewer.

It does **not** mean the content is verified — that is Gate #2 (SME review).
