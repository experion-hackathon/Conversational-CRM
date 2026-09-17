# User Stories — Conversational CRM

- **Epic/Product name:** Conversational CRM / Relationship Memory Assistant
- **Document reference:** `USER-STORIES-conversational-crm-v2.0`
- **Source document:** PRD — `PRD-conversational-crm-v1.8.md` (approved, requirements_review gate, 2026-09-17)
- **Owner:** Product Owner (role, not named individual)
- **Status:** Draft — pending validation and human gate
- **Version:** 2.0
- **Supersedes:** `USER-STORIES-conversational-crm-v1.0` (approved and published to Jira RPOC-23..RPOC-43 against PRD v1.5; stale against PRD v1.8's feature cuts — see "What changed since v1.0" below). v1.0 is kept on disk as the historical record; its full gate/publish evidence is preserved in state under `superseded_v1_0`, not deleted.
- **AC format:** Imperative, matching the PRD's own style (default — a human may request Given/When/Then via `revise:` at this stage's gate).
- **Persona:** Sales Rep — the PRD's single confirmed persona (Section 3). No second persona is invented.

Every story below traces to exactly one PRD feature ID from PRD v1.8's current
scope (F-1, F-2, F-3, F-4, F-5, F-8). Where a PRD acceptance criterion itself
carries an `[INFERRED — needs confirmation]` tag on its *mechanism* (not its
underlying capability), that tag is carried into the story's acceptance
criteria rather than resolved by invention. No feature was blocked — see
"Blocked Features" below.

## What changed since v1.0

This is a full redraft against PRD v1.8, not an incremental patch — story IDs
are renumbered from scratch rather than leaving gaps where features were cut.

- **Removed entirely:** the F-6 (Customer Profile) and F-7 (Shared Customer
  Thread Access) stories (old US-18 through US-21) — both features were cut
  from the PRD in v1.8 as standalone features, per the human's scope
  clarification that this product will not own a dedicated profile view or a
  separate shared-access feature beyond the existing single-shared-login NFR
  (unchanged, still in force, not re-storied — NFRs are never their own story).
  F-9 (Contact/Lead Categorization), also cut in v1.8, was never storied in
  v1.0 (it was added in v1.6, after v1.0 was drafted and gated), so there is
  nothing to remove for it here.
- **Added:** four new stories for F-8 (Unified Home / Conversational Entry
  Surface), a feature that did not exist when v1.0 was drafted.
- **Added:** one new story each for F-1 AC6 and F-3 AC5 — both state that
  capture and Q&A can be submitted through the single conversational input
  with no precondition customer-selection step, generalizing the existing
  AC1/AC4 matching behavior rather than replacing it.
- **Reworded:** F-5's brief-status story (old US-16, now US-18) — the PRD's
  vocabulary shifted from "opportunity/deal status" to "relationship status"
  in v1.8, since this product owns no CRM pipeline/deal-stage data of its own.
  The underlying capability and both branches (has content / no content) are
  unchanged; only the wording changed.
- **Unchanged (carried forward as-is):** all F-1 AC1–AC5 stories (business-card
  capture, attendee extraction), all of F-2, all of F-3 AC1–AC4, all of F-4,
  and F-5 AC1–AC3's happy-path/partial-summary/reject-ambiguous behavior.

---

## F-1 — Conversational Interaction Capture

### US-1 — Capture a typed interaction note against a matched customer
**Type:** Happy path
**Story:** As a Sales Rep, I want to record what happened in a customer interaction as a single typed note that automatically ties to the right customer, so that logging an interaction takes one line of typing instead of stopping to fill out a CRM form.

**Acceptance criteria:**
1. The system must create a structured interaction note and associate it with the correct customer thread when the rep's typed entry unambiguously matches exactly one seeded customer account.
2. The system must save the entry as a new interaction note when the entry contains at least one non-whitespace character.

### US-2 — Reject invalid capture input and flag unmatched customers
**Type:** Validation
**Story:** As a Sales Rep, I want the system to tell me clearly when a note can't be saved or matched, so that I never lose an interaction record or have it silently attached to the wrong customer.

**Acceptance criteria:**
1. The system must reject the submission with an explicit "cannot save an empty note" message, and must not create any note record, when the rep submits an empty or whitespace-only entry.
2. The system must flag the note as "unmatched — needs customer selection" — rather than attaching it to a default or incorrect account — when no seeded customer account can be unambiguously identified from the entry text.

