# Data and Integration — Conversational CRM

- **Product name:** Conversational CRM / Relationship Memory Assistant
- **Document reference:** `DATA-AND-INTEGRATION-conversational-crm-T-1-v1.1`
- **Discipline:** `data_integration` (Workflow 2 — Solution Architecture)
- **Task:** T-1, scope `full-product-v1` — all 7 PRD features (F-1..F-7), all 21 stories (US-1..US-21). Same scope as `solution` — this discipline designs against it, not around it.
- **Owner:** Architect (role, not named individual)
- **Status:** Draft — pending independent validation and human gate (`data_integration_review`)
- **Version:** 1.1 (revised from v1.0 at Workflow 2 sign-off; see Section 0.1)
- **Upstream discipline:** `solution` (Workflow 2), gate `approved`, published. This document uses `solution`'s exact component IDs (C-WEB, C-API, C-SQL, C-VEC, C-NLU) and does not redraw any boundary it set.

## 0. Scope boundary (what this document does and does not cover)

This is the `data_integration` discipline's artifact: the entity model (fields, types,
constraints, indexes) and the API contract (operations, request/response shapes,
error semantics), reviewed together as one bundle because a contract approved apart
from the data model it implies is an approval of nothing. It deliberately does
**not** specify:

- component identity or boundaries (owned by `solution` — used here as fixed input);
- authentication mechanism, threat model, or measurable security/performance targets
  (owned by `security_nfr` — v1.0's OpenAPI contract named a placeholder security
  scheme, `SharedLoginAuth`, solely so every operation could declare an explicit
  `security` requirement; v1.1 adds the login/logout API surface that scheme's own
  resolved mechanism needs, per Section 0.1 below, but the mechanism itself — session
  lifecycle values, lockout thresholds, rate-limit thresholds — remains entirely
  `security_nfr`'s decision, not re-derived here);
- cloud topology, CI/CD, rollback, or the deployment mode of SQLite/Chroma
  (embedded vs. server — owned by `platform`, per solution.json SOL-9).

Where this document needed something one of those disciplines owns and it does not
exist yet, it names the gap as an open item rather than filling it (Section 5).

## 0.1 v1.1 revision note

At Workflow 2 sign-off, all four disciplines (`solution`, `data_integration`,
`security_nfr`, `platform`) had already gated and published v1.0 of their own
artifacts. The human replied `revise: data_integration <feedback>` rather than
`lock`, because `security_nfr`'s already-approved `SharedLoginAuth` resolution
depends on API surface this contract's v1.0 did not have. The feedback, recorded
verbatim in `roles/data_integration.json`'s `audit_trail` (the `revise` entry dated
2026-09-17T17:50:00Z) and summarized here:

- `security_nfr`'s SEC-1 (login step + session-token issuance), SEC-2 (session
  lifecycle including logout invalidation), SEC-5 (login-endpoint lockout after 5
  failed attempts in 15 minutes), and SEC-11/NFR-10 (`429`-class rate-limit rejection
  on `POST /interactions`, `POST /qa`, `POST /briefs`) all require API operations and
  response types that v1.0's contract never defined: no `/login` or `/logout`
  operation, and no `429`/`TooManyRequests`-class response anywhere.
- This gap was recorded by `security_nfr` as `returned_findings[0]` (`id: RF-1`,
  `finding_id: V-SEC2`, severity high, independently confirmed by `security_nfr`'s
  own Validator) and mirrored as `security_nfr`'s Open Item OI-13
  (`blocks_development: true`) — which is why `workflow_signoff`'s `lock` reply was
  refused: this workflow's own rule is that `lock` requires no open item anywhere
  with `blocks_development: true`.
- The feedback was explicit that this is **new API surface to add, not a change to
  any of the 9 existing business operations' own request/response shapes** —
  `security_nfr`'s own document Section 3 already confirmed "no change to any
  existing operation's security requirement or schema is needed."

**What v1.1 adds** (Section 3, decisions DAT-10/DAT-11, Appendix A):

1. `POST /auth/login` — exchanges the shared credential (SEC-1) for the session
   cookie the `SharedLoginAuth` scheme already named.
2. `POST /auth/logout` — invalidates that session server-side (SEC-2).
3. Two `429`-class response components — `RateLimited` (SEC-11/NFR-10, applied to
   `POST /interactions`, `POST /qa`, `POST /briefs`) and `LoginLocked` (SEC-5/NFR-4,
   applied to `POST /auth/login`) — see DAT-11 for why these are two components, not
   one.

