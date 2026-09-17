# Discovery Brief — Conversational CRM

**Product slug:** `conversational-crm`
**Version:** v1.1
**Source:** Discovery interview transcript (8 baseline questions + 1 follow-up, confirmed sufficient by human on 2026-09-17)
**Revision note:** v1.1 responds to the human's Gate 1 reply on v1.0 (`revise: we need to
consider V-D2 as part of the requirements`). It adds Section 9 below, carrying forward
the specific agents/capabilities named in the original idea statement for Requirements
to explicitly reconcile. Sections 1–7 are unchanged from v1.0 — the human confirmed the
single-persona/generic-capture framing in those sections is correctly sourced from the
interview and should not be redrafted to match the richer idea text. Section 8 gains one
new item (5), pointing to the new Section 9.

---

## 1. Problem Statement

Traditional CRMs fail because logging a meeting means stopping to fill in forms — a
disconnected step that reps skip or delay, so data quality suffers and adoption stays
low. Commitments made in conversation live only in memory or inboxes, with no system
tracking follow-through, and reconstructing "where things stand" with a customer means
manually piecing together notes and emails.

Success means capture becomes as effortless as sending a text, so it happens every
time. Commitments get tracked automatically and surfaced before they're due, and
relationship history becomes instantly queryable instead of archaeological. The result:
reps actually use the system because it removes work rather than adds it, and
relationship knowledge lives in a shared system rather than individual heads.

`[INFERRED — needs confirmation]` The framing above is based on the team's general
knowledge of how CRM adoption typically fails, not on validated research into what this
specific group of reps uses today — see the "Current pain points" answer under
Assumptions below, where the interview could not identify a concrete status-quo tool
or workflow.

## 2. Target Users

### Sales Rep (single persona — primary and only audience)

The interview identified one user type. When asked whether a second group with
different needs exists, the answer was explicit: "same role sharing the same
capabilities" — i.e., genuinely single-audience.

- **Who:** Sales rep.
- **Prerequisite to use the product at all:** seeded demo data (customer accounts,
  contacts, prior interaction history) must already exist — the rep is not expected to
  populate the system from a blank state.
- **Access model:** a single shared login is sufficient for the demo, rather than
  individual per-rep logins. This was a deliberate simplification confirmed via
  follow-up, given the product is a web app used on a laptop/desktop after a meeting.
  `[INFERRED — needs confirmation]` A consequence of the shared-login decision is that
  commitments and notes captured during the demo will not be attributed to a specific
  individual rep — this was not stated outright, but follows directly from "single
  shared login is enough" answering the attribution question.

No second persona (e.g., a manager or a different functional role) was identified in
this discovery pass.

## 3. Goals & Success Metrics

**Goals** (from the Problem & Goals answer):
- Make capture effortless enough that it happens every time, instead of being skipped.
- Track commitments automatically and surface them before they are due.
- Make relationship history instantly queryable rather than requiring manual
  reconstruction from notes and emails.
- Keep relationship knowledge in a shared system rather than in individual reps' heads.

**Success metric** (stated directly): the demo succeeds if its agents are functioning
end to end on the seeded data.

`[INFERRED — needs confirmation]` "Functioning end to end on the seeded data" is not
yet broken into concrete, checkable criteria (e.g., which specific interactions must
succeed, what counts as a correct answer to a memory/query request). This is carried
forward as an Open Item for the Requirements stage to define.

## 4. Scope Boundaries

**Explicitly out of scope for this first version** (stated directly):
- No real integrations with external systems.
- No multi-team support.
- No multi-tenant support.
- No real system integrations of any kind — adapters are used to simulate external
  connections instead of connecting to real systems.

**In scope** (by contrast, and consistent with the rest of the transcript): a
single-team, single-tenant demo running entirely against seeded data, with external
systems simulated via adapters rather than integrated for real.

## 5. Constraints & Integrations

- The system runs on **seeded data** for demo purposes — not live/real data.
- The seeded data includes: **MOM (minutes of meeting), emails, and SharePoint**
  content.
- No real system integrations exist; where an integration would normally be needed,
  an adapter simulates the external connection instead.
- `[INFERRED — needs confirmation]` Access is via a single shared login (see Target
  Users) — no individual authentication/authorization system is required for this
  version. The interview never used the words "authentication" or "authorization";
  this follows from the same "single shared login is enough" answer as the
  attribution inference in Target Users.

## 6. Risks & Compliance Notes

- Stated directly: the demo uses **seeded data only — no real data for now**, which
  removes real customer-PII exposure as a concern for this phase.
- `[INFERRED — needs confirmation]` If a later phase moves from seeded to real
  customer data, standard data-privacy/PII handling and access-control requirements
  would need to be revisited at that point — this was not discussed in the interview
  and is out of scope for this demo phase.

No other sensitivity around personal data, money, safety, or regulation was raised in
the interview.

## 7. Assumptions

- `[INFERRED — needs confirmation]` The problem statement rests on general knowledge of
  CRM adoption failure patterns rather than on validated research into this specific
  user base's current tools/workflows — the "Current pain points" answer stated
  explicitly: "not sure what people use these days, we are building a demo based on
  what we know and have shared."
- `[INFERRED — needs confirmation]` A single shared login means commitments/notes are
  not attributed to an individual rep in this version (see Target Users).
- `[INFERRED — needs confirmation]` "Functioning end to end on the seeded data" (the
  stated success metric) implies the seeded data set must exercise every agent-facing
  capability described in the goals (capture, commitment tracking, queryable history),
  but the interview did not enumerate which specific scenarios constitute "end to end."
- `[INFERRED — needs confirmation]` If real customer data is introduced in a later
  phase, additional compliance handling would be needed beyond what this demo requires
  (see Risks & Compliance Notes).

No items were flagged as `[INFERRED — needs confirmation]` during the interview itself
(`discovery.open_items` was empty at handoff); all four items above were identified
while organizing the transcript into this Brief.

## 8. Open Items

1. Current-state pain points and the tools reps actually use today were not
   identified in discovery ("not sure what people use these days") — validate this
   with real reps before treating the problem statement as more than a working
   assumption.
2. Concrete, checkable acceptance criteria for "agents functioning end to end on the
   seeded data" are not yet defined — needed at the Requirements stage.
3. Whether commitments/notes will ever need per-rep attribution beyond this demo
   (given the shared-login decision) is unresolved — worth a deliberate call before any
   future phase, rather than defaulting into it.
4. The compliance/data-handling path for a future phase using real customer data
   (rather than seeded data) has not been discussed and is unresolved.
5. The original idea statement (`project.idea_input`) names specific agents and
   product capabilities that were not restated or elaborated during the discovery
   interview and therefore do not appear as stated fact in Sections 1–7 above. See
   **Section 9** below for the full list and the mandatory reconciliation step this
   creates for Requirements.

## 9. Carried Forward From Original Idea (Not Yet Confirmed via Interview)

This section exists because of validator finding V-D2 on v1.0 of this Brief, and the
human's Gate 1 reply choosing to carry it forward here rather than reopen the
interview: *"we need to consider V-D2 as part of the requirements."*

**Why this section exists, and why it is separate from Sections 1–8.** The original
idea statement recorded in `project.idea_input` (workflow.json) names five specific
agents and several concrete product capabilities. None of this was independently
restated, elaborated, or confirmed during the Discovery interview — the interview's own
answers paraphrased the idea into generic language (e.g., "capture becomes effortless,"
"commitments get tracked automatically") without naming any of the items below. Per
discovery-agent's no-new-facts rule, this Brief's Sections 1–7 are built only from
`discovery.interview_log`, so these items correctly do **not** appear there as
confirmed fact. Listing them only here — tagged as unconfirmed — is the way to avoid
both of the wrong outcomes: silently dropping them, or smuggling them into the Brief's
main sections as if the interview had confirmed them.

**Verbatim from `project.idea_input`, not yet confirmed via the interview transcript:**

- **Capture Agent** — "the Capture Agent extracts contact details into a structured
  meeting note," triggered by a rep scanning a business card or dropping a one-line
  summary after a meeting (e.g., "met Priya and Arjun from Acme, discussed renewal
  pricing, they want a demo of module X by Friday").
- **Business-card scanning** as an input method for the Capture Agent (in addition to,
  or instead of, a typed one-line summary).
- **Extraction Agent** — "pulls out key discussion points, commitments, follow-ups, and
  next steps (capturing a due date where one was mentioned) and tags them to the
  correct customer thread."
- **Memory/Q&A Agent** — reps "query that thread in natural language" instead of using
  dashboards, with the example queries given verbatim: *"what did we discuss last
  time?"*, *"what did I commit to?"*, *"who attended from their side?"* — and get
  "instant answers pulled from history."
- **Commitment Tracking Agent** — "proactively surfaces a due-soon/overdue list of open
  commitments across accounts (no calendar or notification integration needed)."
- **Brief-Me-on-Customer Agent** — "synthesizes a short pre-meeting summary — recent
  history, open items, stakeholders — from a single request like 'brief me on Acme.'"
  The idea statement also specifies this should "remain a short generated summary
  rather than a polished report."
- **A customer profile view** "showing the running conversational history."
- **A simple shared view** "so a second team member (e.g., an account manager) can see
  and add to the same customer thread" — distinct from, and possibly in addition to,
  the single-shared-login simplification recorded in Section 2/5/7 above.
- **Optional voice-note capture** — the idea statement treats this as droppable if
  speech-to-text "eats into build time," with typed one-liners as "an acceptable
  substitute" — i.e., explicitly not a hard requirement even in the original idea.

For completeness: the idea statement also treats "enterprise-grade access control and
governance" as an explicit stretch goal, consistent with (not contradicting) the
single-shared-login/no-individual-auth simplification already carried in Sections 2, 5
and 7 as an interview-grounded inference.

**What this means for Requirements — a mandatory checkpoint, not an assumption:**

`requirements-agent` must explicitly evaluate and reconcile each item listed above
against the confirmed interview scope (Sections 1–8 of this Brief) before treating any
of them as an in-scope PRD feature. For each item, the PRD must record one of:

1. **Adopted as in-scope** — with a rationale tying it to a goal or capability the
   interview actually confirmed (Sections 1, 3 and 4 above), not merely to the
   original idea text; or
2. **Explicitly deferred / out of scope for this version** — with a stated reason; or
3. **Flagged back to a human** for an explicit scope decision, where the interview
   record genuinely does not give Requirements enough to decide either way.

Requirements must not silently drop these items (treating Sections 1–8's generic
framing as if it were the exhaustive intended scope) and must not silently adopt them
either (treating the original idea statement as already-confirmed scope, which it is
not — nothing here has been through the interview or a human gate as a scope decision).
Either silent path would repeat the exact gap V-D2 identified.

---

*Drafted by discovery-agent from the confirmed Discovery interview transcript. No
fact in Sections 1–8 is introduced beyond what is stated in, or a flagged inference
from, `discovery.interview_log`. Section 9 is sourced from `project.idea_input`
directly and is explicitly marked as not yet confirmed via the interview, per the
human's Gate 1 revise reply.*