### US-3 — Resolve an unmatched note to the correct customer
**Type:** Edge case
**Story:** As a Sales Rep, I want to manually pick the right customer for a note the system couldn't match automatically, so that no interaction detail is lost or misfiled while I haven't yet confirmed the account.

**Acceptance criteria:**
1. The system must attach a previously flagged "unmatched" note to the customer thread the rep selects when the rep manually resolves the flag.
2. The system must leave the note in the unmatched/needs-selection state — rather than guessing an account — when the rep has not yet resolved it.

### US-4 — Capture an interaction via a scanned business card
**Type:** Edge case
**Story:** As a Sales Rep, I want to capture a contact's details by scanning their business card instead of typing them, so that I can log a new contact from a meeting without manual data entry.

**Acceptance criteria:**
1. `[INFERRED — needs confirmation]` The system must extract the contact's name, company and role from a scanned business-card image into the structured interaction note when the image is legible and contains recognizable contact fields. The scanning/OCR mechanism itself was never discussed in discovery (PRD Assumption 5, Open Question 5) — whether this capability is worth building for the demo at all, versus typed-only capture, is still open.
2. `[INFERRED — needs confirmation]` The system must inform the rep that manual entry is required — without fabricating contact details — when the image cannot be read or no contact fields are recognized.

### US-5 — Extract attendee names from a typed interaction note
**Type:** Happy path
**Story:** As a Sales Rep, I want the system to automatically pick out everyone I mention meeting with in my typed note, so that I can later ask "who attended from their side?" and get a real answer instead of nothing.

**Acceptance criteria:**
1. The system must extract every distinct attendee name mentioned in a typed free-text interaction entry as a separate structured attendee item — recording one item per distinct person named (e.g., "met Priya and Arjun from Acme..." must yield two attendee items, not one merged string) — when the entry text names at least one identifiable person.
2. The system must record zero attendee items for that interaction — rather than fabricating a name — when the entry text names no one.
3. `[INFERRED — needs confirmation]` The exact name-recognition/disambiguation mechanism — how confidently a token must be recognized as a person's name, and how several names in one entry are told apart — was never discussed in discovery (PRD Assumption 14, Open Question 15) and is not assumed here; only the underlying capability (AC1/AC2) is treated as confirmed.

### US-6 — Capture from the home conversational input without selecting a customer first
**Type:** Happy path
**Story:** As a Sales Rep, I want to type an interaction note straight into the single conversational input on my home screen without first opening or selecting a customer, so that capturing what happened takes one step instead of stopping to find the right account first.

**Acceptance criteria:**
1. The system must accept a capture entry submitted through the single conversational input (F-8) without requiring the rep to have previously selected, opened, or confirmed a specific customer account in a separate step.
2. The system must not present a mandatory customer-selection step before an entry can be typed or submitted — customer identification happens after submission, from the entry's own content (per US-1/US-2), not before.

---

## F-2 — Automatic Extraction & Thread Tagging

### US-7 — Extract commitments and next steps from a captured note
**Type:** Happy path
**Story:** As a Sales Rep, I want the system to automatically pull out commitments and next steps from my captured note, so that I don't have to separately remember or re-type what I promised.

**Acceptance criteria:**
1. The system must extract each distinct commitment, follow-up or next step mentioned in a captured note's text as a separate tracked item when the note contains one or more of them.
2. The system must record no commitment items when the note describes discussion only, with no forward-looking commitment.

### US-8 — Resolve a due date for an extracted commitment
**Type:** Edge case
**Story:** As a Sales Rep, I want the system to work out a commitment's due date from what I actually typed, so that vague wording never gets turned into a fabricated deadline.

**Acceptance criteria:**
1. The system must record a due date on an extracted commitment when the captured text states a concrete date or an unambiguous relative date term resolvable against the interaction's timestamp (e.g., "by Friday").
2. The system must mark the commitment's due date as unspecified — rather than guessing a date — when the text uses a vague temporal reference (e.g., "soon," "sometime") or gives no date information at all.

### US-9 — Tag extracted items to the correct customer thread
**Type:** Validation
**Story:** As a Sales Rep, I want every discussion point and commitment pulled from my note to land on the right customer's thread, so that I never see — or act on — another customer's commitments.

**Acceptance criteria:**
1. The system must tag every extracted discussion point and commitment to the same customer thread as its source interaction note when that note is matched to a thread.
2. The system must flag an extracted item for manual thread assignment — rather than tagging it to an incorrect thread — when the source note itself is unmatched to a customer thread (per F-1).