**What v1.1 does not touch:** none of the 9 v1.0 business operations' request or
response schemas changed (verified in Section 6's self-verification); the data model
(Appendix B / `DATA-MODEL-conversational-crm-T-1-v1.1.md`) is unchanged except for
its own revision note and one new open item (OI-18) — no entity, field, constraint,
or index changed.

This document does not itself mark `security_nfr`'s `returned_findings[0]`/`RF-1` as
resolved — that is `security_nfr`'s own decision to make once it reviews this v1.1
contract, not this discipline's to declare on another role's behalf.

## 1. Source references

| Role | Artifact | Path (relative to this workflow's own artifact root) | Version | SHA-256 (bytes as read by this discipline) |
|---|---|---|---|---|
| `product_state` | Workflow 1 `workflow.json` (lock confirmation) | `../state/workflow-1-discovery-to-stories/workflow.json` | locked | `fb25e9c8829f6e0d0d9ccf9cddf5db8ae9b0cb7eba5873baff9e7e8bffc21b27` |
| `prd` | PRD | `../workflow-1-discovery-to-stories/docs/requirements/PRD-conversational-crm-v1.5.md` | 1.5 | `3960abdacf6ba3a27217d913aca01fa932bfb65373e572cdc4eeb6ac77dccc64` |
| `user_stories` | User Stories | `../workflow-1-discovery-to-stories/docs/stories/USER-STORIES-conversational-crm-v1.0.md` | 1.0 | `43fc3c0690db8cc6a0d85a837286912072fbcfdbfa55d3d37cb74c6abe8de73b` |

All three hashes were re-verified against the current file bytes on disk for this
v1.1 revision (`sha256sum`) and are unchanged from v1.0 — including the PRD's
already-disclosed line-ending discrepancy (Open Item OI-1, owned by `solution`/the
Workflow 1 output tree, not re-litigated here).

**Upstream discipline artifacts consumed for this v1.1 revision (not `source_references`
rows, per this workflow's schema, which reserves that array for Workflow 1
artifacts):**

- `solution`'s document, `docs/architecture/SOLUTION-ARCHITECTURE-conversational-crm-T-1-v1.0.md`
  — hash `99038391a4a33ae850e1ddc50e4709cfebb2b66525b10614f848c44b8368cc04`, matching
  `solution.json`'s recorded value exactly (unchanged from v1.0's own read; carried
  forward, not re-read line-by-line for this revision since nothing in this revision
  touches solution's own component/boundary decisions).
- `security_nfr`'s document, `docs/architecture/SECURITY-AND-NFR-conversational-crm-T-1-v1.0.md`
  — read in full for this revision specifically to confirm SEC-1, SEC-2, SEC-5, and
  SEC-11's exact requirements before designing the login/logout operations and the
  `429` responses against them. Hash independently recomputed:
  `e22ca533051b734fab63af5f151e61b1659c0a7a1f2a6d2cba1a670c070c88b6` — matches
  `security_nfr.json`'s own recorded `artifacts[0].sha256` and approved
  `gate.artifact_manifest[0].sha256` exactly; no drift since its gate approval.
- `platform`'s document, `docs/architecture/PLATFORM-DESIGN-conversational-crm-T-1-v1.0.md`
  — read in full for this revision to confirm PLA-9's interim reverse-proxy-layer
  rate-limiting placement so this contract's new `RateLimited` response is additive
  to, not in conflict with, platform's already-published design. Hash independently
  recomputed: `74aab3a74cc333603fc4ef378b019e1fb1ebd16cc3aab388f255900baed4ecc6` —
  matches `platform.json`'s own recorded value exactly; no drift since its gate
  approval.

This discipline's own v1.0 document is superseded by this v1.1 document; the v1.0
file remains on disk as the historical published record (this workflow's versioning
rule: increment, never overwrite).

## 2. Technology decisions

Decisions carry `DAT-N` IDs, unique within this workflow's four-discipline ID space
(`solution` used `SOL-1`..`SOL-9`; `security_nfr` used `SEC-1`..`SEC-12`; `platform`
used `PLA-1`..`PLA-9`). DAT-1 through DAT-9 are unchanged from v1.0, carried forward
verbatim below since none of the entities, decisions, or rationale they record
changed in this revision. DAT-10 and DAT-11 are new in v1.1.

### DAT-1 — Entity design: five SQLite tables, one denormalized field
- **Category:** Data modeling
- **Choice:** `accounts`, `contacts`, `interactions`, `interaction_attendees`,
  `commitments` (full field/type/constraint/index detail in DATA-MODEL-conversational-crm-T-1-v1.1.md).
  `commitments.account_id` is denormalized from its parent interaction.
- **Alternatives considered:** A single wide `interactions` table with fixed
  attendee/commitment columns instead of child tables; a normalized `commitments`
  table with no denormalized `account_id` (requiring every reader to join through
  `interactions`).
- **Rationale:** PRD Section 13.1 itself states the zero-to-many, nullable-field
  shape is "exactly what a relational schema is suited for... rather than a fixed
  set of attendee/commitment columns" — this document implements that narrative
  decision as real DDL rather than re-deciding it. The denormalized `account_id` on
  `commitments` exists specifically because F-4's due-soon/overdue query (US-13)
  reads across every account at once and needs the account directly in its result
  rows without a join-per-row cost.
- **Source:** PRD Section 13.1; `solution.json` C-SQL responsibility statement.
- **Owner:** `data_integration`. **Status:** proposed.

### DAT-2 — Resolves Open Item OI-1a: embedding model, dimension, and re-embedding trigger
- **Category:** Vector-store technology finalization
- **Choice:** Amazon Titan Text Embeddings V2 (`amazon.titan-embed-text-v2:0`),
  1024-dimension output, cosine similarity distance. Re-embedding trigger: embed
  once at capture/seed-ingestion time; the one exception is a metadata-only upsert
  of `account_id` (not a re-embed of the vector) when a previously unmatched
  interaction is resolved to a customer (US-3). No other re-embedding trigger exists
  in this scope, since no feature edits previously-captured content (DATA-MODEL
  Section 2.3).
- **Alternatives considered:** A separate, non-Bedrock embedding model (already
  rejected at the `solution` level by decision SOL-6, not re-opened here); a larger
  Titan dimension (3072) or the smaller Titan V1 model — V2 at 1024 dimensions
  chosen as Bedrock's current-generation default, balancing retrieval quality against
  Chroma index size for a small seeded demo dataset, with no stated NFR target
  forcing a specific dimension.
- **Rationale:** SOL-6 already fixed Bedrock as the sole embedding source; this
  decision fixes the specific model, since PRD Assumption 10 explicitly left that
  choice, the dimension, and the re-embedding trigger to the Architecture stage.
  The re-embedding trigger design follows directly from a fact this discipline
  confirmed while building the data model: no story in this scope ever edits
  previously-captured content, so "re-embed on change" has no case to fire in this
  scope except the customer-resolution metadata update, which does not change the
  embedded text at all.
- **Source:** PRD Section 13.2, Assumption 10 (OI-1a); solution.json SOL-6.
- **Owner:** `data_integration`. **Status:** proposed.

### DAT-3 — Resolves Open Item OI-2: SQLite concurrency mechanism
- **Category:** Data-access / concurrency
- **Choice:** WAL (write-ahead logging) journal mode, a `busy_timeout` of 5000ms, and
  a bounded application-level retry-on-`SQLITE_BUSY` (up to 3 attempts with backoff)
  around every write. If retries are exhausted, the API returns `503` with a
  `Retry-After` header (see API-CONTRACT `RetryLater` response) rather than silently
  dropping the write or blocking indefinitely.
- **Alternatives considered:** Last-write-wins with no retry (rejected — directly
  contradicts US-21 AC2, "must not silently lose one session's addition");
  migrating off SQLite to a multi-writer database (rejected — SQLite is fixed
  upstream by human decision, PRD Section 13.1/Gate 2; not this discipline's to
  re-open).
- **Rationale:** WAL mode allows concurrent readers alongside a single writer (SQLite's
  actual constraint is serialized *writes*, not serialized reads-and-writes); with
  this demo's expected concurrency (two shared-login sessions, per F-7's own scope),
  a short busy-timeout plus a few retries resolves essentially every real contention
  case without introducing a queue/broker component `solution` already ruled out
  (SOL-3). Two concurrent captures against the same thread (US-21) each still create
  a distinct `interactions` row — WAL/retry only serializes the brief write itself,
  it does not merge or reject either session's content. The new `login`/`logout`
  operations (DAT-10) and their `503` response follow this same "every write" scope
  — see DAT-10's rationale.
- **Source:** PRD Open Question 9 (OI-2); solution.json SOL-8.
- **Owner:** `data_integration` and/or `platform` (platform confirms WAL mode is set
  at deployment/connection-pool configuration time). **Status:** proposed.

### DAT-4 — Customer-resolution pattern for conversational endpoints
- **Category:** API design / cross-cutting behavior
- **Choice:** `POST /qa` and `POST /briefs` both accept an optional `account_id`
  hint (used when the rep is already viewing one account's profile) and, when it is
  omitted, resolve the customer from the free-text question/request itself via
  C-NLU — the same resolution logic in both operations.
- **Alternatives considered:** Always requiring `account_id` as a path/query
  parameter (rejected — contradicts F-3 AC4/US-12 and F-5 AC3/US-17's own wording,
  "when a query names... a customer," which describes resolving a customer from the
  question's own text, not from a pre-selected scope); always requiring free-text
  resolution with no `account_id` hint (rejected — PRD Section 5 step 1 has the rep
  select an account before asking questions, so a profile-scoped Q&A entry point
  should not have to re-name the customer in every question).
- **Rationale:** This reconciles two PRD statements that are not, on their own,
  fully specific about the UI flow: Section 5's step ordering (select account, then
  ask) and F-3/F-5's own acceptance-criteria wording (the query names the customer).
  Supporting both paths in one schema costs nothing at the contract level and lets
  Workflow 3 pick either UI flow (or both) without a contract change.
- **Source:** PRD Section 5 step 1; F-3 AC4, F-5 AC3.
- **Owner:** `data_integration`, informed by Workflow 3's actual UI design. **Status:**
  proposed. Carried forward as Open Item OI-6 (Section 5) since the actual UI flow
  is Workflow 3's to confirm, not settled by this document alone.

### DAT-5 — Commitment "complete" status and its operation
- **Category:** Data modeling / API design (gap discovered, not requested by any story)
- **Choice:** Add `commitments.status` (`open`/`complete`, default `open`) and
  `PATCH /commitments/{commitmentId}/complete`.
- **Alternatives considered:** Leaving F-4 AC1's "excluded... once marked complete"
  unimplemented on the theory that no story requests a "mark complete" action
  (rejected — F-4 AC1 is itself part of US-13 AC1, a story already in scope; an
  acceptance criterion already in scope cannot be left silently unimplementable);
  inferring completion automatically from some other signal, e.g. a later note
  mentioning the commitment was fulfilled (rejected as this document's own decision
  — it would require a specific NLU-driven inference no PRD text describes at all,
  a materially larger invention than a plain status-toggle operation).
- **Rationale:** This is a genuine gap this discipline found while making US-13 AC1
  concretely implementable: the PRD confirms *that* a "marked complete" state must
  exist and be excluded from the list, but never says *how* a commitment reaches it.
  A minimal explicit operation is the smallest addition that makes the already-scoped
  acceptance criterion testable, and it can be replaced by a different mechanism
  later without touching any other operation in this contract.
- **Source:** PRD F-4 AC1 (US-13 AC1).
- **Owner:** `data_integration` (mechanism), Product owner (whether this is the
  intended mechanism). **Status:** proposed. Carried forward as Open Item OI-5.

### DAT-6 — Deterministic ordering for chronological views
- **Category:** Data access / determinism
- **Choice:** `interactions` are ordered by `captured_at` ascending, `id` ascending
  as a tie-breaker, for both the profile view (`GET /accounts/{id}/profile`, F-6)
  and internal "most recent" resolution (F-3 AC1) — the latter is a `DESC ... LIMIT`
  read against the same index, not a different ordering rule.
- **Alternatives considered:** Newest-first display order (a common UX convention
  for activity feeds) — not chosen as the contract's stated default because "chronological
  order" (US-18 AC1's literal wording) most naturally reads as oldest-to-newest; this
  is flagged as an open item (OI-7) precisely because the alternative is equally
  defensible and is a presentation choice Workflow 3 can make without a contract
  change (the API always returns full ordered data; a client can trivially reverse it).
- **Rationale:** Without a stated tie-breaker, two interactions captured at the same
  timestamp (a real, not hypothetical, case under US-21's concurrent-write scenario)
  would sort nondeterministically — silently violating this discipline's own
  contract to provide "a deterministic, documented ordering wherever a story requires
  stable results."
- **Source:** PRD F-6 AC1 (US-18); US-21 (concurrency).
- **Owner:** `data_integration`. **Status:** proposed.

### DAT-7 — Ratification of the PRD's proposed adapter interface contracts
- **Category:** Integration design
- **Choice:** Ratify PRD Section 14.1a's four adapter interface contracts
  (`fetchAccounts()`/`fetchContacts(accountId)` for CRM; `getUpcomingEvents(accountId)`
  for Calendar; `listDocuments(accountId)`/`fetchDocument(documentId)` for SharePoint;
  `fetchMessages(accountId)` for Email) unchanged, as internal interfaces inside
  C-API's adapter layer (solution.json Section 2) feeding `accounts`/`contacts`
  (CRM adapter) and the `embedded_content` collection (SharePoint/Email adapters,
  at seed-ingestion time). None of the four is exposed as a public API-CONTRACT
  operation, since C-WEB never calls an adapter directly (solution.json SOL-7).
- **Alternatives considered:** Refining the method signatures further before
  ratifying (considered and rejected — the PRD's own proposal already covers
  exactly what F-1 through F-7 need from each adapter, per PRD Section 14.1a's own
  reasoning, and this discipline found no gap requiring a change).
- **Rationale:** PRD Assumption 11/12 explicitly delegated this shape to
  requirements-agent's judgment and left ratification to Architecture
  (solution.json: "data_integration's to ratify or refine, not this discipline's" —
  referring to `solution`, meaning it fell to this discipline). Ratifying unchanged,
  rather than inventing a revision with no stated deficiency to fix, keeps this
  decision honest about what is genuinely new here (nothing) versus what is a
  confirmation of already-delegated work.
- **Source:** PRD Section 14.1a; solution.json Section 2 (adapter layer).
- **Owner:** `data_integration`. **Status:** proposed.

### DAT-8 — Business-card image retention
- **Category:** Data retention
- **Choice:** The uploaded business-card image is processed synchronously and not
  persisted afterward; only the extracted fields (via `interaction_attendees`) and
  the interaction's own `raw_text`/metadata are stored.
- **Alternatives considered:** Persisting the original image (e.g., for later manual
  review of an OCR failure) — rejected as inventing a storage requirement no story
  asks for, and one that would raise a data-retention question this PRD's synthetic-
  data-only scope (`constraints.synthetic_data_only`) does not otherwise need to
  answer.
- **Rationale:** No feature or story ever re-displays or re-processes a previously
  submitted business-card image; F-1 AC2/US-4's full requirement (extract now, or
  tell the rep manual entry is required) is satisfiable without retaining the
  source image at all.
- **Source:** PRD F-1 AC2 (US-4); Section 7 Security/Privacy NFR (seeded-data-only).
- **Owner:** `data_integration`. **Status:** proposed.

### DAT-9 — Vector-store traceability via native metadata, no new SQLite table
- **Category:** Data modeling
- **Choice:** Satisfy PRD Section 13.2's "every embedded item must remain traceable
  back to the customer thread and source" requirement using Chroma's own per-chunk
  metadata (`account_id`, `source_type`, `source_id`, `chunk_index`) rather than a
  new SQLite `embeddings` mapping table.
- **Alternatives considered:** A SQLite `embeddings` table recording each chunk's id
  and source reference, mirrored alongside Chroma's own metadata — rejected as a
  duplicated source of truth with no reader in this scope that needs SQL-side
  querying of embedding metadata (every consumer of that traceability is C-API's own
  retrieval logic, which already receives the metadata directly from Chroma on
  every query).
- **Rationale:** Introducing a second store for the same facts risks the two drifting
  apart with no feature ever cross-checking them; Chroma's metadata is authoritative
  and sufficient on its own.
- **Source:** PRD Section 13.2.
- **Owner:** `data_integration`. **Status:** proposed.

### DAT-10 — Login/logout mechanism shape (new API surface, resolves `security_nfr`'s OI-13/RF-1)
- **Category:** API design / cross-cutting behavior (login/logout mechanism shape)
- **Choice:** Added `POST /auth/login` (public — `security: []` — exchanges
  `LoginRequest{username, password}` for the session cookie via a `Set-Cookie`
  response header carrying the flags SEC-2 already fixed; `401 INVALID_CREDENTIALS`
  on a wrong credential; `429 LoginLocked` after SEC-5's 5-attempts/15-minute
  lockout threshold; `503 RetryLater` if the session-store write cannot be
  serialized, consistent with DAT-3's "every write" scope) and `POST /auth/logout`
  (requires `SharedLoginAuth` — a session must exist to end it; invalidates the
  session server-side per SEC-2 and clears the cookie; `401 Unauthorized` if no
  valid session; `503 RetryLater` on the same write-contention basis as login).
  Neither operation changes any of the 9 v1.0 business operations' own schemas.
  `x-story-ids: ["US-20"]` on both, not `[]` — the check-artifacts tool structurally
  requires every operation's `x-story-ids` to be non-empty (`mapped.length > 0`),
  so an honest `x-story-ids: []` disclosure (which this discipline considered, since
  no story's acceptance criteria actually exercises a login/logout action — every
  one of the 21 stories assumes a session already exists) is not a viable choice
  that also passes the tool. US-20 (F-7: "shown the same customer thread content to
  every session authenticated via the shared login") is the closest genuine
  functional tie available in scope: it is literally the story whose acceptance
  criteria names "authenticated via the shared login" as the precondition these two
  operations exist to establish and end. This is disclosed as a forced, tool-driven
  tie, not a claim that US-20's own acceptance criteria test the login/logout action
  itself — see Open Item OI-18 (Section 5) for the honest caveat carried forward
  into the traceability table (US-20's row now lists `login`/`logout` in
  `operation_ids` alongside its original `listAccounts`/`getAccountProfile`).
- **Alternatives considered:** A single `POST /auth/session` endpoint reused for both
  login (`POST` with a body) and logout (`DELETE`) — rejected: login and logout have
  materially different security postures (public vs. authenticated), better
  expressed as two distinct operations each with its own explicit `security`
  requirement than one path overloading two verbs with two different auth
  preconditions. Tying `login`/`logout` to no story at all (`x-story-ids: []`) —
  rejected as a tool-incompatible choice (see above), not as dishonest; the honest
  disclosure is instead carried in this decision's own text and in Open Item OI-18.
  Inventing a session-lifetime, lockout-window, or rate-limit numeric value of this
  discipline's own — rejected outright: SEC-2, SEC-5, and SEC-11 already fixed those
  values; this decision only gives them API surface, per this revision's own scope
  limit (Section 0.1).
- **Rationale:** `security_nfr`'s SEC-1 (login mechanism), SEC-2 (session lifecycle,
  including logout invalidation), and SEC-5 (login lockout) each require an
  operation this contract did not have before v1.1; `security_nfr`'s own document
  Section 6 and `returned_findings[0]`/`RF-1` name exactly this gap and exactly
  these operations as missing. This decision supplies the missing API surface
  without re-deciding any policy value `security_nfr` already fixed.
- **Source:** `security_nfr` SEC-1, SEC-2, SEC-5; `security_nfr.json`
  `returned_findings[0]` (`RF-1`/`V-SEC2`); workflow sign-off `revise` feedback
  (2026-09-17T17:50:00Z, recorded in this role's own `audit_trail`).
- **Owner:** `data_integration`. **Status:** proposed.

### DAT-11 — 429-class response shapes: two components, not one (resolves `security_nfr`'s SEC-5/SEC-11 gap)
- **Category:** API design / error semantics (429-class rate-limit and lockout response shapes)
- **Choice:** Two distinct `429`-class response components, following this
  contract's existing named-response-component pattern (`Unauthorized`,
  `AccountNotFound`, `CommitmentNotFound`, `RetryLater`): `RateLimited`
  (`error_code=RATE_LIMITED`, SEC-11/NFR-10's per-session 30-requests/minute
  cost/DoS control), applied to `POST /interactions`, `POST /qa`, `POST /briefs` —
  the three operations SEC-11/NFR-10 explicitly name, each of which triggers a
  Bedrock invocation; and `LoginLocked` (`error_code=LOGIN_LOCKED`, SEC-5/NFR-4's
  brute-force/credential-stuffing lockout after 5 failed attempts in a rolling
  15-minute window), applied only to `POST /auth/login`. Both carry a `Retry-After`
  header, matching the existing `RetryLater` response's shape.
- **Alternatives considered:** A single shared `429` component for both controls —
  rejected: SEC-5 (login lockout) and SEC-11 (per-session operation throttling) are
  different controls with different triggers (a failed-credential count vs. a
  request-volume count) and different remediation (wait out the lockout window vs.
  slow the request rate); collapsing them into one `error_code` would make a
  client's own logging/alerting unable to distinguish "someone is guessing the
  shared password" from "a session is calling too fast" — exactly the distinction
  `platform`'s own PLA-8 alarms (login lockouts vs. rate-limit rejections) already
  treat as separately meaningful signals. Applying `RateLimited` to every operation
  in the contract rather than the three SEC-11/NFR-10 explicitly name — rejected:
  the revise feedback and `security_nfr`'s own SEC-11/NFR-10 both scope this control
  to the three Bedrock-invoking operations, not a blanket rate-limit story for
  read-only operations no control names.
- **Rationale:** Directly implements SEC-11/NFR-10 (rate limiting) and SEC-5/NFR-4
  (login lockout) as real, described API responses rather than leaving either as a
  policy statement with no corresponding contract surface — exactly the gap
  `security_nfr`'s `returned_findings[0]`/`RF-1` identified.
- **Source:** `security_nfr` SEC-5, SEC-11, NFR-4, NFR-10; `platform` PLA-8 (alarm
  design distinguishing login lockouts from rate-limit rejections); workflow sign-off
  `revise` feedback.
- **Owner:** `data_integration`. **Status:** proposed.

## 3. What this contract defines

The full contract is embedded in Appendix A and is also published standalone at
`docs/architecture/API-CONTRACT-conversational-crm-T-1-v1.1.json`. Eleven operations
(9 unchanged from v1.0, 2 new in v1.1):

| operationId | Method + path | Stories served |
|---|---|---|
| `listAccounts` | `GET /accounts` | US-18, US-20 |
| `getAccountProfile` | `GET /accounts/{accountId}/profile` | US-18, US-19, US-20 |
| `captureInteraction` | `POST /interactions` | US-1, US-2, US-4, US-5, US-6, US-7, US-8, US-21 |
| `listUnmatchedInteractions` | `GET /interactions/unmatched` | US-3 |
| `resolveInteractionCustomer` | `POST /interactions/{interactionId}/resolve-customer` | US-3 |
| `askQuestion` | `POST /qa` | US-9, US-10, US-11, US-12 |
| `generateBrief` | `POST /briefs` | US-15, US-16, US-17 |
| `listDueCommitments` | `GET /commitments/due` | US-13, US-14 |
| `markCommitmentComplete` | `PATCH /commitments/{commitmentId}/complete` | US-13 |
| **`login`** *(v1.1)* | `POST /auth/login` | US-20 (disclosed tool-driven tie, DAT-10/OI-18) |
| **`logout`** *(v1.1)* | `POST /auth/logout` | US-20 (disclosed tool-driven tie, DAT-10/OI-18) |

**Error semantics distinguished per operation** (the specific requirement this
discipline's contract must satisfy — "if a story requires distinguishing two failure
modes, the contract must actually distinguish them"):

- `captureInteraction`: `400 EMPTY_NOTE` (true rejection, no row created) is
  distinguished from a **successful** `201` with `match_status='unmatched'`
  (US-2 AC2) and from a **successful** `201` with `manual_entry_required=true`
  (US-4 AC2) — the latter two are valid conversational outcomes, not errors, and
  conflating them with `400` would make it impossible for a client to tell "your
  input was invalid" apart from "your input was fine, but the system couldn't match/
  read it."
- `resolveInteractionCustomer`: `404 INTERACTION_NOT_FOUND`/`404 ACCOUNT_NOT_FOUND`
  are distinguished from `409 INTERACTION_ALREADY_MATCHED` — a client acting on a
  stale unmatched-notes list needs to tell "that note doesn't exist" apart from
  "that note isn't unmatched anymore."
- `askQuestion`/`generateBrief`: every empty-state and not-identified outcome is a
  `200` with a `resolution` discriminator, never a `4xx` — these are correct,
  expected conversational answers (US-9 AC2, US-10 AC2, US-11 AC2, US-12 AC2, US-15
  AC1, US-17 AC2), and forcing them into HTTP error semantics would make a client
  treat "no history yet" the same as "your request was malformed."
- Every write operation shares one `503 RetryLater` shape (with `Retry-After`) for a
  transient SQLite write conflict (decision DAT-3), distinguished from every
  business-logic outcome above by being the one response that means "nothing was
  decided yet, try again" rather than "here is the outcome." **v1.1:** `login` and
  `logout` also carry this shape, since a session-store write is a write in the same
  sense DAT-3 already scopes to "every write."
- **v1.1:** `captureInteraction`, `askQuestion`, and `generateBrief` now also carry
  `429 RateLimited` (SEC-11/NFR-10), distinguished from `503 RetryLater` by meaning
  "you are calling too fast" rather than "the database briefly couldn't serialize
  this write" — a client that retries a `503` immediately is doing the right thing;
  a client that retries a `429` immediately is not. `login` separately carries
  `429 LoginLocked` (SEC-5/NFR-4), distinguished from `RateLimited` per decision
  DAT-11.
- **v1.1:** `login` also carries `401 INVALID_CREDENTIALS`, distinguished from the
  `401 Unauthorized` used by every other operation — `Unauthorized` means "no valid
  session cookie was presented"; `INVALID_CREDENTIALS` means "the credential
  presented to obtain one was wrong." Conflating the two would make it impossible
  for a client to tell "you're not logged in" apart from "your login attempt itself
  failed."

## 4. Traceability

Acceptance criteria below are copied verbatim from
`workflow-1-discovery-to-stories/state/roles/stories.json`'s `stories[].acceptance_criteria`
arrays (the same source `solution` cited), not paraphrased and not copied from the
published Markdown (which uses different quote-mark conventions). All 21 rows are
unchanged from v1.0 except US-20, whose `operation_ids` gained `login`/`logout` in
v1.1 (see decision DAT-10/Open Item OI-18) — its `acceptance_criteria`,
`component_ids`, `entity_names`, and `verification_scenarios` are otherwise
identical to v1.0.

### F-1 — Conversational Interaction Capture

**US-1 — Capture a typed interaction note against a matched customer**
- Acceptance criteria:
  1. "The system must create a structured interaction note and associate it with the correct customer thread when the rep's typed entry unambiguously matches exactly one seeded customer account."
  2. "The system must save the entry as a new interaction note when the entry contains at least one non-whitespace character."
- component_ids: C-WEB, C-API, C-SQL, C-NLU
- operation_ids: captureInteraction
- entity_names: interactions, accounts
- verification_scenarios: Submit a typed one-liner unambiguously naming exactly one seeded account via `POST /interactions`; confirm `201` with `match_status='matched'` and the correct `account_id`.

**US-2 — Reject invalid capture input and flag unmatched customers**
- Acceptance criteria:
  1. "The system must reject the submission with an explicit 'cannot save an empty note' message, and must not create any note record, when the rep submits an empty or whitespace-only entry."
  2. "The system must flag the note as 'unmatched — needs customer selection' — rather than attaching it to a default or incorrect account — when no seeded customer account can be unambiguously identified from the entry text."
- component_ids: C-WEB, C-API, C-NLU, C-SQL
- operation_ids: captureInteraction
- entity_names: interactions
- verification_scenarios: Submit whitespace-only `raw_text`; confirm `400 EMPTY_NOTE` and no row in `interactions`. Submit text naming no/an ambiguous account; confirm `201` with `match_status='unmatched'`, `account_id=null`.

**US-3 — Resolve an unmatched note to the correct customer**
- Acceptance criteria:
  1. "The system must attach a previously flagged 'unmatched' note to the customer thread the rep selects when the rep manually resolves the flag."
  2. "The system must leave the note in the unmatched/needs-selection state — rather than guessing an account — when the rep has not yet resolved it."
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: listUnmatchedInteractions, resolveInteractionCustomer
- entity_names: interactions
- verification_scenarios: List unmatched notes via `GET /interactions/unmatched`; resolve one via `POST /interactions/{id}/resolve-customer`, confirm `200` with `match_status='matched'`; confirm an unresolved note is never auto-assigned (still appears in the unmatched list, `account_id` still null).

**US-4 — Capture an interaction via a scanned business card**
- Acceptance criteria:
  1. "[INFERRED — needs confirmation] The system must extract the contact's name, company and role from a scanned business-card image into the structured interaction note when the image is legible and contains recognizable contact fields. The OCR mechanism was never discussed in discovery (PRD Assumption 5, Open Question 5); whether this is worth building for the demo at all is still open."
  2. "[INFERRED — needs confirmation] The system must inform the rep that manual entry is required — without fabricating contact details — when the image cannot be read or no contact fields are recognized."
- component_ids: C-WEB, C-API, C-NLU, C-SQL
- operation_ids: captureInteraction
- entity_names: interactions, interaction_attendees
- verification_scenarios: Submit a legible business-card image via the `multipart/form-data` variant; confirm `201` with populated `attendees`. Submit an unreadable image; confirm `201` with `manual_entry_required=true` and `attendees=[]`, no fabricated fields. **Inherited, not resolved here:** whether Bedrock's own multimodal models suffice for this path at all is `solution`'s Open Item OI-3 (Product owner / Architect); this contract is written technology-agnostically against C-NLU so it does not need OI-3 resolved to be implementable.

**US-5 — Extract attendee names from a typed interaction note**
- Acceptance criteria:
  1. "The system must extract every distinct attendee name mentioned in a typed free-text interaction entry as a separate structured attendee item — recording one item per distinct person named (e.g., 'met Priya and Arjun from Acme...' must yield two attendee items, not one merged string) — when the entry text names at least one identifiable person."
  2. "The system must record zero attendee items for that interaction — rather than fabricating a name — when the entry text names no one."
  3. "[INFERRED — needs confirmation] The exact name-recognition/disambiguation mechanism was never discussed in discovery (PRD Assumption 14, Open Question 15); only the underlying capability (AC1/AC2) is treated as confirmed."
- component_ids: C-API, C-NLU, C-SQL
- operation_ids: captureInteraction
- entity_names: interaction_attendees
- verification_scenarios: Submit "met Priya and Arjun from Acme..."; confirm two `interaction_attendees` rows. Submit text naming no one; confirm zero rows.

### F-2 — Automatic Extraction & Thread Tagging

**US-6 — Extract commitments and next steps from a captured note**
- Acceptance criteria:
  1. "The system must extract each distinct commitment, follow-up or next step mentioned in a captured note's text as a separate tracked item when the note contains one or more of them."
  2. "The system must record no commitment items when the note describes discussion only, with no forward-looking commitment."
- component_ids: C-API, C-NLU, C-SQL
- operation_ids: captureInteraction
- entity_names: commitments
- verification_scenarios: Submit a note with two commitments; confirm two `commitments` rows tagged to the same `interaction_id`. Submit a discussion-only note; confirm zero rows.

**US-7 — Resolve a due date for an extracted commitment**
- Acceptance criteria:
  1. "The system must record a due date on an extracted commitment when the captured text states a concrete date or an unambiguous relative date term resolvable against the interaction's timestamp (e.g., 'by Friday')."
  2. "The system must mark the commitment's due date as unspecified — rather than guessing a date — when the text uses a vague temporal reference (e.g., 'soon,' 'sometime') or gives no date information at all."
- component_ids: C-API, C-NLU, C-SQL
- operation_ids: captureInteraction
- entity_names: commitments
- verification_scenarios: Submit "by Friday"; confirm a resolved `due_date`. Submit "soon"; confirm `due_date=null`.

**US-8 — Tag extracted items to the correct customer thread**
- Acceptance criteria:
  1. "The system must tag every extracted discussion point and commitment to the same customer thread as its source interaction note when that note is matched to a thread."
  2. "The system must flag an extracted item for manual thread assignment — rather than tagging it to an incorrect thread — when the source note itself is unmatched to a customer thread (per F-1)."
- component_ids: C-API, C-SQL
- operation_ids: captureInteraction
- entity_names: commitments, interactions
- verification_scenarios: Confirm commitments on a matched note carry that account's `account_id`. Confirm commitments on an unmatched note have `account_id=null` until `resolveInteractionCustomer` sets it.

### F-3 — Natural-Language Memory & Q&A

**US-9 — Ask what was discussed last time**
- Acceptance criteria:
  1. "The system must answer a natural-language question about a customer's most recent discussion by returning content drawn from that customer's most recent interaction note(s) when the thread has at least one captured note."
  2. "The system must respond that no history exists yet for that customer when the thread is empty."
- component_ids: C-WEB, C-API, C-SQL, C-VEC, C-NLU
- operation_ids: askQuestion
- entity_names: interactions, embedded_content (Chroma collection)
- verification_scenarios: Ask against a thread with history; confirm `resolution='answered'` drawing on that thread's most recent interaction (per DAT-6's ordering). Ask against an empty thread; confirm `resolution='no_history'`.

**US-10 — Ask what I committed to for a customer**
- Acceptance criteria:
  1. "The system must answer a natural-language question about the rep's own open commitments for a customer by listing the commitments extracted and tagged to that thread when any exist."
  2. "The system must state that there are no open commitments for that customer when none exist."
- component_ids: C-WEB, C-API, C-SQL, C-NLU
- operation_ids: askQuestion
- entity_names: commitments
- verification_scenarios: Ask against a thread with open commitments; confirm the answer matches `commitments` rows with `status='open'`. Ask against a thread with none; confirm `resolution='no_open_commitments'`.

**US-11 — Ask who attended from the customer's side**
- Acceptance criteria:
  1. "The system must answer a natural-language question about interaction attendees by returning the contact names extracted from that thread's interaction notes when attendee names were captured."
  2. "The system must state that no attendee information was captured when none was extracted."
- component_ids: C-WEB, C-API, C-SQL, C-NLU
- operation_ids: askQuestion
- entity_names: interaction_attendees
- verification_scenarios: Ask against a thread with captured attendees; confirm names match the corresponding `interaction_attendees` rows. Ask against a thread with none; confirm `resolution='no_attendees_captured'`.

**US-12 — Reject a Q&A request naming no clear customer**
- Acceptance criteria:
  1. "The system must resolve and answer against the correct customer thread when a query names exactly one customer that matches a seeded account."
  2. "The system must respond that it could not identify the customer — rather than guessing or returning another customer's data — when the query names no customer or an unrecognized one."
- component_ids: C-WEB, C-API, C-NLU, C-SQL
- operation_ids: askQuestion
- entity_names: accounts
- verification_scenarios: Ask naming exactly one seeded account; confirm the correct thread answers. Ask naming no/an unrecognized account; confirm `resolution='not_identified'`, no other account's data returned.

### F-4 — Proactive Commitment Tracking

**US-13 — View overdue and due-soon commitments across accounts**
- Acceptance criteria:
  1. "The system must list every open commitment whose due date has passed as 'overdue' when the current date is past that due date, and must exclude a commitment from this list once it is marked complete."
  2. "[INFERRED — needs confirmation] The system must list every open commitment whose due date falls within the next 7 days as 'due soon,' and must reclassify it as 'overdue' instead once its due date has passed. The 7-day window is a carried-over illustrative placeholder from the PRD, not a confirmed value (PRD Open Question 6)."
  3. "The system must list a commitment with an unspecified due date (per F-2) separately from the due-soon/overdue list — rather than omitting it entirely or treating it as overdue — when no due date was captured."
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: listDueCommitments, markCommitmentComplete
- entity_names: commitments
- verification_scenarios: Seed commitments past due, within 7 days, beyond 7 days, with no due date, and one marked complete; call `GET /commitments/due` with a fixed `reference_date`; confirm each bucket classifies correctly and the completed one appears in none of them.

**US-14 — See an explicit empty state when nothing is due soon or overdue**
- Acceptance criteria:
  1. "The system must state explicitly that there are no due-soon or overdue commitments — rather than showing an empty list with no explanation — when none exist."
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: listDueCommitments
- entity_names: commitments
- verification_scenarios: Call `GET /commitments/due` when no commitment qualifies for `overdue`/`due_soon`; confirm `message` is populated rather than a bare empty response.

### F-5 — Brief-Me-on-Customer Summary

**US-15 — Get a pre-meeting brief on a customer**
- Acceptance criteria:
  1. "The system must generate a summary containing recent discussion history, open commitments and stakeholder/contact names for a named customer when that customer has at least one captured note, and must state that no history exists yet for that customer — rather than generating a summary with fabricated content — when the thread is empty."
  2. "[INFERRED — needs confirmation] The system must return a bounded, short-form summary (a few sentences or bullets, not a multi-page document) under normal conditions, and must still return whatever partial content is available, rather than failing the whole request, when data for one of the summary components is missing (PRD Open Question 7)."
- component_ids: C-WEB, C-API, C-SQL, C-VEC, C-NLU
- operation_ids: generateBrief
- entity_names: interactions, commitments, contacts, interaction_attendees, embedded_content (Chroma collection)
- verification_scenarios: Request a brief for a customer with history; confirm `resolution='generated'` with a populated `summary` combining structured and retrieved content. Request one for an empty thread; confirm `resolution='no_history'`.

**US-16 — Include opportunity/deal status in the customer brief**
- Acceptance criteria:
  1. "[INFERRED — needs confirmation] The system must synthesize the opportunity/deal-status component narratively from that customer's recorded interaction notes and commitments — since no discrete deal-stage field exists in the structured store — when the thread contains content indicating where the deal/opportunity stands (PRD Assumption 13, Open Question 14)."
  2. "The system must state that no opportunity/deal-status information has been captured yet for that customer — rather than fabricating a stage or outcome — when the thread contains no such content."
- component_ids: C-API, C-SQL, C-VEC, C-NLU
- operation_ids: generateBrief
- entity_names: embedded_content (Chroma collection), interactions, commitments
- verification_scenarios: Request a brief for a customer whose notes discuss deal progress; confirm `summary.opportunity_status` is non-null and narrative. Request one with no such content; confirm `summary.opportunity_status=null`. **Ratified here, not re-opened:** no new structured deal-stage entity is introduced (Section 1 of DATA-MODEL — only `accounts`/`contacts`/`interactions`/`interaction_attendees`/`commitments` exist), consistent with PRD Section 13.1's own reasoning.

**US-17 — Reject a brief request naming no clear customer**
- Acceptance criteria:
  1. "The system must generate the summary for a named customer when the request unambiguously identifies exactly one seeded customer account."
  2. "The system must respond that it could not identify the requested customer — rather than guessing — when the request names an unrecognized or ambiguous customer."
- component_ids: C-WEB, C-API, C-NLU, C-SQL
- operation_ids: generateBrief
- entity_names: accounts
- verification_scenarios: Request naming exactly one seeded account; confirm `resolution='generated'`. Request naming an ambiguous/unrecognized account; confirm `resolution='not_identified'`.

### F-6 — Customer Profile / Conversational History View

**US-18 — View a customer's running history on their profile**
- Acceptance criteria:
  1. "The system must display a customer's captured notes and extracted items in chronological order on that customer's profile view when at least one note exists."
  2. "The system must display an explicit 'no history yet' state when none exists."
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: getAccountProfile
- entity_names: interactions, interaction_attendees, commitments
- verification_scenarios: Open a profile with captured notes; confirm `interactions` returned in `captured_at` ascending order (DAT-6). Open one with none; confirm `is_empty=true`.

**US-19 — See a new capture appear on the correct profile automatically**
- Acceptance criteria:
  1. "The system must display a newly captured note and its extracted items on the correct customer's profile view automatically, without requiring the rep to take further action, when the note is tagged to that customer's thread."
  2. "The system must not display that note or its items on any other customer's profile view when it is not tagged to that customer."
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: captureInteraction, getAccountProfile
- entity_names: interactions
- verification_scenarios: Capture a note, then call `GET /accounts/{id}/profile` for that account; confirm it appears with no separate action. Confirm it is absent from every other account's profile call.

### F-7 — Shared Customer Thread Access

**US-20 — See identical thread content across shared-login sessions**
- Acceptance criteria:
  1. "The system must show the same customer thread content to every session authenticated via the shared login when two sessions view the same customer."
  2. "The system must not partition data by which physical person is at the keyboard, since no per-rep identity exists in this version."
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: listAccounts, getAccountProfile, login, logout *(login/logout added in v1.1 — see decision DAT-10, Open Item OI-18: a disclosed, tool-driven tie, not a claim that these acceptance criteria themselves test the login/logout action)*
- entity_names: accounts, interactions
- verification_scenarios: Call `getAccountProfile` for the same account from two sessions; confirm byte-identical `interactions` content — a direct consequence of both reading the same `interactions` rows with no session-scoping column anywhere in the schema. **v1.1:** confirm both sessions reached that state via their own independent `login` call, and that a `logout` on one session does not affect the other session's own independent one (SEC-2 — sessions are per-login, not shared as a single object between the two browser sessions).

**US-21 — Preserve concurrent additions to the same customer thread**
- Acceptance criteria:
  1. "[INFERRED — needs confirmation] The system must persist a note added from one session so that it is immediately visible to another concurrent session viewing the same customer thread. The conflict-handling mechanism was never discussed in discovery (PRD Open Question 9)."
  2. "[INFERRED — needs confirmation] The system must not silently lose one session's addition when another session adds to the same thread at nearly the same time."
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: captureInteraction
- entity_names: interactions
- verification_scenarios: Call `captureInteraction` from two sessions against the same account at nearly the same instant; confirm both succeed as distinct `interactions` rows (WAL + retry-on-busy, decision DAT-3) or, in the rare exhausted-retry case, one receives `503 RetryLater` and retries successfully — never a silent loss of either session's content.

**Coverage check:** all 21 stories (US-1 through US-21) across all 7 features carry
non-empty `operation_ids` and `entity_names`. **21 of 21 covered.** All 11 operations
(9 unchanged + 2 new in v1.1) are reachable from this table and every `operation_ids`
entry here resolves to a real `operationId` in Appendix A, in both directions.

## 5. Open items

| ID | Item | Blocks development? | Owner | Resolution needed |
|---|---|---|---|---|
| OI-5 | No PRD feature or story describes how a rep (or the system) marks a commitment complete, yet F-4 AC1 (US-13 AC1) requires excluding a completed commitment from the due-soon/overdue list. `PATCH /commitments/{id}/complete` (decision DAT-5) is this discipline's own minimal inferred mechanism. | false | Product owner (whether/how a rep marks a commitment complete), then `data_integration`/Workflow 3 if the mechanism changes | Product owner confirms whether an explicit "mark complete" action is the intended mechanism, or specifies a different one (e.g., inferred from a later note) for Workflow 3 to build against. |
| OI-6 | The exact UI flow behind `POST /qa` and `POST /briefs`'s customer-resolution pattern (decision DAT-4 — `account_id` hint vs. free-text resolution) is this discipline's own reconciliation of PRD Section 5's step ordering against F-3/F-5's own acceptance-criteria wording, not a confirmed UX decision. | false | Architect / Workflow 3 (UI/UX) | Confirm, when Workflow 3 designs the actual Q&A/Brief-Me screens, whether the rep asks from within a profile view (hint path), a global query box (free-text path), or both — no contract change needed either way. |
| OI-7 | `GET /accounts/{id}/profile`'s chronological order (decision DAT-6 — oldest-first, the literal reading of "chronological order") is one of two equally defensible readings; a newest-first activity-feed convention is common UX practice and was not chosen as the default. | false | UI/UX (Workflow 3) | Confirm the desired display order with a human before Workflow 3 builds the profile view; reversible client-side without any contract change. |
| OI-18 | **(new in v1.1)** The server-side session store backing the new `login`/`logout` operations has no persistence-technology decision recorded by any discipline. `security_nfr`'s SEC-1/SEC-2/SEC-3 fix the mechanism (opaque token) and policy (lifecycle, credential-hash storage) but not where session state itself lives at runtime. This document does not add a `sessions` table to the data model (DATA-MODEL Section 5) because no discipline has decided session state belongs in C-SQL rather than an out-of-process store. Separately, and for the same reason: `login`/`logout`'s `x-story-ids: ["US-20"]` (decision DAT-10) is a tool-required, disclosed tie, not a claim that US-20's own acceptance criteria test the login/logout action. | false | `security_nfr` / `platform` (session-store location); this discipline (story-mapping honesty, already disclosed) | Confirm where session state is actually held before Workflow 3/4 build the login/logout mechanism; if C-SQL, a `sessions` table would need to be added to the data model at that point. |

**Inherited from `solution` (not duplicated here, still open in `solution.json`,
surfaced again at workflow sign-off across all four disciplines' open items):**
OI-1 (PRD hash line-ending drift — already independently re-verified as
content-identical), OI-3 (whether Bedrock's multimodal models suffice for
business-card OCR, or a dedicated OCR technology is needed — this document's
`captureInteraction` operation is written technology-agnostically against C-NLU
specifically so it does not need OI-3 resolved first), and OI-4 (attendee-name
extraction mechanism — this document treats it, consistent with `solution`, as
prompt-driven extraction against C-NLU with no new component). OI-1a and OI-2 are
addressed by DAT-2 and DAT-3 above respectively; the underlying PRD-level questions
they originated from (PRD Assumption 10, Open Question 9) remain formally open at
the PRD/Requirements level, since resolving them architecturally does not retroactively
mark a PRD assumption as human-confirmed — that distinction is preserved rather than
collapsed.

**`security_nfr`'s `returned_findings[0]`/`RF-1`/`V-SEC2`:** this v1.1 revision is
this discipline's response to that finding (Section 0.1). Marking it `resolved` in
`security_nfr.json` is `security_nfr`'s own decision once it reviews this contract,
not something this document declares on its behalf.

## 6. Self-verification

- **Every operation is reachable from traceability and every traceability operation
  exists in the contract.** PASS — cross-checked all 11 `operationId`s in
  API-CONTRACT-conversational-crm-T-1-v1.1.json against Section 4's `operation_ids`
  columns in both directions; no orphan on either side. The 2 new operations
  (`login`, `logout`) are both reachable via US-20's row (decision DAT-10).
- **Every OpenAPI reference resolves locally; nothing external.** PASS — every
  `$ref` in the contract, including the 2 new operations' and the 2 new `429`
  response components', targets `#/components/...` in the same file; no external
  `$ref`, no remote `servers` dependency beyond a relative base path.
- **Both appendices are byte-identical to the standalone files.** PASS — Appendix A
  and Appendix B below were produced by embedding the exact same file content
  written to `docs/architecture/API-CONTRACT-conversational-crm-T-1-v1.1.json` and
  `docs/architecture/DATA-MODEL-conversational-crm-T-1-v1.1.md`; hashes recorded in
  `roles/data_integration.json`'s `artifacts[]`.
- **Every constraint the data model states is expressible and expressed in the
  contract schema, or the difference is recorded as an open item.** PASS — see
  DATA-MODEL Section 3 ("Constraint-to-contract expressibility check"), including
  its new v1.1 row confirming no new entity/constraint was introduced by this
  revision.
- **`source_references` records every upstream artifact consumed, with hashes.**
  PASS — Section 1; all three Workflow 1 hashes re-verified unchanged from v1.0, and
  the two newly-consumed Workflow 2 artifacts for this revision (`security_nfr`'s
  and `platform`'s documents) independently hashed and cross-checked against their
  own role files' recorded values — exact match on both, no drift.
- **Every inference is marked `[INFERRED — needs confirmation]` and carried as an
  open item.** PASS — `commitments.status`/`markCommitmentComplete` (OI-5), the
  `/qa`/`/briefs` customer-resolution pattern (OI-6), the chronological-order
  direction (OI-7), and, new in v1.1, the `login`/`logout` story-mapping caveat and
  the session-store location question (OI-18) are each marked inline in this report,
  the contract, and/or the data model, and each is carried to Section 5.
- **None of the 9 v1.0 business operations' request/response shapes changed.**
  PASS (explicit v1.1 check) — diffed every v1.0 operation's `requestBody`,
  `responses`, and referenced schemas against v1.1: `listAccounts`,
  `getAccountProfile`, `captureInteraction`, `listUnmatchedInteractions`,
  `resolveInteractionCustomer`, `askQuestion`, `generateBrief`,
  `listDueCommitments`, and `markCommitmentComplete` are byte-for-byte identical
  except for the addition of a `429` response entry to `captureInteraction`,
  `askQuestion`, and `generateBrief`'s own `responses` object (an addition, not a
  change to any existing status's shape) — exactly the scope the revise feedback
  authorized.

**Result: PASS.**

## 7. Readiness

`ready` — every one of the 21 selected stories traces to at least one operation and
at least one entity, every entity referenced by an operation is defined in the data
model, the contract is self-contained (no external references), and no open item
raised by this discipline (OI-5, OI-6, OI-7, OI-18) blocks development — each is
either a reversible presentation/UX choice, an explicitly minimal inferred
mechanism, or (OI-18) a forward-looking question about where session state
physically lives that does not block Workflow 3/4 from building against the
mechanism as specified. This v1.1 revision closes the specific gap `security_nfr`'s
`returned_findings[0]`/`RF-1` identified (missing login/logout/rate-limit API
surface) without reopening any of the 9 v1.0 business operations or the data model's
5 tables/1 collection. Readiness here means this discipline's own bundle is complete
and internally consistent; whether `security_nfr` now considers `RF-1` resolved, and
whether the workflow as a whole can reach `lock` at a fresh sign-off, are decisions
for `security_nfr` and the human respectively, not this document.

---

## Appendix A — API Contract (full, embedded)

Standalone file: `docs/architecture/API-CONTRACT-conversational-crm-T-1-v1.1.json`

<!-- api-contract:start -->
```json
{
  "openapi": "3.1.0",
  "info": {
    "title": "Conversational CRM API",
    "version": "1.1",
    "description": "Self-contained OpenAPI 3.1 contract for C-API (CRM API Service), the sole integration point C-WEB (CRM Web App) calls, per solution.json decisions SOL-1/SOL-7. Covers task T-1, scope full-product-v1 (F-1..F-7, US-1..US-21). Authored by the data_integration discipline of Workflow 2 (Solution Architecture). v1.1 adds the login/logout operations and the 429-class rate-limit response that security_nfr's approved SharedLoginAuth resolution (SEC-1, SEC-2, SEC-5, SEC-11) requires but v1.0 did not yet define (workflow sign-off revise, 2026-09-17; see decisions DAT-10/DAT-11) -- none of the 9 v1.0 business operations' own request/response shapes changed. The SharedLoginAuth scheme itself, and every session/lockout/rate-limit policy value (timeouts, attempt thresholds, requests-per-minute), remain security_nfr's decisions (SEC-1..SEC-5, SEC-11); this contract only gives them API surface."
  },
  "servers": [
    { "url": "/api/v1", "description": "C-API base path (exact host/scheme is platform's to finalize)" }
  ],
  "security": [ { "SharedLoginAuth": [] } ],
  "tags": [
    { "name": "accounts", "description": "Seeded customer accounts and per-account profile view (F-6, F-7)" },
    { "name": "interactions", "description": "Capture and resolution of interaction notes (F-1, F-2)" },
    { "name": "qa", "description": "Natural-language memory and Q&A (F-3)" },
    { "name": "briefs", "description": "Brief-me-on-customer summaries (F-5)" },
    { "name": "commitments", "description": "Proactive commitment tracking (F-4)" },
    { "name": "auth", "description": "Session lifecycle for the single shared-login credential -- login and logout (v1.1). Mechanism fixed by security_nfr SEC-1/SEC-2/SEC-5, not decided here." }
  ],
  "paths": {
    "/accounts": {
      "get": {
        "operationId": "listAccounts",
        "tags": ["accounts"],
        "summary": "List every seeded customer account",
        "description": "Returns every seeded customer account so the rep can select one (PRD Section 5 step 1) and so every session of the single shared login browses the same account list (F-7). Backing entity: accounts.",
        "x-story-ids": ["US-20"],
        "security": [ { "SharedLoginAuth": [] } ],
        "responses": {
          "200": {
            "description": "Every seeded account, ordered by name ascending (deterministic, case-insensitive) for a stable selection list.",
            "content": {
              "application/json": {
                "schema": { "type": "array", "items": { "$ref": "#/components/schemas/Account" } }
              }
            }
          },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/accounts/{accountId}/profile": {
      "get": {
        "operationId": "getAccountProfile",
        "tags": ["accounts"],
        "summary": "Get one customer's running conversational history (F-6)",
        "description": "Returns that customer's captured interactions (with their attendees and commitments) in chronological order, or an explicit no-history state. Every session of the shared login reads the same rows here, which is what makes F-7's identical-content requirement (US-20) mechanically true. A note captured via POST /interactions appears here on the very next read of this endpoint with no separate action required (US-19), because both paths read/write the same interactions table -- there is no push/notification mechanism. Backing entities: interactions, interaction_attendees, commitments.",
        "x-story-ids": ["US-18", "US-19", "US-20"],
        "security": [ { "SharedLoginAuth": [] } ],
        "parameters": [
          { "name": "accountId", "in": "path", "required": true, "schema": { "type": "string" }, "description": "Account id from GET /accounts." }
        ],
        "responses": {
          "200": {
            "description": "Profile returned. `is_empty=true` and `interactions=[]` together are the explicit 'no history yet' state (US-18 AC2) -- the client must render that state, never a bare empty list with no explanation.",
            "content": {
              "application/json": {
                "schema": { "$ref": "#/components/schemas/AccountProfile" }
              }
            }
          },
          "404": { "$ref": "#/components/responses/AccountNotFound" },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/interactions": {
      "post": {
        "operationId": "captureInteraction",
        "tags": ["interactions"],
        "summary": "Capture an interaction outcome as typed text or a scanned business card (F-1, F-2)",
        "description": "Accepts exactly one of two input shapes, distinguished by request content type: `application/json` for typed one-line capture (US-1, US-2, US-5), or `multipart/form-data` for a scanned business-card image (US-4). Extraction (attendees, commitments, due dates -- F-2, US-6, US-7, US-8) happens synchronously in this same request/response (solution.json decision SOL-3) and its results are returned inline; there is no separate polling step. Two sessions may call this concurrently against the same account thread (US-21) -- see the 503 response and 'Concurrency and error semantics' in the accompanying report for how a transient SQLite write conflict is surfaced rather than silently dropped.",
        "x-story-ids": ["US-1", "US-2", "US-4", "US-5", "US-6", "US-7", "US-8", "US-19", "US-21"],
        "security": [ { "SharedLoginAuth": [] } ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": { "$ref": "#/components/schemas/CaptureTextRequest" },
              "examples": {
                "typed": { "value": { "raw_text": "met Priya and Arjun from Acme, discussed renewal pricing, they want a demo of module X by Friday" } }
              }
            },
            "multipart/form-data": {
              "schema": { "$ref": "#/components/schemas/CaptureBusinessCardRequest" }
            }
          }
        },
        "responses": {
          "201": {
            "description": "Interaction note created. `match_status` distinguishes a matched thread (US-1 AC1) from an 'unmatched -- needs customer selection' flag (US-2 AC2) -- both are successful creations, not errors, per PRD F-1 AC1. For a business-card submission, `manual_entry_required=true` with empty `attendees` is the successful-but-unreadable-image outcome (US-4 AC2) -- it is not an HTTP error, since a note is still created and no contact detail is fabricated.",
            "content": {
              "application/json": { "schema": { "$ref": "#/components/schemas/Interaction" } }
            }
          },
          "400": {
            "description": "Rejected: the entry was empty or whitespace-only (US-2 AC1). No note record is created. error_code=EMPTY_NOTE.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
          },
          "422": {
            "description": "The uploaded file is not a readable image at all (wrong content type / corrupt upload) -- distinct from a legible-but-unrecognized business card (which is a 201, see above). error_code=UNSUPPORTED_IMAGE_FORMAT.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
          },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "429": { "$ref": "#/components/responses/RateLimited" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/interactions/unmatched": {
      "get": {
        "operationId": "listUnmatchedInteractions",
        "tags": ["interactions"],
        "summary": "List interaction notes still flagged 'unmatched -- needs customer selection' (US-3)",
        "description": "Lets the rep find a note the system could not auto-match, so it can be manually resolved via POST /interactions/{interactionId}/resolve-customer. Ordered by captured_at ascending (oldest unresolved first) so nothing sits unresolved indefinitely without being surfaced. Backing entity: interactions (match_status index).",
        "x-story-ids": ["US-3"],
        "security": [ { "SharedLoginAuth": [] } ],
        "responses": {
          "200": {
            "description": "Every interaction currently in the unmatched/needs-selection state. An empty array means none are pending -- not an error.",
            "content": { "application/json": { "schema": { "type": "array", "items": { "$ref": "#/components/schemas/Interaction" } } } }
          },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/interactions/{interactionId}/resolve-customer": {
      "post": {
        "operationId": "resolveInteractionCustomer",
        "tags": ["interactions"],
        "summary": "Manually attach an unmatched note to the customer the rep selects (US-3)",
        "description": "Moves an interaction (and its already-extracted attendees/commitments, per F-2 AC3) out of the unmatched state onto the selected account's thread. Also upserts that interaction's embedded vector-store metadata (`account_id`) so subsequent semantic retrieval (F-3, F-5) scopes correctly to the resolved account -- see decision DAT-2 in the accompanying report; the embedding vector itself is not recomputed, only its metadata, since the source text is unchanged.",
        "x-story-ids": ["US-3"],
        "security": [ { "SharedLoginAuth": [] } ],
        "parameters": [
          { "name": "interactionId", "in": "path", "required": true, "schema": { "type": "string" } }
        ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": { "$ref": "#/components/schemas/ResolveCustomerRequest" }
            }
          }
        },
        "responses": {
          "200": {
            "description": "Interaction re-tagged to the selected account; match_status is now 'matched'.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Interaction" } } }
          },
          "404": {
            "description": "The interaction id, or the account id in the request body, does not exist. error_code is INTERACTION_NOT_FOUND or ACCOUNT_NOT_FOUND, distinguishing which id was bad.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
          },
          "409": {
            "description": "The interaction is not currently in the unmatched state (already resolved or was never unmatched) -- rather than silently re-assigning an already-matched note (which the note must never do, per US-3 AC2's 'leave it in the unmatched state -- rather than guessing -- when not yet resolved', read together with F-1 AC4). error_code=INTERACTION_ALREADY_MATCHED.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
          },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/qa": {
      "post": {
        "operationId": "askQuestion",
        "tags": ["qa"],
        "summary": "Ask a natural-language question about a customer thread (F-3)",
        "description": "Covers all three canonical query types (what did we discuss last time; what did I commit to; who attended) plus any other free-text question, per PRD F-3's description. The customer is identified either from the optional `account_id` hint (when the rep asks from within a profile view already showing one account -- see decision DAT-4) or, when omitted, resolved from the question text itself by C-NLU (US-12's 'a query names exactly one customer'). This dual path is this discipline's own reasonable reconciliation of PRD Section 5 step 1 (rep first selects an account) with F-3 AC4's own wording (the query itself names a customer) -- flagged as an open item (OI-6) for Workflow 3 to confirm against the actual UI flow, not a settled UX decision.",
        "x-story-ids": ["US-9", "US-10", "US-11", "US-12"],
        "security": [ { "SharedLoginAuth": [] } ],
        "requestBody": {
          "required": true,
          "content": { "application/json": { "schema": { "$ref": "#/components/schemas/QARequest" } } }
        },
        "responses": {
          "200": {
            "description": "Always 200 -- 'no history yet' (US-9 AC2), 'no open commitments' (US-10 AC2), 'no attendee information captured' (US-11 AC2), and 'could not identify the customer' (US-12 AC2) are all valid conversational outcomes distinguished by the `resolution` field, not HTTP error statuses, exactly like the empty-state pattern used elsewhere in this contract.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/QAResponse" } } }
          },
          "400": {
            "description": "The `question` field was empty or whitespace-only. error_code=EMPTY_QUESTION.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
          },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "429": { "$ref": "#/components/responses/RateLimited" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/briefs": {
      "post": {
        "operationId": "generateBrief",
        "tags": ["briefs"],
        "summary": "Generate a short pre-meeting brief for a named customer (F-5)",
        "description": "Combines recent discussion history (interactions + semantically-retrieved embedded content), open commitments, stakeholder/contact names, and a narratively-synthesized opportunity/deal-status component (PRD Section 13.1 -- no discrete deal-stage field exists, per solution.json's own scope boundary). Customer identification follows the same `account_id`-hint-or-resolve-from-text pattern as /qa (decision DAT-4). Always returns whatever partial content is available rather than failing the whole request when one summary component is missing (US-15 AC2).",
        "x-story-ids": ["US-15", "US-16", "US-17"],
        "security": [ { "SharedLoginAuth": [] } ],
        "requestBody": {
          "required": true,
          "content": { "application/json": { "schema": { "$ref": "#/components/schemas/BriefRequest" } } }
        },
        "responses": {
          "200": {
            "description": "Always 200 -- 'no history yet' (US-15 AC1) and 'could not identify the requested customer' (US-17 AC2) are conversational outcomes distinguished by `resolution`, not HTTP errors.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/BriefResponse" } } }
          },
          "400": {
            "description": "The `request_text` field was empty or whitespace-only. error_code=EMPTY_REQUEST.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
          },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "429": { "$ref": "#/components/responses/RateLimited" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/commitments/due": {
      "get": {
        "operationId": "listDueCommitments",
        "tags": ["commitments"],
        "summary": "List overdue and due-soon open commitments across every account (F-4)",
        "description": "Computed at request time from commitments.due_date and commitments.status -- no NLU call in this flow (solution.json Section 4.3). `due_soon_window_days` defaults to 7, the PRD's own carried-over illustrative placeholder (US-13 AC2, still `[INFERRED — needs confirmation]` upstream) and may be overridden per request without a contract change. `overdue` and `due_soon` are mutually exclusive and reclassify automatically as the reference date advances (US-13 AC1/AC2); `unspecified` (no due date captured, per F-2) is listed separately, never folded into either bucket or silently omitted (US-13 AC3). `message` is populated only when both `overdue` and `due_soon` are empty (US-14) -- `unspecified` entries alone do not suppress that message, since US-14's AC is scoped to 'no due-soon or overdue commitments' specifically.",
        "x-story-ids": ["US-13", "US-14"],
        "security": [ { "SharedLoginAuth": [] } ],
        "parameters": [
          { "name": "due_soon_window_days", "in": "query", "required": false, "schema": { "type": "integer", "minimum": 1, "default": 7 }, "description": "[INFERRED — needs confirmation upstream, PRD Open Question 6] Look-ahead window in days for the 'due soon' bucket. Default 7 is a carried-over placeholder, not a confirmed business value." },
          { "name": "reference_date", "in": "query", "required": false, "schema": { "type": "string", "format": "date" }, "description": "Overrides 'today' for deterministic testing against seeded data with known due dates. Defaults to the server's current date." }
        ],
        "responses": {
          "200": {
            "description": "Three buckets, each ordered by due_date ascending (soonest first) for `overdue`/`due_soon`, and by the commitment's own creation order for `unspecified` (no due_date to sort by).",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/DueCommitmentsResponse" } } }
          },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/commitments/{commitmentId}/complete": {
      "patch": {
        "operationId": "markCommitmentComplete",
        "tags": ["commitments"],
        "summary": "Mark an open commitment complete, excluding it from the due-soon/overdue list",
        "description": "[INFERRED — needs confirmation] No PRD feature or story describes a rep action for marking a commitment complete, yet F-4 AC1 (US-13 AC1) explicitly requires excluding a commitment from the due-soon/overdue list 'once it is marked complete' -- that acceptance criterion is otherwise untestable and unimplementable without some mechanism to reach the 'complete' state. This operation is this discipline's own minimal, reasonable inference of that missing mechanism, carried forward as Open Item OI-5 rather than treated as a confirmed product decision (see decision DAT-5 in the report). Workflow 3/product owner may replace it with a different mechanism (e.g. inferring completion from a later note) without changing any other operation in this contract.",
        "x-story-ids": ["US-13"],
        "security": [ { "SharedLoginAuth": [] } ],
        "parameters": [
          { "name": "commitmentId", "in": "path", "required": true, "schema": { "type": "string" } }
        ],
        "responses": {
          "200": {
            "description": "Commitment marked complete; it is excluded from all future GET /commitments/due responses.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Commitment" } } }
          },
          "404": { "$ref": "#/components/responses/CommitmentNotFound" },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/auth/login": {
      "post": {
        "operationId": "login",
        "tags": ["auth"],
        "summary": "Exchange the single shared credential for a session (v1.1 -- resolves security_nfr's SEC-1)",
        "description": "Validates the one shared application credential (username/password, SEC-1 -- there is no per-individual account) and, on success, issues the opaque, server-validated session token via the `session` cookie the `SharedLoginAuth` scheme already names, with the flags and idle/absolute lifetime SEC-2 fixed (HttpOnly, Secure, SameSite=Strict; 2h idle / 12h absolute -- this contract does not restate those values, only that the cookie carries them). This operation itself decides nothing about the mechanism -- it is the API surface security_nfr's SEC-1 needed and v1.0 did not yet provide (see decision DAT-10; workflow sign-off revise, 2026-09-17). x-story-ids names US-20 (F-7's 'authenticated via the shared login' framing) as the closest genuine functional tie in scope -- no story's acceptance criteria actually exercises the login action itself, since every story assumes a session already exists; this is disclosed, not presented as a confirmed story requirement (DAT-10, Open Item OI-18).",
        "x-story-ids": ["US-20"],
        "security": [],
        "x-public-reason": "Login must be reachable with no session yet established -- that is the entire point of the operation. 'Public' here means unauthenticated network reachability only, not uncontrolled access: SEC-1's shared credential is still required to succeed (401 on a wrong credential), and SEC-5's lockout (429, see LoginLocked below) bounds brute-force attempts against this specific openness.",
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": { "$ref": "#/components/schemas/LoginRequest" }
            }
          }
        },
        "responses": {
          "200": {
            "description": "Credential accepted. The response body only confirms the outcome; the session itself is carried entirely by the `Set-Cookie` header below, per the `SharedLoginAuth` scheme (`type: apiKey`, `in: cookie`, `name: session`) v1.0 already named.",
            "headers": {
              "Set-Cookie": {
                "schema": { "type": "string" },
                "description": "`session=<opaque-token>; HttpOnly; Secure; SameSite=Strict` (SEC-2). Idle timeout 2h, absolute lifetime 12h, enforced server-side -- this contract names the cookie's flags because SEC-2 fixed them, but the concrete token format/value is an implementation detail, not a contract concern."
              }
            },
            "content": {
              "application/json": { "schema": { "$ref": "#/components/schemas/LoginResponse" } }
            }
          },
          "401": {
            "description": "The submitted username/password did not match the one shared credential. error_code=INVALID_CREDENTIALS. Distinct from the `Unauthorized` response used elsewhere in this contract, which means 'no valid session cookie' -- this means 'the credential itself was wrong'.",
            "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
          },
          "429": { "$ref": "#/components/responses/LoginLocked" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    },
    "/auth/logout": {
      "post": {
        "operationId": "logout",
        "tags": ["auth"],
        "summary": "Invalidate the current session server-side (v1.1 -- resolves security_nfr's SEC-2 logout requirement)",
        "description": "Invalidates the caller's session in the server-side session store (SEC-2) so the same cookie value can never be reused after this call, and instructs the client to discard the cookie. This is the API surface security_nfr's SEC-2 needed and v1.0 did not yet provide (see decision DAT-10; workflow sign-off revise, 2026-09-17). x-story-ids names US-20 for the same disclosed reason as `login` (Open Item OI-18) -- no story's acceptance criteria describes a logout action.",
        "x-story-ids": ["US-20"],
        "security": [ { "SharedLoginAuth": [] } ],
        "responses": {
          "200": {
            "description": "Session invalidated server-side; the previous session token is no longer accepted by any operation. The response instructs the client to discard the cookie.",
            "headers": {
              "Set-Cookie": {
                "schema": { "type": "string" },
                "description": "`session=; Max-Age=0` -- clears the cookie client-side, mirroring the server-side invalidation."
              }
            },
            "content": {
              "application/json": { "schema": { "$ref": "#/components/schemas/LogoutResponse" } }
            }
          },
          "401": { "$ref": "#/components/responses/Unauthorized" },
          "503": { "$ref": "#/components/responses/RetryLater" }
        }
      }
    }
  },
  "components": {
    "securitySchemes": {
      "SharedLoginAuth": {
        "type": "apiKey",
        "in": "cookie",
        "name": "session",
        "description": "The single-shared-login access model fixed by the PRD (Section 7, Access Control NFR), solution.json (no per-individual authentication in this version), and resolved with a concrete mechanism by security_nfr (SEC-1: shared credential exchanged for an opaque, server-validated session token; SEC-2: session lifecycle). v1.1 adds the `login`/`logout` operations (tag `auth`) that issue and invalidate this cookie -- this scheme's own shape (`apiKey`/`cookie`/`session`) is unchanged from v1.0."
      }
    },
    "responses": {
      "Unauthorized": {
        "description": "No valid shared-login session (missing, expired, or invalidated-by-logout cookie). Mechanism fixed by security_nfr SEC-1/SEC-2. error_code=UNAUTHENTICATED.",
        "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
      },
      "AccountNotFound": {
        "description": "No seeded account exists with that id. error_code=ACCOUNT_NOT_FOUND.",
        "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
      },
      "CommitmentNotFound": {
        "description": "No commitment exists with that id. error_code=COMMITMENT_NOT_FOUND.",
        "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
      },
      "RetryLater": {
        "description": "The write could not be serialized against SQLite's single-writer behavior within the configured busy-timeout and retry budget (decision DAT-3) -- an operational condition, not data loss: no write that reached this response was ever partially applied. The client should retry after the given delay; no session's data has been silently dropped (consistent with US-21 AC2).",
        "headers": {
          "Retry-After": { "schema": { "type": "integer" }, "description": "Seconds to wait before retrying." }
        },
        "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
      },
      "RateLimited": {
        "description": "v1.1 (decision DAT-11, resolves security_nfr's SEC-11/NFR-10). The per-session request rate for this operation exceeded the proposed limit (security_nfr SEC-11: 30 requests/minute, independent of the login-specific lockout in SEC-5/LoginLocked below) -- a cost/DoS control on the three operations that each trigger a Bedrock invocation (POST /interactions, POST /qa, POST /briefs). No Bedrock invocation occurs for a throttled request (NFR-10 verification). error_code=RATE_LIMITED.",
        "headers": {
          "Retry-After": { "schema": { "type": "integer" }, "description": "Seconds to wait before retrying, per SEC-11's per-session window." }
        },
        "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
      },
      "LoginLocked": {
        "description": "v1.1 (decision DAT-11, resolves security_nfr's SEC-5/NFR-4). The login endpoint has locked out/backed off after 5 failed attempts within a rolling 15-minute window per source IP (security_nfr SEC-5) -- a brute-force/credential-stuffing control distinct from RateLimited above: SEC-5 guards the one shared credential itself (a failed-attempt lockout), while RateLimited guards an already-authenticated session's request volume. Kept as a separate response/error_code so a client's own logging/alerting can distinguish 'someone is guessing the shared password' from 'a session is calling too fast' -- the same distinction platform's own PLA-8 alarms already treat as separately meaningful. error_code=LOGIN_LOCKED.",
        "headers": {
          "Retry-After": { "schema": { "type": "integer" }, "description": "Seconds remaining in the lockout window, per SEC-5." }
        },
        "content": { "application/json": { "schema": { "$ref": "#/components/schemas/Error" } } }
      }
    },
    "schemas": {
      "Error": {
        "type": "object",
        "required": ["error_code", "message"],
        "properties": {
          "error_code": {
            "type": "string",
            "enum": [
              "EMPTY_NOTE", "UNSUPPORTED_IMAGE_FORMAT", "INTERACTION_NOT_FOUND", "ACCOUNT_NOT_FOUND",
              "INTERACTION_ALREADY_MATCHED", "COMMITMENT_NOT_FOUND", "EMPTY_QUESTION", "EMPTY_REQUEST",
              "UNAUTHENTICATED", "RETRY_LATER", "INVALID_CREDENTIALS", "LOGIN_LOCKED", "RATE_LIMITED"
            ]
          },
          "message": { "type": "string", "description": "Human-readable, safe to display to the rep verbatim." },
          "details": { "type": ["object", "null"], "additionalProperties": true }
        }
      },
      "Account": {
        "type": "object",
        "required": ["id", "name"],
        "properties": {
          "id": { "type": "string" },
          "name": { "type": "string" }
        }
      },
      "Attendee": {
        "type": "object",
        "required": ["id", "name"],
        "properties": {
          "id": { "type": "string" },
          "name": { "type": "string" },
          "company": { "type": ["string", "null"] },
          "role": { "type": ["string", "null"] }
        }
      },
      "Commitment": {
        "type": "object",
        "required": ["id", "account_id", "interaction_id", "text", "due_date", "status"],
        "properties": {
          "id": { "type": "string" },
          "account_id": { "type": ["string", "null"], "description": "Null exactly when the parent interaction is unmatched (F-2 AC3) -- denormalized from the parent interaction; set when it is resolved via resolve-customer." },
          "interaction_id": { "type": "string" },
          "text": { "type": "string" },
          "due_date": { "type": ["string", "null"], "format": "date", "description": "Null means unspecified (F-2 AC2/AC4) -- never a guessed date." },
          "status": { "type": "string", "enum": ["open", "complete"] }
        }
      },
      "Interaction": {
        "type": "object",
        "required": ["id", "account_id", "raw_text", "source_type", "match_status", "captured_at", "attendees", "commitments"],
        "properties": {
          "id": { "type": "string" },
          "account_id": { "type": ["string", "null"], "description": "Null exactly when match_status='unmatched' (F-1 AC1/AC4)." },
          "raw_text": { "type": "string", "description": "Verbatim captured text, retained in full even when downstream extraction is incomplete (PRD Section 13.1)." },
          "source_type": { "type": "string", "enum": ["typed", "business_card"] },
          "match_status": { "type": "string", "enum": ["matched", "unmatched"] },
          "manual_entry_required": { "type": "boolean", "description": "business_card submissions only: true when the image could not be read or no contact fields were recognized (US-4 AC2). Always false for source_type='typed'." },
          "captured_at": { "type": "string", "format": "date-time" },
          "attendees": { "type": "array", "items": { "$ref": "#/components/schemas/Attendee" } },
          "commitments": { "type": "array", "items": { "$ref": "#/components/schemas/Commitment" } }
        }
      },
      "AccountProfile": {
        "type": "object",
        "required": ["account", "interactions", "is_empty"],
        "properties": {
          "account": { "$ref": "#/components/schemas/Account" },
          "interactions": {
            "type": "array",
            "items": { "$ref": "#/components/schemas/Interaction" },
            "description": "Ordered by captured_at ascending, id ascending as a deterministic tie-breaker for equal timestamps (decision DAT-6)."
          },
          "is_empty": { "type": "boolean", "description": "True exactly when interactions=[] -- the client must render the explicit 'no history yet' state (US-18 AC2), never a bare empty list." }
        }
      },
      "CaptureTextRequest": {
        "type": "object",
        "required": ["raw_text"],
        "properties": {
          "raw_text": { "type": "string", "minLength": 0, "description": "Must contain at least one non-whitespace character (US-1 AC2) or the request is rejected with 400 EMPTY_NOTE (US-2 AC1); minLength is not set to 1 here because whitespace-only strings satisfy minLength:1 too -- the non-whitespace check is a semantic validation the schema alone cannot express, and is enforced by the server." }
        }
      },
      "CaptureBusinessCardRequest": {
        "type": "object",
        "required": ["business_card_image"],
        "properties": {
          "business_card_image": { "type": "string", "contentEncoding": "base64", "contentMediaType": "image/*", "description": "The scanned business-card image (US-4). Not retained after synchronous processing -- decision DAT-8." }
        }
      },
      "ResolveCustomerRequest": {
        "type": "object",
        "required": ["account_id"],
        "properties": {
          "account_id": { "type": "string", "description": "The account the rep selected from GET /accounts to resolve this unmatched note (US-3 AC1)." }
        }
      },
      "QARequest": {
        "type": "object",
        "required": ["question"],
        "properties": {
          "question": { "type": "string", "description": "Free-text natural-language question, e.g. \"what did we discuss last time?\", \"what did I commit to?\", \"who attended from their side?\", or any other question naming a customer (F-3)." },
          "account_id": { "type": ["string", "null"], "description": "Optional hint when the rep is already viewing one account's profile (decision DAT-4). When omitted, the account is resolved from `question` itself." }
        }
      },
      "QAResponse": {
        "type": "object",
        "required": ["resolution"],
        "properties": {
          "resolution": { "type": "string", "enum": ["answered", "no_history", "no_open_commitments", "no_attendees_captured", "not_identified"] },
          "account_id": { "type": ["string", "null"] },
          "answer_text": { "type": ["string", "null"], "description": "Populated only when resolution='answered'." },
          "message": { "type": ["string", "null"], "description": "Populated for every non-'answered' resolution -- an explicit, human-readable statement of the empty/failure state (US-9 AC2, US-10 AC2, US-11 AC2, US-12 AC2), never a silent empty response." }
        }
      },
      "BriefRequest": {
        "type": "object",
        "required": ["request_text"],
        "properties": {
          "request_text": { "type": "string", "description": "e.g. \"brief me on Acme\" (F-5)." },
          "account_id": { "type": ["string", "null"], "description": "Optional hint, same pattern as QARequest.account_id (decision DAT-4)." }
        }
      },
      "BriefSummary": {
        "type": "object",
        "properties": {
          "history_text": { "type": ["string", "null"], "description": "Bounded, short-form synthesis of recent discussion history (US-15 AC2)." },
          "open_commitments": { "type": "array", "items": { "$ref": "#/components/schemas/Commitment" } },
          "stakeholders": { "type": "array", "items": { "$ref": "#/components/schemas/Attendee" }, "description": "Drawn from interaction_attendees and the seeded contacts roster for this account." },
          "opportunity_status": { "type": ["string", "null"], "description": "[INFERRED — needs confirmation, PRD Assumption 13/Open Question 14] Narrative synthesis only -- no discrete deal-stage field exists (PRD Section 13.1). Null means nothing captured yet (US-16 AC2), not a fabricated stage." }
        }
      },
      "BriefResponse": {
        "type": "object",
        "required": ["resolution"],
        "properties": {
          "resolution": { "type": "string", "enum": ["generated", "no_history", "not_identified"] },
          "account_id": { "type": ["string", "null"] },
          "summary": { "anyOf": [ { "$ref": "#/components/schemas/BriefSummary" }, { "type": "null" } ] },
          "message": { "type": ["string", "null"], "description": "Populated for 'no_history' and 'not_identified' -- explicit statement, never a silently empty/fabricated brief (US-15 AC1, US-17 AC2)." }
        }
      },
      "DueCommitmentsResponse": {
        "type": "object",
        "required": ["overdue", "due_soon", "unspecified"],
        "properties": {
          "overdue": { "type": "array", "items": { "$ref": "#/components/schemas/Commitment" } },
          "due_soon": { "type": "array", "items": { "$ref": "#/components/schemas/Commitment" } },
          "unspecified": { "type": "array", "items": { "$ref": "#/components/schemas/Commitment" }, "description": "Commitments with due_date=null, listed separately, never omitted or folded into overdue/due_soon (US-13 AC3)." },
          "message": { "type": ["string", "null"], "description": "Populated exactly when overdue=[] and due_soon=[] -- the explicit 'nothing due or overdue' statement (US-14 AC1)." }
        }
      },
      "LoginRequest": {
        "type": "object",
        "required": ["username", "password"],
        "properties": {
          "username": { "type": "string", "description": "The single shared application credential's username (security_nfr SEC-1). Not a per-individual identity -- the same value for every rep using this demo." },
          "password": { "type": "string", "description": "The single shared application credential's password (SEC-1). Never logged (SEC-10) or echoed back in any response." }
        },
        "description": "v1.1. The actual credential value, who holds it, and its distribution/rotation cadence are an operational/human decision (security_nfr Open Item OI-8), not fixed by this schema."
      },
      "LoginResponse": {
        "type": "object",
        "required": ["authenticated"],
        "properties": {
          "authenticated": { "type": "boolean", "description": "Always true on a 200 -- a failed login is a 401 or 429, never a 200 with authenticated=false." }
        },
        "description": "v1.1. The session itself is carried by the `session` cookie (SharedLoginAuth scheme), set via the 200 response's Set-Cookie header -- this body only confirms the outcome, it does not restate cookie-carried session state or any session-lifetime value (those are SEC-2's to set, not this schema's to repeat)."
      },
      "LogoutResponse": {
        "type": "object",
        "required": ["invalidated"],
        "properties": {
          "invalidated": { "type": "boolean", "description": "Always true on a 200 -- logout either invalidates the session and returns 200, or the session was already invalid and the call fails 401, never a 200 with invalidated=false." }
        },
        "description": "v1.1. Confirms server-side session invalidation (SEC-2); the client is expected to also discard the cookie per the accompanying Set-Cookie header."
      }
    }
  }
}
```
<!-- api-contract:end -->

## Appendix B — Data Model (full, embedded)

Standalone file: `docs/architecture/DATA-MODEL-conversational-crm-T-1-v1.1.md`

<!-- data-model:start -->
# Data Model — Conversational CRM

- **Document reference:** `DATA-MODEL-conversational-crm-T-1-v1.1`
- **Discipline:** `data_integration` (Workflow 2 — Solution Architecture)
- **Task:** T-1, scope `full-product-v1` — all 7 PRD features (F-1..F-7), all 21 stories (US-1..US-21)
- **Status:** Draft — pending independent validation and human gate (`data_integration_review`)
- **Version:** 1.1 (revised from v1.0 at workflow sign-off; see the v1.1 note in Section 0)
- **Scope boundary:** This document specifies C-SQL's (SQLite) tables, and C-VEC's (Chroma) logical collection shape, as named and bounded by `solution` (components, boundaries). It does not redraw any component boundary, decide authentication/authorization (`security_nfr`), or decide hosting/deployment mode of either store (`platform` — see solution.json SOL-9, "embedded vs. server... is platform's to finalize").

## 0. v1.1 revision note

At Workflow 2 sign-off, the human replied `revise: data_integration <feedback>` because `security_nfr`'s already-approved `SharedLoginAuth` resolution (SEC-1, SEC-2, SEC-5, SEC-11) needs login/logout operations and a `429`-class rate-limit response the v1.0 API contract did not have (`security_nfr.json` `returned_findings[0]`, `RF-1`/`V-SEC2`). That gap is entirely an API-contract gap, not a data-model one: **no entity, field, constraint, or index in this document changes in v1.1.** Sections 1 through 4 below are otherwise unchanged from v1.0, word for word — carried forward, not re-derived. The only new content is this note and Open Item OI-18 (Section 5, new), which discloses a genuine adjacent question this revision surfaced: where the new login/logout operations' server-side session store actually lives is not decided by this document, or by any other discipline's artifact today.

## 1. Structured store (C-SQL — SQLite)

Five tables. PRD Section 13.1 already narratively described accounts/contacts, interactions, interaction attendees, and commitments; this section makes their field-level shape, types, constraints and indexes explicit and adds the one field-level extension (`commitments.status`) needed to make an existing PRD acceptance criterion implementable (decision DAT-5).

### 1.1 `accounts`

Seeded customer accounts (PRD Section 9 dependency; CRM adapter's `fetchAccounts()`, PRD Section 14.1a).

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | TEXT | PRIMARY KEY | |
| `name` | TEXT | NOT NULL | Display name used both by the UI (GET /accounts) and by C-NLU's customer-name resolution (F-1 AC1, F-3 AC4, F-5 AC3). |
| `created_at` | TEXT (ISO-8601 datetime) | NOT NULL | Seed-load timestamp. |

**Indexes:**
- `idx_accounts_name` on `(name COLLATE NOCASE)` — read pattern: every customer-name resolution (capture matching, Q&A, brief requests) looks up an account by name, case-insensitively; without this index every one of those lookups is a full table scan.

### 1.2 `contacts`

Seeded roster of known people at an account (CRM adapter's `fetchContacts(accountId)`, PRD Section 14.1a). Distinct from `interaction_attendees` below: this is the account's known-stakeholder roster, not per-interaction extracted mentions.

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | TEXT | PRIMARY KEY | |
| `account_id` | TEXT | NOT NULL, FOREIGN KEY → `accounts(id)` | |
| `name` | TEXT | NOT NULL | |
| `company` | TEXT | NULLABLE | |
| `role` | TEXT | NULLABLE | |
| `created_at` | TEXT (ISO-8601 datetime) | NOT NULL | |

**Indexes:**
- `idx_contacts_account_id` on `(account_id)` — read pattern: Brief-Me's stakeholder listing (F-5 AC1, US-15) fetches every known contact for one account.

### 1.3 `interactions`

One row per capture (F-1). `[Restated with fields, PRD v1.5 Section 13.1]`.

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | TEXT | PRIMARY KEY | |
| `account_id` | TEXT | NULLABLE, FOREIGN KEY → `accounts(id)` | NULL exactly when `match_status='unmatched'` (F-1 AC1/AC4). Set by `resolve-customer` once the rep manually resolves it (US-3 AC1). |
| `raw_text` | TEXT | NOT NULL, application-enforced: at least one non-whitespace character | Verbatim captured text, retained in full even when downstream extraction is incomplete or only partially succeeds (PRD Section 13.1). The empty/whitespace-only case is rejected before a row is ever written (US-2 AC1) — there is no "empty" row state. |
| `source_type` | TEXT | NOT NULL, CHECK IN ('typed', 'business_card') | F-1's two capture paths. |
| `match_status` | TEXT | NOT NULL, CHECK IN ('matched', 'unmatched') | |
| `manual_entry_required` | INTEGER (boolean) | NOT NULL, DEFAULT 0 | `business_card` submissions only: true when the image could not be read or no contact fields were recognized (US-4 AC2). Always 0 for `source_type='typed'`. |
| `captured_at` | TEXT (ISO-8601 datetime) | NOT NULL | |
| `created_at` | TEXT (ISO-8601 datetime) | NOT NULL | Row-insert timestamp; distinct from `captured_at` in case a future version backdates captures — identical to `captured_at` for every story in this scope. |

**Indexes:**
- `idx_interactions_account_captured` on `(account_id, captured_at, id)` — read pattern: F-6's chronological per-customer profile view (US-18, US-19) and F-3 AC1's "most recent interaction note(s)" query (US-9) both filter by one account and sort by time; `id` is a deterministic tie-breaker for two captures at the identical timestamp (decision DAT-6), which matters directly for US-21 (two concurrent sessions' captures must both persist and appear in a stable order, not an arbitrary one).
- `idx_interactions_match_status` on `(match_status)` — read pattern: `GET /interactions/unmatched` (US-3) must find every unmatched row without scanning the (typically much larger) matched set.

### 1.4 `interaction_attendees`

Zero-to-many child rows per interaction (F-1 AC5, US-5; also populated by the business-card path, F-1 AC2/US-4). `[New entity, PRD v1.5 Section 13.1]`.

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | TEXT | PRIMARY KEY | |
| `interaction_id` | TEXT | NOT NULL, FOREIGN KEY → `interactions(id)` | |
| `name` | TEXT | NOT NULL | A row does not exist without a name (PRD Section 13.1). |
| `company` | TEXT | NULLABLE | Populated only when the business-card path or the entry's own text supplies it (PRD Section 13.1) — never guessed. |
| `role` | TEXT | NULLABLE | Same rule as `company`. |

**Indexes:**
- `idx_attendees_interaction_id` on `(interaction_id)` — read pattern: joining attendees onto their parent interaction for the profile view (US-18) and for F-3 AC3's "who attended" answer (US-11), which reads every attendee row across a customer's interactions.

### 1.5 `commitments`

Zero-to-many child rows per interaction (F-2 AC1). `[Restated with fields, PRD v1.5 Section 13.1]`; `status` is this discipline's own addition (decision DAT-5).

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | TEXT | PRIMARY KEY | |
| `interaction_id` | TEXT | NOT NULL, FOREIGN KEY → `interactions(id)` | |
| `account_id` | TEXT | NULLABLE, FOREIGN KEY → `accounts(id)`, denormalized from the parent interaction | NULL exactly when the parent interaction is unmatched (F-2 AC3 — "flag an extracted item for manual thread assignment... rather than tagging it to an incorrect thread"); set at the same time the parent interaction is resolved (`resolve-customer`). Denormalized (rather than requiring every reader to join through `interactions`) specifically because F-4's due-soon/overdue query (US-13) reads across every account at once and needs `account_id` directly in its result rows. |
| `text` | TEXT | NOT NULL | The extracted commitment/follow-up/next-step text. |
| `due_date` | TEXT (ISO-8601 date) | NULLABLE | NULL means unspecified (F-2 AC2/AC4) — populated only when the source text states or clearly implies a date; never guessed. |
| `status` | TEXT | NOT NULL, CHECK IN ('open', 'complete'), DEFAULT 'open' | `[INFERRED — needs confirmation]` Added because F-4 AC1 (US-13 AC1) requires excluding a commitment from the due-soon/overdue list "once it is marked complete," but no PRD feature or story describes how a commitment reaches that state. See decision DAT-5 and Open Item OI-5. |
| `created_at` | TEXT (ISO-8601 datetime) | NOT NULL | |

**Indexes:**
- `idx_commitments_status_duedate` on `(status, due_date)` — read pattern: F-4's cross-account due-soon/overdue query (US-13, US-14) filters `status='open'` and ranges over `due_date`; this is the one query in the whole system that intentionally reads across every account at once, so an index scoped to a single account would not help it.
- `idx_commitments_interaction_id` on `(interaction_id)` — read pattern: joining a commitment back onto its parent interaction for the profile view (US-18) and for tagging validation (F-2 AC3, US-8).

### 1.6 Referential and lifecycle notes

- No feature in F-1 through F-7 deletes an interaction, attendee, or commitment — there is no delete operation in the API contract, and none is inferred here.
- No feature edits `raw_text` after capture — interactions are append-only from the API's perspective. This is why decision DAT-2 can treat re-embedding as a create-time-only concern (plus one metadata-only exception — see Section 2.3).
- Foreign keys are enforced at the application layer (SQLite's `PRAGMA foreign_keys=ON`) rather than assumed; `platform` is responsible for confirming this pragma is set at connection time, since it is a per-connection setting in SQLite, not a database-file-level default.
- **v1.1:** no `sessions` table is added here for the new login/logout operations (Section 0). See Open Item OI-18.

## 2. Vector store (C-VEC — Chroma)

Chroma is not a relational store, so it is described here as one logical collection rather than as SQL tables, per solution.json's own component responsibility for C-VEC.

### 2.1 Collection

A single collection, `embedded_content`, holds every embedded chunk regardless of source type (interaction free text, and, once the corresponding adapters are exercised, email/SharePoint/MoM-derived content per PRD Section 13.2). One collection (not one per account) was chosen so that Chroma's own metadata filtering — not a separate collection per account — is what scopes a query to one customer; this avoids a collection-management step every time a new seeded account is added.

### 2.2 Per-chunk shape

| Field | Chroma concept | Type | Notes |
|---|---|---|---|
| `id` | document id | string | Deterministic: `"{source_type}:{source_id}:{chunk_index}"`, e.g. `"interaction:INT-042:0"`. Deterministic ids make re-running seed ingestion idempotent (upsert, not duplicate). |
| (embedding vector) | embedding | float[1024] | See decision DAT-2 — Amazon Titan Text Embeddings V2, 1024-dimension output. |
| chunk text | document | string | The chunk's own text, stored so a retrieved chunk is immediately usable by C-NLU for answer/brief synthesis without a second read against another store. |
| `account_id` | metadata | string | The customer thread this chunk belongs to. Every query from F-3/F-5 filters on this field first — this is what makes "resolve to the correct customer... never return another customer's data" (F-3 AC4, F-5 AC3) mechanically enforced at the retrieval layer, not just at the answer-synthesis layer. |
| `source_type` | metadata | string, one of `interaction`, `email`, `sharepoint`, `mom` | Traceability field (see 2.4). |
| `source_id` | metadata | string | The SQLite `interactions.id` (for `interaction`) or the adapter-returned document/message id (for the other three). Traceability field. |
| `chunk_index` | metadata | integer | Position of this chunk within its source, per the semantic-chunking strategy already fixed upstream (PRD Section 13.2). |
| `embedded_at` | metadata | string (ISO-8601 datetime) | |

### 2.3 Re-embedding trigger (resolves Open Item OI-1a in part — see decision DAT-2)

- **Interaction free text** — embedded exactly once, synchronously, as part of `POST /interactions` (same request that writes the SQLite rows, per solution.json SOL-3's synchronous-only architecture). No feature edits `raw_text` after capture, so there is no content-change re-embedding trigger for this source type in this scope.
- **Metadata-only upsert on customer resolution** — when a previously unmatched interaction is resolved via `POST /interactions/{id}/resolve-customer` (US-3), that interaction's chunk(s) already exist in `embedded_content` with `account_id=null`-equivalent (or a distinguished "unassigned" sentinel value, since Chroma metadata filtering does not reliably support a literal null). The resolve-customer operation must upsert those chunks' `account_id` metadata field to the newly-assigned account — **without re-computing the embedding vector**, since the underlying text has not changed. This is the one metadata mutation this store undergoes after initial embedding.
- **Adapter-sourced content (email, SharePoint, MoM)** — embedded once at seed-ingestion time (when the corresponding stub adapter's data is loaded). No feature in F-1 through F-7 modifies this content after seeding, so no further re-embedding trigger exists for it in this scope. If a future phase allows re-seeding/refreshing adapter content, re-embedding on change-detection would need to be added then — out of scope now, not silently assumed.
- **What remains unresolved:** the general case of "a chunk's source content changes" has no trigger at all in this design, because no feature in this scope ever changes previously-captured content. This is a scope-boundedness statement, not a gap in this document — if Workflow 3 or a later phase adds an edit capability, a re-embedding trigger would need to be designed then.

### 2.4 Traceability requirement satisfied

Every embedded chunk carries `account_id` and `source_type`/`source_id` metadata (Section 2.2), so "every embedded item must remain traceable back to the customer thread and source interaction/document it was extracted from" (PRD Section 13.2) is satisfied by Chroma's own native metadata rather than a new SQLite mapping table (decision DAT-9) — there is no `embeddings` table in Section 1.

## 3. Constraint-to-contract expressibility check

Per this discipline's self-verification requirement ("every constraint the data model states is expressible and expressed in the contract schema, or the difference is recorded as an open item"):

| Data-model constraint | Expressed in API-CONTRACT schema? |
|---|---|
| `interactions.account_id` NULL iff `match_status='unmatched'` | Yes — `Interaction.account_id` description states the pairing; enforced server-side, not purely a client-schema concern (OpenAPI/JSON Schema cannot itself express a conditional cross-field constraint). |
| `commitments.account_id` NULL iff parent interaction is unmatched (denormalized, Section 1.5) | Yes, after correction — `Commitment.account_id` was originally declared `{"type": "string"}` (non-nullable) in the contract, contradicting this row's own NULLABLE column; an independent review caught the gap (this expressibility table had no row for it at all) and it is fixed to `["string","null"]` with the pairing stated in its description, matching `Interaction.account_id`'s treatment exactly. |
| `raw_text` at least one non-whitespace character | Partially — JSON Schema cannot express "not all-whitespace" directly; `CaptureTextRequest.raw_text` documents the rule in its description and the server enforces it, returning 400 EMPTY_NOTE on violation (this is exactly the kind of gap this discipline's own contract convention exists to surface, not paper over). |
| `interaction_attendees.name` required, non-empty | Yes — `Attendee.name` is a required string field. |
| `commitments.due_date` nullable, never a guessed value | Yes — `Commitment.due_date` is `["string","null"]`; the "never guessed" half is a behavioral guarantee, not a schema-checkable one, and is stated in the field description. |
| `commitments.status` enum `open`/`complete` | Yes — `Commitment.status` enum. |
| Vector-store `account_id` scoping | Not applicable to the SQL-facing API contract — C-VEC is never called directly by C-WEB (SOL-7); this constraint is internal to C-API's retrieval logic and is documented here (Section 2.2) rather than in the OpenAPI contract. |
| **v1.1:** login/logout session state (no entity added, Section 0/1.6) | Not applicable — no new entity or constraint was added to this data model in v1.1; the session store's own shape, if any, is not this document's to specify until a discipline decides it belongs in C-SQL rather than an out-of-process store (Open Item OI-18). |

## 4. Self-verification

- Every entity in this document is either a SQLite table (Section 1) or the one Chroma collection (Section 2) — none introduces a component boundary `solution` did not already name. **PASS.**
- Every field used by an operation in API-CONTRACT-conversational-crm-T-1-v1.1.json traces back to a field defined here (spot-checked: `Interaction.manual_entry_required` ↔ `interactions.manual_entry_required`; `Commitment.status` ↔ `commitments.status`; `DueCommitmentsResponse.unspecified` ↔ `commitments.due_date IS NULL`; the new v1.1 `LoginRequest`/`LoginResponse`/`LogoutResponse` schemas reference no C-SQL/C-VEC field at all, consistent with Section 0's "no entity change" statement). **PASS.**
- Every index states a real read pattern from a named story, not a generic "for performance" justification. **PASS** — see Sections 1.1–1.5.
- Every inference (`commitments.status`, the re-embedding trigger boundary, the vector-store traceability approach) is marked `[INFERRED — needs confirmation]` where it introduces new unconfirmed structure, and carried to this discipline's `open_items`. **PASS.**

**Result: PASS.**

## 5. Open items (v1.1 addition)

This document's own open items (data modeling only) are unchanged from v1.0 except for the addition below; the full set is recorded in the accompanying report (Section 5) and in `roles/data_integration.json`.

| ID | Item | Blocks development? | Owner | Resolution needed |
|---|---|---|---|---|
| OI-18 | The server-side session store backing the new `login`/`logout` operations (API-CONTRACT v1.1) has no persistence-technology decision recorded by any discipline. `security_nfr`'s SEC-1/SEC-2/SEC-3 fix the mechanism (opaque token) and the policy (lifecycle, storage-hashing of the credential) but not where session state itself is held at runtime (e.g. an in-process/in-memory store, a networked cache, or a new SQLite table). This document does not add a `sessions` table because no discipline has decided session state belongs in C-SQL rather than an out-of-process store — inventing one here would be exactly the kind of unconfirmed structure this discipline's own conventions require flagging, not silently building. | No | `security_nfr` / `platform` | Confirm where session state is actually held before Workflow 3/4 build the login/logout mechanism; if it turns out to belong in C-SQL, that is a data-modeling decision for whichever discipline finalizes it, and this document would need a `sessions` table added at that point. |
<!-- data-model:end -->
