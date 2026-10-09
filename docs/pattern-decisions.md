# Pattern Decisions

**Project:** Student Registration Queue Management System — Team 17
**Lab:** CSI473 Laboratory 9 — Patterns, interfaces, detailed design and framework decisions
**Branch:** lab-09
**Date:** 9 October 2026
**Applies to:** the team's own approved project (Phase 1, approved 13/08/2026). Core workflow: **UC-06 Book Appointment** (the `bookAppointment` operation specified in Lab 8), with UC-08/UC-09 cancellation and UC-11 notification.

Each technique below starts from a design problem that exists in this project and names the
requirement, use case, business rule or quality scenario that makes it a problem. Patterns that
did not earn a place are recorded in the rejection section.

## Design problems identified

| ID | Design problem | Where it appears | Named source |
|---|---|---|---|
| DP-1 | **Persistence coupling.** `reserveSlot` must read slot and appointment state and save a booking, but the outcome of the race in UC-06 exception 4a is a database `UNIQUE(slot_id)` violation. If the service sees SQL/JPA/Spring exceptions, business logic and persistence technology become entangled and the race cannot be unit tested. | `AppointmentService` ↔ database | UC-06, FR-06, FR-07, BR-02, BR-04, QS-04, report ADR-03, Lab 8 failure-recovery |
| DP-2 | **Secondary effect inside a critical transaction.** The booking confirmation (UC-06 step 5 → UC-11) must never roll back, delay or duplicate a booking. | booking commit ↔ notification | UC-11, QS-04, report ADR-04, Lab 8 data-integrity rule "notification does not block or roll back the originating transaction" |
| DP-3 | **External dependency with a fallback.** Students are notified in-app first and by email if that fails; SMS is a possible later channel. The gateway is outside the trust boundary and can fail. | `NotificationService` ↔ in-app / email / Notification Gateway | UC-11 extension 3a and exception 3a, report ADR-04, Lab 7 failure domain, Lab 8 deployment |
| DP-4 | *Variable behaviour — considered, not a real problem:* the call-next rule (UC-14) and the cancellation cut-off (UC-09). | `StaffOperationsService`, cancellation check | UC-14, UC-09, Phase 1 glossary |
| DP-5 | *Object creation — considered, not a real problem:* `Appointment` is created in exactly one place. | `AppointmentService.reserveSlot` | UC-06 |

## Pattern-decision table

| Decision | Problem | Technique | Participants (in the class model and `src/interfaces-or-ports/`) | Why it fits this project | Cost accepted | Test that confirms it |
|---|---|---|---|---|---|---|
| **PD-1** | DP-1 | **Repository (port)** with exception translation | `AppointmentRepository` (port), `JpaAppointmentRepository` (adapter, Phase 2), `AppointmentService`, `Appointment`, `AppointmentSlot`, `BookingException.SlotAlreadyBooked` | The service states what it needs (`findSlot`, `slotHasActiveAppointment`, `save`) in business terms. The adapter turns the database's unique-constraint violation into `SlotAlreadyBooked`, so the race result is an ordinary domain outcome mapped to `409 SLOT_ALREADY_BOOKED` in the Lab 8 contract. The database constraint stays the final arbiter. | One extra interface; JPA entity classes plus a small mapper in the adapter (records cannot be JPA entities); the adapter must translate the exception correctly or the guarantee silently weakens. | (a) Unit test of `AppointmentService` with an in-memory fake repository; (b) integration test of `JpaAppointmentRepository.save` against a real database inserting a second appointment for one `slot_id` and expecting `SlotAlreadyBooked`; (c) the Lab 8 two-thread concurrency test. |
| **PD-2** | DP-2 | **Domain event (Observer) delivered after commit** | `AppointmentBooked` (event), `DomainEventPublisher` (port), `SpringDomainEventPublisher` + `@TransactionalEventListener(AFTER_COMMIT)` (adapter, Phase 2), `NotificationService` | The booking transaction ends when the row commits; notification runs after that and cannot undo it. If `save` throws (the losing request in the race), no event is published, so the losing student is never told they have a booking. | Control flow is indirect (harder to trace in a debugger). Events live in memory, so a crash between commit and delivery loses the notification; the student still sees the booking in "My Appointments" (UC-10). Persisting the `NOTIFICATION` row (Lab 8 data model) so undelivered messages surface on next login is a Phase 2 task. | (a) Forced `save` failure publishes zero events — **checked against the skeleton in this lab** (the losing request in the race produced no event). (b) A failing listener leaves the booking committed — **to be tested in Phase 2**, because it needs the Spring adapter and a real transaction. |
| **PD-3** | DP-3 | **Adapter port with ordered fallback** | `NotificationSender` (port), `InAppNotificationSender`, `EmailNotificationSender` (adapters, Phase 2), `NotificationService` | The email provider's API stays behind one interface. Fallback order (in-app then email) is a list given to `NotificationService`, matching report ADR-04. Adding SMS later means one new adapter, no core change. | One interface per channel; the order lives in configuration; every adapter must report failure through `DeliveryResult` or an exception rather than hiding it. | (a) In-app sender throws, email sender delivers: `onAppointmentBooked` returns `true` and logs the fallback — **checked against the skeleton in this lab**. (b) All senders fail: should return `false` and log the UC-11 exception 3a condition — **to be added as a unit test in Phase 2**. |