---

## F-3 — Natural-Language Memory & Q&A

### US-10 — Ask what was discussed last time
**Type:** Happy path
**Story:** As a Sales Rep, I want to ask "what did we discuss last time?" about a customer and get an answer pulled from that customer's actual history, so that I can walk into a follow-up without hunting through old notes.

**Acceptance criteria:**
1. The system must answer a natural-language question about a customer's most recent discussion by returning content drawn from that customer's most recent interaction note(s) when the thread has at least one captured note.
2. The system must respond that no history exists yet for that customer when the thread is empty.

### US-11 — Ask what I committed to for a customer
**Type:** Happy path
**Story:** As a Sales Rep, I want to ask "what did I commit to?" for a customer and get a list of my open commitments for them, so that I never walk into a meeting having forgotten a promise I made.

**Acceptance criteria:**
1. The system must answer a natural-language question about the rep's own open commitments for a customer by listing the commitments extracted and tagged to that thread when any exist.
2. The system must state that there are no open commitments for that customer when none exist.

### US-12 — Ask who attended from the customer's side
**Type:** Happy path
**Story:** As a Sales Rep, I want to ask "who attended from their side?" for a customer and get back the names that were actually captured, so that I can quickly recall stakeholders without re-reading the whole thread.

**Acceptance criteria:**
1. The system must answer a natural-language question about interaction attendees by returning the contact names extracted from that thread's interaction notes when attendee names were captured.
2. The system must state that no attendee information was captured when none was extracted.

### US-13 — Reject a Q&A request naming no clear customer
**Type:** Validation
**Story:** As a Sales Rep, I want the system to tell me plainly when it can't tell which customer I mean, so that I never mistake someone else's history for the account I actually asked about.

**Acceptance criteria:**
1. The system must resolve and answer against the correct customer thread when a query names exactly one customer that matches a seeded account.
2. The system must respond that it could not identify the customer — rather than guessing or returning another customer's data — when the query names no customer or an unrecognized one.

### US-14 — Ask a question from the home conversational input without selecting a customer first
**Type:** Happy path
**Story:** As a Sales Rep, I want to ask a natural-language question about a customer directly in the same conversational input I use for everything else, without first opening that customer's record, so that getting an answer takes one step instead of two.

**Acceptance criteria:**
1. The system must accept and answer a natural-language question submitted through the single conversational input (F-8) without requiring the rep to have previously selected, opened, or confirmed a specific customer account, resolving the target customer from the query's own text alone (per US-13).
2. The system must not present a mandatory customer-selection step before a question can be asked.

---

## F-4 — Proactive Commitment Tracking

### US-15 — View overdue and due-soon commitments across accounts
**Type:** Happy path
**Story:** As a Sales Rep, I want a single list of commitments that are overdue or coming due soon across all my accounts, so that I can act before I miss a promise instead of finding out after the fact.

**Acceptance criteria:**
1. The system must list every open commitment whose due date has passed as "overdue" when the current date is past that due date, and must exclude a commitment from this list once it is marked complete.
2. `[INFERRED — needs confirmation]` The system must list every open commitment whose due date falls within the next 7 days as "due soon," and must reclassify it as "overdue" instead once its due date has passed. The 7-day window is not a confirmed value — discovery never stated a specific look-ahead window (PRD Open Question 6); 7 days is carried over only because the PRD itself uses it as an illustrative example, and needs explicit human confirmation before development.
3. The system must list a commitment with an unspecified due date (one with no stated or resolvable date, per F-2) separately from the due-soon/overdue list — rather than omitting it entirely or treating it as overdue — when no due date was captured.

### US-16 — See an explicit empty state when nothing is due soon or overdue
**Type:** Edge case
**Story:** As a Sales Rep, I want the commitment list to say plainly when nothing is due or overdue, so that I know the list is genuinely empty rather than wondering if it failed to load.

**Acceptance criteria:**
1. The system must state explicitly that there are no due-soon or overdue commitments — rather than showing an empty list with no explanation — when none exist.

---

## F-5 — Brief-Me-on-Customer Summary

### US-17 — Get a pre-meeting brief on a customer
**Type:** Happy path
**Story:** As a Sales Rep, I want to ask "brief me on [customer]" and get a short summary of their recent history, open commitments, known stakeholders, and where the relationship currently stands, so that I can walk into the next meeting prepared without re-reading the entire thread myself.

