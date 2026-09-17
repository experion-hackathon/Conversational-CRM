# Discovery Brief — Conversational CRM

**Product slug:** `conversational-crm`
**Version:** v1.0
**Source:** Discovery interview transcript (8 baseline questions + 1 follow-up, confirmed sufficient by human on 2026-09-17)

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

---

*Drafted by discovery-agent from the confirmed Discovery interview transcript. No
fact in this document is introduced beyond what is stated in, or a flagged inference
from, `discovery.interview_log`.*