The cancellation rule also has an interface (`CancellationPolicy`) but is **not** counted as a
pattern: it is a boundary onto configuration data, with a single implementation (see R-2).

## Explicit rejection decisions

### R-1 — External message broker (RabbitMQ / Kafka) for events — rejected
- **Problem it would solve:** reliable, durable delivery of `AppointmentBooked` to notification handling (the in-memory event loss named under PD-2).
- **Why rejected:** the system is one university's registration periods on a single application node with one database (ADR-001, QS-06 at 500 concurrent users). A broker adds another server to deploy, secure, monitor and keep available against the 99% availability target (QS-02), a new failure mode (broker down while the database is up), and another thing five students must learn within a semester (AD-03). It would also pull the deployment toward the microservice style ADR-001 already rejected.
- **Consequence accepted:** possible loss of an in-memory notification on a crash between commit and delivery.
- **Cheaper mitigation:** persist a `NOTIFICATION` row (status `queued`) in the same transaction as the booking, and let a scheduled job retry anything still `queued`. This keeps durability inside the single database.
- **Revisit if:** notification volume or reliability needs outgrow the in-process approach, or the system is split into separately deployed services (the ADR-001 reconsideration triggers).

### R-2 — Strategy hierarchies for the call-next rule (UC-14) and the cancellation cut-off (UC-09) — rejected
- **Problem it would solve:** variable behaviour behind a common interface.
- **Why rejected:** the behaviour does not vary. The Phase 1 service rule is one fixed rule (due appointment first, otherwise earliest waiting queue number), and the cancellation cut-off is a duration that is *configured* per service with a University-wide default. Different values are data, not different algorithms, so a `DefaultCancellationPolicy` / `ServiceSpecificCancellationPolicy` pair, or a `CallNextStrategy` family, would be classes with nothing to differ on: accidental complexity and extra indirection to trace.
- **What is built instead:** one `ConfiguredCancellationPolicy` behind the `CancellationPolicy` interface (read the service's cut-off, otherwise the default), and the call-next rule as plain code in the staff-operations service.
- **Consequence accepted:** if the University later adds genuinely different rules (for example priority groups), the call-next logic must be refactored then.
- **Revisit if:** a second distinct call-order or cancellation algorithm is actually required by a stakeholder.

### Also considered and not adopted
| Candidate | Why not |
|---|---|
| GoF **State** pattern for `AppointmentStatus` | The state model has five states and a handful of transitions; `AppointmentStatus.canTransitionTo` in one enum already enforces BR-08 in a single place. Five state classes would scatter that rule. |
| **Factory** for `Appointment` | One construction site (`reserveSlot`); a factory would add a class without removing duplication. |
| **CQRS** / separate read model for queue position | QS-01's 2-second target is met by an indexed query at this scale (ADR-001, report ADR-02 polling). Revisit only on ADR-001 reconsideration trigger 1. |
| Interface-per-service (`IAppointmentService`) | One implementation exists; the controller depends on the class. A port is added only where an external or persistence dependency needs one. |

## Consistency with related artefacts
- Method names, error codes and check order match `docs/api-contracts/core-operation.yaml` (Lab 8).
- `BookingException` codes are exactly `VALIDATION_ERROR`, `SLOT_NOT_FOUND`, `SLOT_UNAVAILABLE`, `DUPLICATE_ACTIVE_APPOINTMENT`, `SLOT_ALREADY_BOOKED`; `CANCELLATION_DENIED` is new for UC-08 and sits outside the Lab 8 contract.
- Components match the Lab 7 component model (Appointment Service, Notification Service, repositories, Notification Gateway) and the Lab 8 deployment (single application node, single database, external gateway).
- Status values and transitions match the Phase 1 state model (Section 7.2).

## Exit record

**Pattern or interface boundary with the greatest benefit:** the **`AppointmentRepository` port with exception translation (PD-1)**. It turns the concurrency risk named in Lab 8 (two students booking one slot) into an ordinary, unit-testable domain outcome while the database `UNIQUE(slot_id)` constraint remains the real guarantee.

**Its cost:** a JPA entity-plus-mapper layer in the adapter and a translation step that must be right. If the adapter fails to translate the violation, the service would surface a framework exception instead of `409 SLOT_ALREADY_BOOKED`.

**Test that will confirm the design works:** an integration test, against a real database, that saves one appointment for a slot and then a second for the same `slot_id`, asserting that `save` throws `BookingException.SlotAlreadyBooked` and that exactly one row remains. It is run with the Lab 8 two-thread test so the translation is exercised under genuine concurrency. A unit test of `AppointmentService` with a fake repository covers the rest of the rule order.