**Acceptance criteria:**
1. The system must generate a summary containing recent discussion history, open commitments, stakeholder/contact names, and a narrative synthesis of where the relationship currently stands for a named customer when that customer has at least one captured note, and must state that no history exists yet for that customer — rather than generating a summary with fabricated content — when the thread is empty.
2. `[INFERRED — needs confirmation]` The system must return a bounded, short-form summary (a few sentences or bullets, not a multi-page document) under normal conditions — no explicit length limit was stated in discovery (PRD Open Question 7) — and must still return whatever partial content is available, rather than failing the whole request, when data for one of the four summary components (history, commitments, stakeholders, relationship status) is missing.

### US-18 — Include relationship status in the customer brief
**Type:** Edge case
**Story:** As a Sales Rep, I want the brief to also tell me where things currently stand with a customer relationship, so that I can gauge momentum going into a meeting without piecing it together myself from old notes.

**Acceptance criteria:**
1. `[INFERRED — needs confirmation]` The system must synthesize the relationship-status component narratively from that customer's recorded interaction notes and commitments — since no discrete pipeline-stage field exists in the structured store (PRD Section 13.1) — when the thread contains content indicating where the relationship stands. This system holds no CRM pipeline/deal-stage data of its own (PRD Section 7, Data Ownership NFR); whether narrative synthesis is sufficient, or something more structured is wanted from a future phase, is still open (PRD Assumption 13, Open Question 14).
2. The system must state that no relationship-status information has been captured yet for that customer — rather than fabricating a stage or outcome — when the thread contains no such content.

### US-19 — Reject a brief request naming no clear customer
**Type:** Validation
**Story:** As a Sales Rep, I want the system to tell me plainly when it can't identify which customer I want a brief on, so that I never receive a brief that's actually about the wrong account.

**Acceptance criteria:**
1. The system must generate the summary for a named customer when the request unambiguously identifies exactly one seeded customer account.
2. The system must respond that it could not identify the requested customer — rather than guessing — when the request names an unrecognized or ambiguous customer.

---

## F-8 — Unified Home / Conversational Entry Surface

### US-20 — See my prioritized to-do list on login
**Type:** Happy path
**Story:** As a Sales Rep, I want to be greeted with my prioritized to-do/activity list the moment I log in, so that I immediately know what needs attention without having to ask a question first.

**Acceptance criteria:**
1. The system must greet the rep conversationally and display the prioritized to-do/activity list, drawn from Commitment Tracking's due-soon/overdue computation (F-4 US-15), when the rep logs in and at least one qualifying commitment exists.
2. The system must state explicitly that there is nothing due or overdue right now — consistent with F-4/US-16 — rather than showing an empty list with no explanation, when none exist.

### US-21 — See only the top items with a way to see more
**Type:** Edge case
**Story:** As a Sales Rep, I want the home list to show only my most important items with an obvious way to see the rest, so that a busy day's full list doesn't bury what matters most right now.

**Acceptance criteria:**
1. `[INFERRED — needs confirmation]` The system must display only the top N highest-priority items on the to-do/activity list when more than N qualifying items exist. The exact value of N was not stated by the reviewed UI concept this feature is sourced from (PRD Open Question 16) — a specific number needs human confirmation before development.
2. The system must give the rep an explicit way to see the remaining items ("see more") rather than truncating the list with no indication that more items exist, when more than N qualifying items exist.

### US-22 — Continue the conversation about a selected to-do item
**Type:** Happy path
**Story:** As a Sales Rep, I want to tap an item on my to-do list and keep talking about it in the same conversational input, so that acting on a commitment doesn't mean leaving the home screen and starting over somewhere else.

**Acceptance criteria:**
1. The system must continue the conversation about the specific commitment/activity the rep selects from the list — surfacing that item's associated customer thread in the same conversational input — when the rep selects an item.
2. The system must state clearly that the underlying item could no longer be located — rather than opening an unrelated thread — when the item's source data has since been removed.

### US-23 — See an explicit "nothing to show yet" state on first login
**Type:** Edge case
**Story:** As a Sales Rep, I want the home screen to say plainly that there's nothing yet on a brand-new account, so that I know the system is working correctly rather than wondering if the page failed to load.

**Acceptance criteria:**
1. The system must display an explicit "nothing to show yet" state on first login, when the seeded data yields no commitments and no prior interaction history at all — distinct from US-20 AC2's due-soon-empty state, which still implies some interaction history exists — rather than presenting a blank home surface with no explanation.

---

## Blocked Features

