# Data Model — Conversational CRM

- **Document reference:** `DATA-MODEL-conversational-crm-T-1-v1.0`
- **Discipline:** `data_integration` (Workflow 2 — Solution Architecture)
- **Task:** T-1, scope `full-product-v1` — all 7 PRD features (F-1..F-7), all 21 stories (US-1..US-21)
- **Status:** Draft — pending independent validation and human gate (`data_integration_review`)
- **Version:** 1.0
- **Scope boundary:** This document specifies C-SQL's (SQLite) tables, and C-VEC's (Chroma) logical collection shape, as named and bounded by `solution` (components, boundaries). It does not redraw any component boundary, decide authentication/authorization (`security_nfr`), or decide hosting/deployment mode of either store (`platform` — see solution.json SOL-9, "embedded vs. server... is platform's to finalize").

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

## 4. Self-verification

- Every entity in this document is either a SQLite table (Section 1) or the one Chroma collection (Section 2) — none introduces a component boundary `solution` did not already name. **PASS.**
- Every field used by an operation in API-CONTRACT-conversational-crm-T-1-v1.0.json traces back to a field defined here (spot-checked: `Interaction.manual_entry_required` ↔ `interactions.manual_entry_required`; `Commitment.status` ↔ `commitments.status`; `DueCommitmentsResponse.unspecified` ↔ `commitments.due_date IS NULL`). **PASS.**
- Every index states a real read pattern from a named story, not a generic "for performance" justification. **PASS** — see Sections 1.1–1.5.
- Every inference (`commitments.status`, the re-embedding trigger boundary, the vector-store traceability approach) is marked `[INFERRED — needs confirmation]` where it introduces new unconfirmed structure, and carried to this discipline's `open_items`. **PASS.**

**Result: PASS.**
