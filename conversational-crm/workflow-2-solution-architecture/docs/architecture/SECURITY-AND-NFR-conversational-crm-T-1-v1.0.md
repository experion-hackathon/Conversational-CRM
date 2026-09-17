# Security and NFR Design — Conversational CRM

- **Product name:** Conversational CRM / Relationship Memory Assistant
- **Document reference:** `SECURITY-AND-NFR-conversational-crm-T-1-v1.0`
- **Discipline:** `security_nfr` (Workflow 2 — Solution Architecture)
- **Task:** T-1, scope `full-product-v1` — all 7 PRD features (F-1..F-7), all 21 stories (US-1..US-21). Same whole-scope selection `solution` and `data_integration` designed against.
- **Owner:** Security-architect (role, not named individual)
- **Status:** Draft — pending independent validation and human gate (`security_nfr_review`)
- **Version:** 1.0

## 0. Scope boundary (what this document does and does not cover)

This is the `security_nfr` discipline's artifact. It analyses the components, boundaries
and operations `solution` and `data_integration` already designed — it does not redesign
them. Where a control requires a change to another discipline's artifact, that is recorded
below as a finding against that discipline, not edited directly.

This document covers:
- Every trust boundary in the approved solution/data_integration design: what crosses it,
  in which direction, what an attacker positioned there can read, forge, or deny.
- Which controls apply, which were considered and rejected (with a recorded reason), and
  which are recorded as a finding against another discipline.
- The concrete authentication/session mechanism the API contract's `SharedLoginAuth`
  scheme deliberately left as a placeholder for this discipline to decide.
- Every measurable NFR target this scope needs, its source (inherited from the PRD vs.
  derived/inferred by this discipline), and the verification that would demonstrate it.

It does **not** cover: component naming or responsibilities (`solution`), table/schema or
API operation shapes (`data_integration` — this document analyses those shapes, it does
not restate or re-specify them), or cloud topology, hosting, CI/CD, secrets-management
tooling, or rollback procedure (`platform` — several controls below name a requirement
that `platform` must implement; this document states the requirement, not the mechanism).

## 1. Source references