None. Every PRD v1.8 feature (F-1, F-2, F-3, F-4, F-5, F-8) produced at least
one testable story. Where a PRD acceptance criterion's underlying *capability*
was confirmed but its *mechanism* was not (business-card OCR, attendee-name
disambiguation, the due-soon look-ahead window, the brief's exact length
bound, relationship-status synthesis, and the home list's top-N bound), the
story was still written against the confirmed capability with the unresolved
mechanism flagged inline as `[INFERRED — needs confirmation]`, rather than
dropping the feature — per the standing rule that not every open item makes
an AC unverifiable.

## Open Items (carried forward from story drafting)

1. Is business-card OCR scanning worth implementing for this demo, or is typed-only capture acceptable? (US-4; PRD Open Question 5.)
2. What mechanism should recognize and disambiguate distinct attendee names in a typed entry, and at what confidence? (US-5 AC3; PRD Open Question 15.)
3. Is the "due soon" look-ahead window actually 7 days, or some other number? US-15 AC2 uses 7 days only as a carried-over illustrative placeholder, not a confirmed value. (PRD Open Question 6.)
4. Is there a maximum length or format for the Brief-Me-on-Customer summary beyond "short"? (US-17 AC2; PRD Open Question 7.)
5. Is narrative-only relationship-status synthesis (US-18) sufficient, or does a future phase need something more structured? (PRD Open Question 14.)
6. What is the exact top-N bound for the home to-do/activity list, and does "see more" page, expand inline, or navigate elsewhere? (US-21; PRD Open Question 16.)

None of these are new — each is an existing PRD open question/assumption
surfaced again because it bears directly on a story's acceptance criteria.

## Traceability

| Feature | Stories |
|---|---|
| F-1 Conversational Interaction Capture | US-1, US-2, US-3, US-4, US-5, US-6 |
| F-2 Automatic Extraction & Thread Tagging | US-7, US-8, US-9 |
| F-3 Natural-Language Memory & Q&A | US-10, US-11, US-12, US-13, US-14 |
| F-4 Proactive Commitment Tracking | US-15, US-16 |
| F-5 Brief-Me-on-Customer Summary | US-17, US-18, US-19 |
| F-8 Unified Home / Conversational Entry Surface | US-20, US-21, US-22, US-23 |

**Total: 23 stories across 6 features, 0 blocked.**

## Self-Verification

- [x] Every non-blocked PRD feature has at least one story. — All 6 in-scope features covered (23 stories total), 0 blocked.
- [x] Every blocked feature is listed in `blocked_features` with a specific reason, not generic. — N/A, none blocked.
- [x] Every story's persona is one the PRD actually names, never "user." — All 23 stories use "Sales Rep," the PRD's single confirmed persona (Section 3).
- [x] Every story has one goal — no "and" hiding a second goal. — Checked each story individually; multi-part PRD ACs (e.g., F-4, F-5, F-8) were split across separate stories (US-15/US-16, US-17/US-18/US-19, US-20/US-21/US-22/US-23) rather than joined under one goal.
- [x] Every AC is testable and observable, not vague. — Where a PRD AC's mechanism was unconfirmed, a concrete carried-over placeholder or the PRD's own qualitative wording was used and flagged `[INFERRED — needs confirmation]`, rather than left untestable.
- [x] Happy path, validation failure, and empty state are each covered somewhere per feature. — F-1: US-1 (happy), US-2 (validation/empty via reject), US-3 (edge), US-6 (happy); F-2: US-7 (happy/empty), US-9 (validation); F-3: US-10/11/12 (happy + empty-state branch each), US-13 (validation), US-14 (happy); F-4: US-15 (happy), US-16 (empty state); F-5: US-17 (happy/empty), US-19 (validation); F-8: US-20 (happy/empty), US-23 (empty, first-login).
- [x] No NFR is duplicated as its own story. — Checked against all 6 PRD NFRs (Security/Privacy, Access Control, Integration, Data Ownership, Concurrency, Data Architecture, Architecture/Agent Composition); none was turned into a standalone story. The single-shared-login NFR is referenced by nothing here since F-7's dedicated feature/stories were cut — the NFR itself still stands but is not re-storied, consistent with the rule that NFRs are referenced, not repeated.
- [x] Every story is tagged with the PRD feature ID it traces to. — See Traceability table above; each story's heading and JSON record cites its `feature_id`.

**Result: PASS.** No items required a fix before handoff; the checklist passed on first pass.
