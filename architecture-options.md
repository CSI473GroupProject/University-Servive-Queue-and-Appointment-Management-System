# Architecture Alternatives — Comparison

**Project:** Student Registration Queue Management System ;Team 17
**Lab:** CSI473 Laboratory 7
**202105048 M.W.W Keimetswe 202105048**
**202105212 P Morapedi**
**Applies to:** Team's own approved project (Phase 1, approved 13/08/2026) — not the University Service Hub example

## Architecture drivers used as comparison criteria

- **AD-01 — Responsive access under concurrent peak load.** Source: QS-01 (2-second queue position response), QS-06 (500 concurrent users, 95% under 3 seconds).
- **AD-02 — High availability with reliable state changes.** Source: QS-02 (99% availability during registration periods), QS-04 (99.9% of queue-joining transactions recorded correctly, no loss/duplication).
- **AD-03 — Secure, role-based access delivered within limited team resources.** Source: QS-03 (account lockout after 5 failed attempts) and the project constraint (Phase 1 §2) that the team has limited time/resources and must avoid unnecessary personal-data collection.

A secondary constraint, modifiability (QS-07: availability changes must apply within 1 minute, no restart), is treated as a design obligation on the Appointment Service rather than a fourth structural driver, since it affects how one component reads configuration rather than the overall shape of the system.

## Alternative 1: Layered monolithic web application

A single deployable application structured into presentation, application/service, domain and data
access layers, backed by one relational database. The Student, Registration Staff and Administrator
front ends (UC-01–UC-18) are separate interfaces calling the same backend services.

**Against AD-01:** Adequate at this project's real scale — a single university's registration periods,
not a multi-tenant national system. Shared runtime is a risk only if a heavy operation (e.g. UC-18 View
Reports and Activity) is allowed to compete with queue/appointment request handling.

**Against AD-02:** Strong fit. A single database means UC-02 (Join Virtual Queue) and UC-06 (Book
Appointment) stay within one transactional boundary — there is no risk of a queue entry being saved
while its notification silently fails to be recorded, which directly protects QS-04's 99.9% correctness
target.

**Against AD-03:** Strong fit. One codebase means one authentication/authorisation implementation
(supporting UC-01's login precondition and the role checks in UC-12–UC-18) rather than duplicated or
federated security logic across services — realistic for a five-person team on a semester timeline.

**Consequences (negative accepted):** All functionality shares one runtime, so an unoptimised reporting
query (UC-18) could in principle slow down queue operations (UC-04, QS-01) if not isolated. This is the
named risk carried into ADR-001's reconsideration trigger.

## Alternative 2: Microservices architecture

Separate independently deployable services for Queue, Appointment, Notification, Authentication and
Reporting, communicating over a network, each with its own datastore.

**Against AD-01:** Good fit at large scale — the Queue service could scale independently of Reporting.
But this project's QS-06 target (500 concurrent users) does not require independent scaling; a
monolith comfortably meets it.

**Against AD-02:** Poor fit. Splitting Queue and Notification into separate services with separate
datastores turns UC-02's single-transaction queue-entry creation (Main Success Flow steps 5–9) into a
distributed transaction — a queue entry could be persisted while its UC-11 notification is lost,
directly threatening QS-04.

**Against AD-03:** Poor fit given team constraints. Service discovery, inter-service auth, and multiple
deployments are exactly the "custom security infrastructure" AD-03 says to avoid, and none of the five
team members has prior operational experience with this at the Phase 1 constraint's stated resourcing.

## Alternative 3: Serverless / function-based architecture

Individual cloud functions triggered per request (e.g. `joinQueue`, `bookAppointment`, `notify`), backed
by a managed database.

**Against AD-01:** Mixed. Scales automatically for QS-06's concurrency spikes, but cold-start latency
during low-traffic periods threatens QS-01's 2-second response target for something as simple as UC-04
Monitor Queue Status.

**Against AD-02:** Neutral-to-weak. A managed database avoids some distributed-transaction risk, but
coordinating UC-02's queue-number generation (UC-03, which requires an atomic increment per BR-06) across
independently invoked functions needs careful design the team has not scoped.

**Against AD-03:** Poor fit given team constraints. No team member has serverless tooling experience;
local testing and debugging (needed to verify UC-06's exception 4a slot-race handling) are materially
harder than in a conventional deployed application.

## Comparison summary

| Criterion | Alt 1: Layered Monolith | Alt 2: Microservices | Alt 3: Serverless |
|---|---|---|---|
| AD-01 (performance/scale, QS-01/QS-06) | Adequate | Best (at scale not needed here) | Mixed (cold starts) |
| AD-02 (availability/reliability, QS-02/QS-04) | Best | Weak (distributed txns) | Weak-to-neutral |
| AD-03 (security within team resources, QS-03) | Best | Weak | Weak |

## Decision

**Alternative 1, the layered monolithic web application, is selected.** It best satisfies AD-02 and
AD-03 without meaningfully giving up on AD-01 at the concurrency level this project actually needs to
support. Full rationale, accepted negative consequences, and the reconsideration trigger are recorded in
`decisions/ADR-001-architecture.md`.