| Role | Artifact | Path (relative to this workflow's own artifact root) | Version | SHA-256 |
|---|---|---|---|---|
| `product_state` | Workflow 1 `workflow.json` (lock confirmation) | `../state/workflow-1-discovery-to-stories/workflow.json` | locked | `fb25e9c8829f6e0d0d9ccf9cddf5db8ae9b0cb7eba5873baff9e7e8bffc21b27` |
| `prd` | PRD | `../workflow-1-discovery-to-stories/docs/requirements/PRD-conversational-crm-v1.5.md` | 1.5 | `3960abdacf6ba3a27217d913aca01fa932bfb65373e572cdc4eeb6ac77dccc64` |
| `user_stories` | User Stories | `../workflow-1-discovery-to-stories/docs/stories/USER-STORIES-conversational-crm-v1.0.md` | 1.0 | `43fc3c0690db8cc6a0d85a837286912072fbcfdbfa55d3d37cb74c6abe8de73b` |
| `solution` | Solution Architecture | `docs/architecture/SOLUTION-ARCHITECTURE-conversational-crm-T-1-v1.0.md` | 1.0 | `99038391a4a33ae850e1ddc50e4709cfebb2b66525b10614f848c44b8368cc04` |
| `data_integration` | Data and Integration report | `docs/architecture/DATA-AND-INTEGRATION-conversational-crm-T-1-v1.0.md` | 1.0 | `98933f68639a8cf1640bc1d74c3007a17283bb6e35fd4970f4becb87318c4001` |
| `data_integration` | API Contract (OpenAPI 3.1) | `docs/architecture/API-CONTRACT-conversational-crm-T-1-v1.0.json` | 1.0 | `b048e263f2b43c43ee99e4844df48b12856a1cef977d3af2e65041db8d2ec745` |
| `data_integration` | Data Model | `docs/architecture/DATA-MODEL-conversational-crm-T-1-v1.0.md` | 1.0 | `66b0f4fb0b3462e8706eb48969efd704f7106777b33bc0fe1e895390e5da8dab` |

All seven hashes were computed directly against the current file bytes on disk by this
discipline (`sha256sum`), not copied from another role's state. All seven match exactly
what `roles/solution.json` and `roles/data_integration.json` themselves recorded — no new
drift found on top of the already-disclosed, already-resolved PRD line-ending drift
(solution's Open Item OI-1, not re-litigated here; see Section 8).

Before drafting, this discipline confirmed: Workflow 1 is `locked` (`workflow.status:
"locked"`, all three stage gates — `discovery`, `requirements`, `stories` — individually
`gate.status: "approved"`); `solution.gate.status: "approved"` and `solution.readiness:
"ready"`; `data_integration.gate.status: "approved"` and `data_integration.readiness:
"ready"`. `inputs.task` (T-1, full-product-v1, all 7 features / 21 stories) and
`stack.components[]` (5 components, each with a `technology`) were read directly from
`workflow.json` — no `needs_input` stop required.

## 2. Trust boundaries

### 2.1 Boundary inventory

| ID | Boundary | Crosses | Direction / carries |
|---|---|---|---|
| TB-1 | Internet ↔ C-WEB | Rep's browser (arbitrary network, arbitrary client machine) to the served CRM Web App | HTTPS; loads the SPA bundle, then browser-side JS execution begins |
| TB-2 | C-WEB ↔ C-API | The rep's browser (an **untrusted execution context** — see 2.3) to the sole backend integration point | REST/JSON over HTTPS; carries the shared-login session credential on every call, plus every request/response payload (capture text, images, questions, briefs) |
| TB-3 | C-API ↔ C-SQL | Backend process to its local structured store | SQL read/write, same host/process trust zone (no network hop assumed) |
| TB-4 | C-API ↔ C-VEC | Backend process to its local/embedded vector store | Vector upsert/query, same host/process trust zone (SOL-9: embedded-vs-server mode is `platform`'s to finalize; this boundary's threat profile changes if `platform` chooses a networked Chroma server instead of embedded — flagged as a contingent finding to `platform` in Section 8) |
| TB-5 | C-API ↔ C-NLU | Backend process, out of process, across the public internet, to AWS Bedrock | Model-invocation network calls carrying captured free text, business-card images, and retrieval context; also carries the AWS credential authorizing the call |
| TB-6 | C-API internal adapter layer ↔ (stub data only) | None today — all four adapters (CRM, Calendar, SharePoint, Email) return stub/seeded data in-process, per PRD Section 14.1 | Not a real network boundary in this scope; recorded because it becomes one the moment a real integration replaces a stub (see Section 8, contingent finding) |

Component IDs match `solution`'s Section 2 exactly (C-WEB, C-API, C-SQL, C-VEC, C-NLU); no
new component is introduced by this discipline.

### 2.2 What an attacker positioned at each boundary can read, forge, or deny

**TB-1 (Internet ↔ C-WEB).** An attacker on the network path can attempt to intercept or
downgrade the connection before the SPA even loads. What they get if they succeed: the
static SPA bundle only (no data) — but a successful downgrade here undermines every
control assumed at TB-2 (a stolen session cookie is only as safe as the transport it
travels over). Control: TLS everywhere, HSTS, no plaintext HTTP fallback (SEC-1, SEC-2).

**TB-2 (C-WEB ↔ C-API) — the boundary that actually matters most.** C-WEB is a
browser-rendered SPA; the code enforcing anything at all lives in C-API, not in the
browser. An attacker positioned here is not a single actor — it is any of: (a) an
attacker who has stolen or observed the shared-login session credential (via a leaked
cookie, a compromised rep laptop, or a network capture if TLS is ever bypassed); (b) an
attacker who has achieved script execution *inside* C-WEB's own origin (e.g. a
cross-site-scripting bug, a malicious browser extension, or a supply-chain-compromised
frontend dependency); (c) a legitimate rep acting outside the intended UI (e.g. scripting
`curl` calls directly against C-API using their own valid session). All three can, with
nothing more than the shared session credential:
- **Read**: every seeded account's full interaction history, commitments, attendees, and
  contacts (`GET /accounts/{id}/profile`), and pose arbitrary Q&A/brief questions against
  any account (`POST /qa`, `POST /briefs`) — there is no additional authorization layer
  narrowing "which account can this session see," because the PRD's own Access Control
  NFR (Section 7) deliberately makes every session under the shared login equivalent
  (F-7 AC1/AC2, US-20). This is a **stated, accepted design property**, not an oversight
  — see Section 4, "controls considered and rejected" — but it means TB-2 is the entire
  perimeter: once past it, there is no second gate anywhere in this architecture.
- **Forge**: new interactions, commitments, and customer-resolutions (`POST
  /interactions`, `POST /interactions/{id}/resolve-customer`, `PATCH
  /commitments/{id}/complete`) indistinguishable from a legitimate rep's own input, since
  no per-individual identity exists to attribute or later dispute a forged entry against
  (an explicit, accepted PRD consequence of the single-shared-login decision, not a gap
  this discipline can close without contradicting the PRD's own NFR).
- **Deny**: drive unbounded `POST /interactions` / `POST /qa` / `POST /briefs` traffic,
  each of which triggers a paid, rate-limited-by-nobody-today Bedrock invocation (TB-5) —
  a cost and availability denial-of-service vector reachable from exactly this boundary.

  Controls: SEC-1 through SEC-5 and SEC-11 (Section 4) exist specifically because this is
  the boundary where "authenticated" vs. "not authenticated" is actually decided, and the
  API contract's `SharedLoginAuth` scheme left that decision entirely open (see Section 3).

**TB-3 / TB-4 (C-API ↔ C-SQL / C-VEC).** Same-process trust zone in this architecture (no
component other than C-API ever calls either store directly — SOL-7, reaffirmed below).
An attacker who has *not* compromised the C-API process itself cannot reach this boundary
at all; an attacker who *has* (e.g. via a server-side vulnerability in C-API, out of this
discipline's scope to threat-model since no such vulnerability class is named by any
story) could read/forge/delete anything either store holds — the same blast radius as a
full C-API compromise, which is a general application-security concern rather than a
distinct boundary-specific finding. Controls: file-permission and disk-encryption
requirements (SEC-9) reduce what a *separate* host-level compromise (not a C-API process
compromise) could recover from the raw files.

**TB-5 (C-API ↔ C-NLU/Bedrock).** This is the one boundary in the whole architecture that
leaves the deployment environment entirely, crossing to a third-party cloud provider. An
attacker positioned on this network path (again, only relevant if TLS were ever bypassed —
the AWS SDK defaults to TLS, so this is a configuration-integrity concern, not a design
gap) could read captured free text and business-card images in transit, or forge/replay
model-invocation requests using a leaked AWS credential. The credential itself — not the
network path — is the realistic attack surface: an over-broadly-scoped IAM credential
compromised anywhere (leaked in a log, committed to a repo, over-permissioned on the host)
grants whatever Bedrock (and any other AWS service the credential happens to also permit)
allows. Control: least-privilege IAM scoping (SEC-6).

**TB-6 (adapter layer).** No real external system exists behind any of the four adapters
in this scope (PRD Section 14.1, reaffirmed by `data_integration` DAT-7) — there is
nothing to threat-model here today. Recorded as a contingent finding (Section 8): the
moment any adapter is pointed at a real CRM/calendar/SharePoint/email system, it becomes a
new instance of a TB-2/TB-5-class boundary (untrusted-network crossing carrying
credentials) that this document has not analysed, because it does not exist yet.

### 2.3 Is SOL-7 sufficient? (analysis requested for this discipline's review)

`solution`'s SOL-7 decision — C-VEC and C-NLU are reachable only from C-API; C-WEB never
calls either directly — is **correct and worth keeping, but it answers a different
question than "is the system secure."** SOL-7 is a *component-reachability* boundary: it
guarantees that whatever compromises C-WEB's own execution context cannot obtain direct
network access to Bedrock or Chroma, cannot see the AWS credential, and cannot run an
arbitrary vector query — every such access is forced through C-API's own defined
operations, which is exactly what makes rate-limiting, input validation, and prompt-
hardening (Section 4) possible to apply at all in one place. That containment value is
real and this document does not propose changing it.

What SOL-7 does **not** do — and does not claim to, since `solution`'s own Section 0 scope
boundary explicitly defers "threats, controls, or measurable security/performance
targets" to this discipline — is constrain what an actor who already holds a valid
`SharedLoginAuth` session can do *through* C-API's own front door. As Section 2.2's TB-2
analysis shows, an attacker positioned at C-WEB (via a stolen session, an XSS bug, or a
malicious script run by a legitimate rep) reaches exactly the same data and exactly the
same mutating operations a real rep would, through C-API's normal, SOL-7-compliant
operations — `GET /accounts/{id}/profile`, `POST /qa`, `POST /briefs`, `POST
/interactions` are all "correctly" routed through the one integration point; SOL-7 was
never designed to stop *authorized-looking* traffic, only to stop C-WEB from bypassing
C-API's logic entirely. An attacker positioned on the network path *between* C-WEB and
C-API (rather than inside the browser) gains the same access the instant they can read or
replay the session cookie — again, something SOL-7 does not touch, because SOL-7 is about
which *component* calls which, not which *credential* is valid.

**Conclusion:** SOL-7 is sufficient for its stated purpose (a single, reasoning-friendly
choke point for retrieval/synthesis logic) and this document does not recommend changing
it. It is not, and was never meant to be, a substitute for an actual authentication/
authorization boundary at TB-2 — which is precisely the gap the API contract's
`SharedLoginAuth` placeholder left open for this discipline to close (Section 3). Treating
SOL-7 as "the" security boundary would be a mistake; TB-2 is.

## 3. Resolving the `SharedLoginAuth` placeholder

The API contract's `SharedLoginAuth` security scheme (`type: apiKey`, `in: cookie`, `name:
session`) is explicitly described in the contract itself as a placeholder: *"the actual
authentication mechanism, token issuance and threat controls belong to the security_nfr
discipline and are not decided by this contract."* This section is that decision.

**Decision (SEC-1):** a single shared application credential (one username/password pair,
consistent with the PRD's Access Control NFR — no per-individual accounts) is presented at
a login step and exchanged for a server-issued, opaque session token. The token is
delivered as the cookie the contract already names (`session`), so no change to any
existing operation's `security` requirement or schema is needed — only the mechanism
behind that cookie is being decided here, not its name or transport. See SEC-1..SEC-5
below for the full mechanism (issuance, storage, lifecycle, CSRF, brute-force protection).

**What this does not do:** it does not choose the actual password value, who holds it, or
how it is distributed to the demo team — that is an operational/human decision, carried
forward as Open Item OI-8 (Section 8), not invented here.

## 4. Controls

### 4.1 Applied controls (recorded as decisions)

| ID | Category | Control |
|---|---|---|
| SEC-1 | Authentication mechanism | Single shared credential validated server-side at a login endpoint; exchanged for an opaque, server-validated session token (not a self-contained JWT — see rationale below) delivered via the `session` cookie the contract already names. |
| SEC-2 | Session lifecycle | Session cookie set `HttpOnly`, `Secure`, `SameSite=Strict`. Sliding idle timeout of 2 hours; absolute session lifetime capped at 12 hours regardless of activity; server-side session store invalidated on logout. |
| SEC-3 | Credential storage & rotation | The one shared credential's secret is stored hashed (Argon2id or bcrypt), never in plaintext in code, config, logs, or this or any other artifact (per root `CLAUDE.md`'s standing rule); rotated on a defined cadence and immediately on any suspected leak or demo-team membership change. |
| SEC-4 | CSRF defense | Every state-changing operation (`POST`/`PATCH`) requires a custom request header (e.g. `X-Requested-With`) that a simple cross-site form submission cannot set, in addition to `SameSite=Strict` — defense in depth, since cookie-based auth without any CSRF control is exploitable even with `SameSite` misconfigured or relaxed later. |
| SEC-5 | Brute-force / credential-stuffing protection | Login endpoint: lockout/backoff after 5 failed attempts within a rolling 15-minute window per source IP. Justified specifically because a single, comparatively low-entropy shared credential is the *entire* perimeter (Section 2.2, TB-2) — there is no second factor and no per-account lockout to fall back on. |
| SEC-6 | C-NLU (Bedrock) egress | IAM credential scoped to `bedrock:InvokeModel` on the specific approved model ARN(s) only — no broader AWS permissions. Credential supplied via `platform`'s secrets mechanism (instance role or secrets manager), never embedded in C-API source or committed config. TLS enforced for all Bedrock traffic (AWS SDK default, verified not disabled). |
| SEC-7 | Upload ingress validation | Business-card image uploads (`multipart/form-data` on `POST /interactions`) capped at 5 MB and restricted to `image/jpeg`/`image/png` content types, enforced by C-API **before** any Bedrock invocation — bounds both request size and the cost/DoS surface a malformed or oversized upload could otherwise reach at TB-5. |
| SEC-8 | Prompt-injection hardening | Every prompt template C-API sends to C-NLU must structurally delimit untrusted, attacker-reachable content (`raw_text`, retrieved `embedded_content` chunks, free-text questions) from the model's system instructions. C-NLU output is treated as advisory synthesis text only — it is never executed, never used to trigger a privileged action, and never merged back into `raw_text`/stored fact without going through the same validated write path every other extraction result uses. |
| SEC-9 | Data-at-rest boundary | The `C-SQL` file and `C-VEC` persistent store are restricted to the `C-API` process's own OS-level user/file permissions; disk-level encryption enabled wherever the hosting environment (`platform`) supports it. **This is a requirement recorded against `platform`, not implemented by this discipline** — see Section 8. |
| SEC-10 | Logging / no-PII-in-logs | Application logs must never include full `raw_text`/free-text payloads, uploaded images, or the session token value at INFO level or above; structured logs may carry IDs and `error_code`s only. Applied now even though the PRD confirms synthetic-data-only for this version (Section 7, Security/Privacy NFR), because the PRD's own Open Questions already flag a possible future real-data phase, and building the logging discipline in now costs nothing and avoids a habit change later. |
| SEC-11 | Cost/DoS rate limiting | `POST /interactions`, `POST /qa`, and `POST /briefs` — every operation that triggers a Bedrock invocation — are rate-limited per session (proposed: 30 requests/minute), independent of the login-specific limit in SEC-5, since a valid session (not just a stolen login) is enough to drive unbounded paid model calls (Section 2.2, TB-2 "Deny"). |
| SEC-12 | Trust-boundary determination | SOL-7 (C-VEC/C-NLU reachable only from C-API) is accepted as sufficient for its stated component-containment purpose and is not revised. The primary authentication/authorization boundary is TB-2 (C-WEB ↔ C-API), which SOL-7 does not itself secure; SEC-1 through SEC-5 and SEC-11 are the controls that actually govern it. See Section 2.3 for the full analysis. |

**Rationale note on SEC-1's "opaque token, not JWT" choice:** a self-contained JWT would
let any future second backend instance validate a session without a shared session store
— a stateless-scaling benefit this architecture does not need, since SOL-1 already fixed
C-API as a single modular-monolith service (not a fleet needing stateless auth), and an
opaque, server-validated token is trivially revocable at logout or on suspected leak,
where a self-contained JWT is not revocable before its own expiry without an extra
denylist mechanism — the exact complexity a single-instance demo does not need to carry.

### 4.2 Controls considered and explicitly rejected

| Control considered | Rejected because |
|---|---|
| Per-individual authentication / role-based access control | The PRD's own Access Control NFR (Section 7) explicitly requires a single shared login and explicitly excludes per-individual authentication as out of scope for this version — this is not a gap this discipline can close without contradicting a confirmed, human-decided requirement. Recorded as an **accepted risk**, not an oversight: Section 2.2's "flat authorization" finding is a direct, named consequence of this NFR, not new information. |
| Mutual TLS between C-API and C-SQL/C-VEC | No network hop exists at TB-3/TB-4 in the declared architecture (same-process/same-host) — mTLS secures a network boundary that is not actually crossed here. Revisit only if `platform` chooses a networked (non-embedded) Chroma deployment mode, which would newly create a TB-4 network hop (flagged as a contingent finding, Section 8). |
| Web Application Firewall / managed bot-detection in front of C-WEB/C-API | No stated requirement or budget signal justifies it for an internal demo with a small, known user population; not cost-justified at this scale. Revisit if C-WEB is ever exposed beyond the demo team. |
| Horizontal scaling / high-availability architecture for C-API | No NFR or `constraints.budget_usd` signal calls for it; consistent with SOL-1's single-instance modular-monolith choice. A single-instance, best-effort availability target (NFR-9, Section 5) is recorded instead of an HA target. |
| A second factor (MFA) on the shared login | No PRD requirement calls for it, and the PRD's own framing treats the single shared credential as the intentional, minimal control for this version. Recorded as an accepted risk alongside "per-individual authentication," for the same reason — SEC-5's brute-force protection is the compensating control actually justified by this scope. |

## 5. NFR targets

| ID | Category | Target | Source | Verification | Verification owner |
|---|---|---|---|---|---|
| NFR-1 | Security / Privacy | System operates only on seeded/synthetic data; no real customer PII in C-SQL or C-VEC, structured or embedded. | **Inherited** — PRD Section 7, Security/Privacy NFR (verbatim). | Manual data-provenance review of the seed dataset before each demo run; code review confirming all four adapters (CRM, Calendar, SharePoint, Email) remain stub-only per PRD Section 14.1/DAT-7, with no real external connection introduced. | Whoever loads seed data, plus `platform` at build review. |
| NFR-2 | Access Control | Single shared login authenticates all use; no per-individual authentication/authorization exists. | **Inherited** — PRD Section 7, Access Control NFR (verbatim). | Confirm exactly one credential set exists in the session store; confirm no per-user role/permission table or per-rep attribution field exists anywhere in the schema. | Security-architect, at Workflow 3 build review. |
| NFR-3 | Concurrency | At least two concurrent shared-login sessions can write to the same customer thread without silently losing either session's data. | **Inherited** — PRD Section 7, Concurrency NFR `[INFERRED — needs confirmation, carried from PRD]`; mechanism resolved by `data_integration`'s DAT-3 (SQLite WAL + busy_timeout + bounded retry, 503/Retry-After on exhaustion). | Automated concurrency test: fire two `POST /interactions` near-simultaneously against the same account; confirm both persist as distinct rows, or the losing request receives `503`/`Retry-After` and a retry succeeds — never a silent drop (ties directly to US-21). | Backend/QA, Workflow 3. |
| NFR-4 | Security (derived) | Login endpoint tolerates and blocks credential-stuffing: lockout/backoff after 5 failed attempts per 15-minute window (SEC-5). | **Derived** — `[INFERRED — needs confirmation]`; no PRD number exists, this discipline set the threshold given TB-2's flat-authorization risk (Section 2.2). | Automated brute-force simulation against the login endpoint; confirm lockout/backoff triggers at the stated threshold and resets appropriately. | Security-architect / backend, Workflow 3. |
| NFR-5 | Security (derived) | Session idle timeout 2 hours; absolute session lifetime 12 hours (SEC-2). | **Derived** — `[INFERRED — needs confirmation]`; no PRD number exists. | Automated session-expiry test (mocked clock or wait); confirm session is rejected after each threshold and a fresh login is required. | Backend, Workflow 3. |
| NFR-6 | Performance (derived) | Typed-note capture (`POST /interactions`, JSON path) completes with p95 latency ≤ 5 seconds. | **Derived** — `[INFERRED — needs confirmation]`; no PRD NFR states a response-time budget (`solution`'s SOL-3 explicitly notes this absence). Set because F-6 AC2's "appear automatically" UX assumes a responsive synchronous round trip. | Latency measurement during a manual/automated walkthrough of the seeded dataset at single-user demo scale — **not** a production-concurrency load test, since none is requested by any story. | Backend/QA, Workflow 3. |
| NFR-7 | Performance (derived) | `POST /qa` and `POST /briefs` (retrieval + synthesis, possibly a vision call) complete with p95 latency ≤ 8 seconds. | **Derived** — `[INFERRED — needs confirmation]`; same absence of a PRD number as NFR-6, higher bound reflecting an added retrieval + synthesis step. | Same walkthrough method as NFR-6. | Backend/QA, Workflow 3. |
| NFR-8 | Security / cost (derived) | Business-card image upload accepted only up to 5 MB and `image/jpeg`/`image/png` (SEC-7). | **Derived** — `[INFERRED — needs confirmation]`; no PRD number exists. | Automated test posting an oversized/unsupported file; confirm `422 UNSUPPORTED_IMAGE_FORMAT` (or a new size-specific error) is returned **before** any Bedrock call is made (verified via call-count = 0 on a stubbed/mocked C-NLU). | Backend, Workflow 3. |
| NFR-9 | Availability (derived, control explicitly scoped down) | Best-effort, single-instance availability; no formal SLA or HA target. | **Derived** — explicit scope decision (Section 4.2), consistent with `constraints.budget_usd: null` framed as "internal demo/hackathon... not a cost-constrained production deployment" and PRD's own success metric being functional completeness, not uptime. | PRD's own stated success metric: manual walkthrough of the full capture → extraction → query → commitment-tracking → brief flow against seeded accounts (PRD Success Metrics, verbatim). | Whoever runs the demo. |
| NFR-10 | Security / cost (derived) | `POST /interactions`, `POST /qa`, `POST /briefs` rate-limited per session at 30 requests/minute (SEC-11), independent of the login-specific limit (NFR-4). | **Derived** — `[INFERRED — needs confirmation]`; no PRD number exists; set to bound Bedrock cost/availability exposure from a single valid session (Section 2.2, TB-2 "Deny"). | Automated rate-limit test: exceed the threshold from one session; confirm `429`-class rejection (or an equivalent backoff response) is returned, and Bedrock invocation count for the throttled requests is 0. | Backend/`platform`, Workflow 3. |

No performance/throughput/availability target above is inherited from a numbered PRD or
Stories statement — none exists in this scope. Every such target (NFR-4 through NFR-10,
7 of the table's 10 rows) is this discipline's own conservative proposal, explicitly
tagged `[INFERRED — needs confirmation]` and carried to Open Items (OI-9) rather than
presented as a confirmed requirement.

## 6. Traceability

Verbatim acceptance criteria are read directly from `workflow-1/state/roles/stories.json`
`stories[].acceptance_criteria`, matching `solution.json`/`data_integration.json`'s own
copies exactly. `operation_ids`/`entity_names` are carried forward from `data_integration`
(now available, unlike at the `solution` stage). `verification_scenarios` below are this
discipline's own — security/NFR-focused, not a restatement of the functional scenarios
`solution`/`data_integration` already recorded.

### F-1 — Conversational Interaction Capture

**US-1**
- acceptance_criteria: ["The system must create a structured interaction note and associate it with the correct customer thread when the rep's typed entry unambiguously matches exactly one seeded customer account.", "The system must save the entry as a new interaction note when the entry contains at least one non-whitespace character."]
- component_ids: C-WEB, C-API, C-SQL, C-NLU
- operation_ids: captureInteraction — entity_names: interactions, accounts
- verification_scenarios: Confirm `POST /interactions` requires a valid `SharedLoginAuth` session (SEC-1); an unauthenticated request returns `401` per the contract's `Unauthorized` response, never a silent `201`. Confirm capture latency meets NFR-6.

**US-2**
- acceptance_criteria: ["The system must reject the submission with an explicit 'cannot save an empty note' message, and must not create any note record, when the rep submits an empty or whitespace-only entry.", "The system must flag the note as 'unmatched — needs customer selection' — rather than attaching it to a default or incorrect account — when no seeded customer account can be unambiguously identified from the entry text."]
- component_ids: C-WEB, C-API, C-NLU, C-SQL
- operation_ids: captureInteraction — entity_names: interactions
- verification_scenarios: Confirm the authentication requirement holds on the rejection path too (400 EMPTY_NOTE still requires a valid session first). Confirm `raw_text` is never written to application logs at INFO level or above (SEC-10).

**US-3**
- acceptance_criteria: ["The system must attach a previously flagged 'unmatched' note to the customer thread the rep selects when the rep manually resolves the flag.", "The system must leave the note in the unmatched/needs-selection state — rather than guessing an account — when the rep has not yet resolved it."]
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: listUnmatchedInteractions, resolveInteractionCustomer — entity_names: interactions
- verification_scenarios: Confirm both operations require a valid session. Confirm the flat single-tenant authorization model (any valid session can resolve any unmatched note to any account) is the accepted design (Section 2.2/4.2), not treated as an undiscovered defect.

**US-4**
- acceptance_criteria: ["[INFERRED — needs confirmation] The system must extract the contact's name, company and role from a scanned business-card image into the structured interaction note when the image is legible and contains recognizable contact fields. The OCR mechanism was never discussed in discovery (PRD Assumption 5, Open Question 5); whether this is worth building for the demo at all is still open.", "[INFERRED — needs confirmation] The system must inform the rep that manual entry is required — without fabricating contact details — when the image cannot be read or no contact fields are recognized."]
- component_ids: C-WEB, C-API, C-NLU, C-SQL
- operation_ids: captureInteraction — entity_names: interactions, interaction_attendees
- verification_scenarios: Confirm the 5 MB / image-content-type ceiling (SEC-7, NFR-8) is enforced before any Bedrock invocation. Confirm the uploaded image is never persisted or logged (data_integration's DAT-8, reaffirmed by SEC-10).

**US-5**
- acceptance_criteria: ["The system must extract every distinct attendee name mentioned in a typed free-text interaction entry as a separate structured attendee item — recording one item per distinct person named (e.g., 'met Priya and Arjun from Acme...' must yield two attendee items, not one merged string) — when the entry text names at least one identifiable person.", "The system must record zero attendee items for that interaction — rather than fabricating a name — when the entry text names no one.", "[INFERRED — needs confirmation] The exact name-recognition/disambiguation mechanism was never discussed in discovery (PRD Assumption 14, Open Question 15); only the underlying capability (AC1/AC2) is treated as confirmed."]
- component_ids: C-API, C-NLU, C-SQL
- operation_ids: captureInteraction — entity_names: interaction_attendees
- verification_scenarios: Confirm the `raw_text` passed to C-NLU for attendee extraction is delimited from system instructions in the prompt template (SEC-8) — this text is attacker-reachable free-form input (Section 2.2, TB-2).

### F-2 — Automatic Extraction & Thread Tagging

**US-6**
- acceptance_criteria: ["The system must extract each distinct commitment, follow-up or next step mentioned in a captured note's text as a separate tracked item when the note contains one or more of them.", "The system must record no commitment items when the note describes discussion only, with no forward-looking commitment."]
- component_ids: C-API, C-NLU, C-SQL
- operation_ids: captureInteraction — entity_names: commitments
- verification_scenarios: Confirm the same SEC-8 prompt-delimiting control applies to commitment extraction as to attendee extraction (US-5) — same `raw_text` source, same risk.

**US-7**
- acceptance_criteria: ["The system must record a due date on an extracted commitment when the captured text states a concrete date or an unambiguous relative date term resolvable against the interaction's timestamp (e.g., 'by Friday').", "The system must mark the commitment's due date as unspecified — rather than guessing a date — when the text uses a vague temporal reference (e.g., 'soon,' 'sometime') or gives no date information at all."]
- component_ids: C-API, C-NLU, C-SQL
- operation_ids: captureInteraction — entity_names: commitments
- verification_scenarios: Same SEC-8 control as US-6. Confirm a fabricated/guessed due date is never treated as a "system decision" that bypasses the null-due-date safeguard already in the schema (`data_integration`'s `Commitment.due_date`).

**US-8**
- acceptance_criteria: ["The system must tag every extracted discussion point and commitment to the same customer thread as its source interaction note when that note is matched to a thread.", "The system must flag an extracted item for manual thread assignment — rather than tagging it to an incorrect thread — when the source note itself is unmatched to a customer thread (per F-1)."]
- component_ids: C-API, C-SQL
- operation_ids: captureInteraction — entity_names: commitments, interactions
- verification_scenarios: Confirm tagging is derived server-side from the already-resolved interaction's own `account_id`, never from a client-supplied value on this internal step — no additional trust-boundary crossing beyond TB-2 is introduced here.

### F-3 — Natural-Language Memory & Q&A

**US-9**
- acceptance_criteria: ["The system must answer a natural-language question about a customer's most recent discussion by returning content drawn from that customer's most recent interaction note(s) when the thread has at least one captured note.", "The system must respond that no history exists yet for that customer when the thread is empty."]
- component_ids: C-WEB, C-API, C-SQL, C-VEC, C-NLU
- operation_ids: askQuestion — entity_names: interactions, embedded_content (Chroma collection)
- verification_scenarios: Confirm `askQuestion` requires a valid session (NFR-2) and meets the NFR-7 latency target. Confirm C-VEC's `account_id` metadata filter is applied **before** any retrieved content reaches C-NLU's synthesis step, so one account's content can never leak into another account's answer (DATA-MODEL Section 2.2).

**US-10**
- acceptance_criteria: ["The system must answer a natural-language question about the rep's own open commitments for a customer by listing the commitments extracted and tagged to that thread when any exist.", "The system must state that there are no open commitments for that customer when none exist."]
- component_ids: C-WEB, C-API, C-SQL, C-NLU
- operation_ids: askQuestion — entity_names: commitments
- verification_scenarios: Same session/latency checks as US-9.

**US-11**
- acceptance_criteria: ["The system must answer a natural-language question about interaction attendees by returning the contact names extracted from that thread's interaction notes when attendee names were captured.", "The system must state that no attendee information was captured when none was extracted."]
- component_ids: C-WEB, C-API, C-SQL, C-NLU
- operation_ids: askQuestion — entity_names: interaction_attendees
- verification_scenarios: Same account-scoping check as US-9 (C-VEC/C-SQL filtered by `account_id` before synthesis).

**US-12**
- acceptance_criteria: ["The system must resolve and answer against the correct customer thread when a query names exactly one customer that matches a seeded account.", "The system must respond that it could not identify the customer — rather than guessing or returning another customer's data — when the query names no customer or an unrecognized one."]
- component_ids: C-WEB, C-API, C-NLU, C-SQL
- operation_ids: askQuestion — entity_names: accounts
- verification_scenarios: Confirm an unrecognized/ambiguous customer resolves to `resolution='not_identified'` and never to any other account's data — a data-isolation control (Section 2.2), not only a UX behavior.

### F-4 — Proactive Commitment Tracking

**US-13**
- acceptance_criteria: ["The system must list every open commitment whose due date has passed as 'overdue' when the current date is past that due date, and must exclude a commitment from this list once it is marked complete.", "[INFERRED — needs confirmation] The system must list every open commitment whose due date falls within the next 7 days as 'due soon,' and must reclassify it as 'overdue' instead once its due date has passed. The 7-day window is a carried-over illustrative placeholder from the PRD, not a confirmed value (PRD Open Question 6).", "The system must list a commitment with an unspecified due date (per F-2) separately from the due-soon/overdue list — rather than omitting it entirely or treating it as overdue — when no due date was captured."]
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: listDueCommitments, markCommitmentComplete — entity_names: commitments
- verification_scenarios: Confirm both operations require a valid session. Confirm `markCommitmentComplete` is idempotent — repeating the call on an already-complete commitment does not error or corrupt state (data_integration's OI-5 mechanism; no new trust boundary introduced by this operation).

**US-14**
- acceptance_criteria: ["The system must state explicitly that there are no due-soon or overdue commitments — rather than showing an empty list with no explanation — when none exist."]
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: listDueCommitments — entity_names: commitments
- verification_scenarios: Confirm the empty-state response path carries the same authentication requirement as the populated-list path — no unauthenticated shortcut for the trivial case.

### F-5 — Brief-Me-on-Customer Summary

**US-15**
- acceptance_criteria: ["The system must generate a summary containing recent discussion history, open commitments and stakeholder/contact names for a named customer when that customer has at least one captured note, and must state that no history exists yet for that customer — rather than generating a summary with fabricated content — when the thread is empty.", "[INFERRED — needs confirmation] The system must return a bounded, short-form summary (a few sentences or bullets, not a multi-page document) under normal conditions, and must still return whatever partial content is available, rather than failing the whole request, when data for one of the summary components is missing (PRD Open Question 7)."]
- component_ids: C-WEB, C-API, C-SQL, C-VEC, C-NLU
- operation_ids: generateBrief — entity_names: interactions, commitments, contacts, interaction_attendees, embedded_content (Chroma collection)
- verification_scenarios: Confirm `generateBrief` applies the same C-VEC account-scoping control as `askQuestion` before synthesis. Confirm NFR-7 latency target.

**US-16**
- acceptance_criteria: ["[INFERRED — needs confirmation] The system must synthesize the opportunity/deal-status component narratively from that customer's recorded interaction notes and commitments — since no discrete deal-stage field exists in the structured store — when the thread contains content indicating where the deal/opportunity stands (PRD Assumption 13, Open Question 14).", "The system must state that no opportunity/deal-status information has been captured yet for that customer — rather than fabricating a stage or outcome — when the thread contains no such content."]
- component_ids: C-API, C-SQL, C-VEC, C-NLU
- operation_ids: generateBrief — entity_names: embedded_content (Chroma collection), interactions, commitments
- verification_scenarios: Confirm narrative synthesis never fabricates an opportunity/deal stage when underlying retrieval is empty — an accuracy control tied to SEC-8's "advisory output only" principle, not a confidentiality control.

**US-17**
- acceptance_criteria: ["The system must generate the summary for a named customer when the request unambiguously identifies exactly one seeded customer account.", "The system must respond that it could not identify the requested customer — rather than guessing — when the request names an unrecognized or ambiguous customer."]
- component_ids: C-WEB, C-API, C-NLU, C-SQL
- operation_ids: generateBrief — entity_names: accounts
- verification_scenarios: Same data-isolation check as US-12 — an unrecognized/ambiguous customer never resolves to another account's brief.

### F-6 — Customer Profile / Conversational History View

**US-18**
- acceptance_criteria: ["The system must display a customer's captured notes and extracted items in chronological order on that customer's profile view when at least one note exists.", "The system must display an explicit 'no history yet' state when none exists."]
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: getAccountProfile — entity_names: interactions, interaction_attendees, commitments
- verification_scenarios: Confirm `getAccountProfile` requires a valid session. Confirm no cross-account leakage is possible by construction of the query (`idx_interactions_account_captured` scoping in `data_integration`'s data model) — an interaction not tagged to the requested `account_id` is never returned.

**US-19**
- acceptance_criteria: ["The system must display a newly captured note and its extracted items on the correct customer's profile view automatically, without requiring the rep to take further action, when the note is tagged to that customer's thread.", "The system must not display that note or its items on any other customer's profile view when it is not tagged to that customer."]
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: captureInteraction, getAccountProfile — entity_names: interactions
- verification_scenarios: Same cross-account isolation check as US-18, exercised across the capture-then-read sequence.

### F-7 — Shared Customer Thread Access

**US-20**
- acceptance_criteria: ["The system must show the same customer thread content to every session authenticated via the shared login when two sessions view the same customer.", "The system must not partition data by which physical person is at the keyboard, since no per-rep identity exists in this version."]
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: listAccounts, getAccountProfile — entity_names: accounts, interactions
- verification_scenarios: Confirm identical-content-across-sessions (F-7 AC1) holds because both sessions are independently, validly authenticated against the one authoritative store (NFR-2) — not because authentication was skipped or weakened for either session.

**US-21**
- acceptance_criteria: ["[INFERRED — needs confirmation] The system must persist a note added from one session so that it is immediately visible to another concurrent session viewing the same customer thread. The conflict-handling mechanism was never discussed in discovery (PRD Open Question 9).", "[INFERRED — needs confirmation] The system must not silently lose one session's addition when another session adds to the same thread at nearly the same time."]
- component_ids: C-WEB, C-API, C-SQL
- operation_ids: captureInteraction — entity_names: interactions
- verification_scenarios: NFR-3's concurrency test (Section 5) — confirm both concurrent captures persist as distinct rows, or the losing request receives `503`/`Retry-After` with a successful retry; never a silent drop and never one session's payload bleeding into the other's record.

**Coverage check:** all 21 stories across all 7 features carry component_ids and a
security/NFR-specific verification_scenario. The `operation_ids`/`entity_names` shown
per-story above are the real mapping against `data_integration`'s approved API
contract, independently checked and confirmed accurate. However, in this document's
companion state file (`security_nfr.json`), those same structured `operation_ids`/
`entity_names` fields are recorded as `[]` with a `not_applicable` note on all 21 rows —
**not** because the mapping is unknown, but as a disclosed workaround for a genuine
limitation in this repo's narrowed single-discipline `check-artifacts` tool, which
cannot resolve another discipline's operation IDs unless the discipline being checked
is `data_integration` itself. See `security_nfr.json`'s `audit_trail` for the full
diagnosis; the Validator for this stage independently confirmed the underlying mapping
is correct.

**Coverage gap found during this review (returned to `data_integration`):** SEC-1
(login), SEC-5 (login lockout) and SEC-11 (per-session rate limiting, `429` responses)
all require API operations — a login endpoint, a logout endpoint, and `429`-class
rate-limit responses on `POST /interactions`, `POST /qa`, `POST /briefs` — that do not
exist in `data_integration`'s already-approved API contract. This is a real gap, not a
tool limitation; it cannot be fixed inside this document since it requires changing an
already-gated artifact this discipline does not own. It is recorded in
`security_nfr.json`'s own `returned_findings` array (`finding_id: V-SEC2`, `to_role:
"data_integration"`) for resolution at workflow sign-off, per this framework's
documented mechanism for a finding one stage hands to another.

## 7. Decisions summary

| ID | Category | Status |
|---|---|---|
| SEC-1 | Authentication mechanism (resolves `SharedLoginAuth`) | proposed |
| SEC-2 | Session lifecycle (cookie flags, idle/absolute timeout) | proposed |
| SEC-3 | Credential storage & rotation policy | proposed |
| SEC-4 | CSRF defense for cookie-based auth | proposed |
| SEC-5 | Login brute-force / credential-stuffing protection | proposed |
| SEC-6 | C-NLU (Bedrock) egress — least-privilege IAM | proposed |
| SEC-7 | Business-card upload ingress validation | proposed |
| SEC-8 | Prompt-injection hardening for C-NLU calls | proposed |
| SEC-9 | Data-at-rest / file-permission boundary (requirement on `platform`) | proposed |
| SEC-10 | Application logging / no-PII-in-logs policy | proposed |
| SEC-11 | Cost/DoS rate limiting on NLU-triggering endpoints | proposed |
| SEC-12 | Trust-boundary determination (SOL-7 sufficiency finding) | proposed |

All twelve are `status: "proposed"` — none is `user_confirmed`; nothing here has been
built, and every one is subject to the `security_nfr_review` human gate.

## 8. Open items

| ID | Item | Blocks development? | Owner | Resolution needed |
|---|---|---|---|---|
| OI-8 | SEC-1 proposes the authentication *mechanism*; it does not choose the actual shared credential value, who holds it, how it is distributed to the demo team, or the concrete rotation cadence (SEC-3 names the policy, not the schedule). | No | Product owner / whoever administers the demo | Confirm the actual credential, its distribution, and rotation cadence before Workflow 3 build. |
| OI-9 | No PRD or Stories acceptance criterion states a numeric performance/rate-limit target. NFR-4 through NFR-10 (login lockout threshold, session timeouts, latency targets, upload size ceiling, rate-limit thresholds) are this discipline's own conservative, inferred proposals, not confirmed by a human. | No | Product owner / Architect | Confirm or revise each specific numeric target in Section 5 before it is treated as a committed requirement in Workflow 3. |
| OI-13 | SEC-1/SEC-5/SEC-11 require login, logout, and per-session-rate-limited (`429`) API operations that do not exist in `data_integration`'s already-approved API contract — a real cross-discipline gap, not something this discipline can fix in its own artifact. | Yes — blocks a complete Workflow 3 build of the authentication/rate-limiting controls this document specifies | `data_integration` (at workflow sign-off) | `data_integration`'s API contract needs a login endpoint, a logout endpoint, and documented `429` responses on `POST /interactions`/`POST /qa`/`POST /briefs`. Recorded as a `returned_findings` entry (`V-SEC2`) in `security_nfr.json` for the human to action via `revise: data_integration ...` at workflow sign-off. |
| OI-10 | Contingent on solution's OI-3 (business-card OCR technology): if OI-3 resolves to introduce a dedicated OCR/vision technology as a sixth stack component (rather than Bedrock's own multimodal models), this document's trust-boundary analysis (Section 2, TB-5-class reasoning) must be redone for the new component before development proceeds — a new external network boundary and a new data-handling review would be needed, not just a technology swap. | No (contingent — does not block today's scope) | Security-architect (contingent on OI-3's resolution) | Re-run Section 2's trust-boundary and Section 5's NFR analysis if OI-3 resolves to a new component. |
| OI-11 | `platform`'s eventual Chroma deployment-mode decision (embedded vs. networked server, SOL-9's own deferral) changes TB-4's threat profile: an embedded Chroma has no network hop to secure; a networked Chroma server would newly need the mTLS/network-isolation control this document currently rejects in Section 4.2 as inapplicable. | No (contingent) | `platform` | Confirm the deployment mode; if networked, revisit TB-4 and Section 4.2's mTLS rejection. |
| OI-12 | TB-6 (adapter layer) is not a real network boundary today (all four adapters are stub-only, PRD Section 14.1/DAT-7). The moment any adapter is pointed at a real external system, it becomes a new TB-2/TB-5-class boundary this document has not analysed. | No (contingent — no real adapter exists in this scope) | Security-architect (contingent), whenever a real adapter is introduced | Re-run a trust-boundary analysis for each real adapter before it goes live. |

**Inherited open items checked for a security dimension (none carried forward as new
security open items beyond what is captured above):**
- **solution's OI-1** (PRD file line-ending drift) — checked; **no security dimension**.
  Purely a hashing/representation artifact, already verified content-identical and
  disclosed by `solution`; not re-raised here.
- **solution's OI-4** (attendee-extraction mechanism inferred as prompt-driven against
  C-NLU) — checked; its security dimension (prompt-injection exposure from free-text
  input) is exactly what SEC-8 already covers. Not a separate open item.
- **data_integration's OI-5** (commitment-completion mechanism, `PATCH
  .../complete`) — checked; no new trust boundary or authentication gap — the operation
  carries the same `SharedLoginAuth` requirement as every other mutating operation. The
  only note worth recording is the idempotency check already captured in US-13's
  verification_scenarios above.
- **data_integration's OI-6** (Q&A/Brief-Me UX flow — account-id hint vs. free-text
  resolution) — checked; **no security dimension** — a client-side UX/routing choice with
  no effect on authentication, authorization, or data isolation, since both paths resolve
  to the same server-side account-scoping logic (Section 6, US-9/US-15 verification).
- **data_integration's OI-7** (profile chronological-order direction) — checked; **no
  security dimension** — a display-order preference with no effect on which data is
  returned or to whom.

## 9. Self-verification

- **Every trust boundary in the solution design is analysed, including the ones nobody
  asked about.** PASS — Section 2.1 names all six (TB-1 through TB-6, including the
  same-process C-SQL/C-VEC boundaries and the currently-inert adapter-layer boundary),
  Section 2.2 states what an attacker at each can read/forge/deny, and Section 2.3
  directly answers the requested SOL-7-sufficiency question rather than treating it as
  already settled.
- **Every control is either applied, or rejected with a recorded reason.** PASS —
  Section 4.1 (12 applied controls) and Section 4.2 (5 explicitly rejected controls, each
  with a stated reason and a named revisit condition where one exists).
- **Every NFR target names its source and distinguishes inherited from derived.** PASS —
  Section 5's table has an explicit Source column on every row; 3 rows are marked
  **Inherited** (verbatim PRD NFRs), 7 are marked **Derived** and additionally tagged
  `[INFERRED — needs confirmation]` inline, none is presented as a confirmed number.
- **Every target has a stated verification method and owner.** PASS — Section 5,
  Verification and Verification owner columns populated for all 10 rows.
- **No control or target is asserted as met — nothing has been built.** PASS — every
  decision in Section 7 is `status: "proposed"`; Section 5's verification column
  describes what *would* demonstrate each target, in future tense, not a claim that any
  test has already run; this document is a design artifact, not a build/test report.
- **`source_references` records every upstream artifact consumed, with hashes.** PASS —
  Section 1, 7 entries (product_state, prd, user_stories, solution, and three
  data_integration bundle files), all hashes computed directly by this discipline via
  `sha256sum` and cross-checked to match exactly what `solution.json` and
  `data_integration.json` themselves recorded — no new drift found.
- **Every inference is marked `[INFERRED — needs confirmation]` and carried as an open
  item.** PASS — every derived NFR target (Section 5) and every genuinely open mechanism
  question (Section 8, OI-8 through OI-12) is tagged and carried forward; none is stated
  with the confidence of a confirmed fact.

**Result: PASS.**

This self-verification is not independent review. The shared Validator runs separately,
afterwards, against this same draft.

## 10. Readiness

`ready` — every trust boundary in the approved `solution`/`data_integration` design has
been analysed (including the explicitly-requested SOL-7 sufficiency question), the
`SharedLoginAuth` placeholder has a concrete proposed resolution (SEC-1 through SEC-5),
every control is either applied or rejected with a recorded reason, every NFR target
names its source and a verification method, and all 21 stories in scope carry a
security/NFR-specific traceability row. One open item, OI-13, is marked
`blocks_development: true` — not because this discipline's own artifact is incomplete,
but because it names a real gap in `data_integration`'s already-approved API contract
(missing login/logout/rate-limit operations) that a full Workflow 3 build of SEC-1/
SEC-5/SEC-11 genuinely cannot proceed without; it is recorded as a `returned_findings`
entry for `data_integration` to resolve at workflow sign-off, not something this
discipline can close in its own artifact. Every other open item is either genuinely
contingent on an upstream decision this discipline does not own (OI-10, OI-11, OI-12)
or a numeric/operational confirmation that does not block Workflow 3 from starting
against the proposed mechanism (OI-8, OI-9). Readiness here means this discipline's own
artifact is complete and internally consistent, not that every cross-discipline gap it
found is already resolved.
