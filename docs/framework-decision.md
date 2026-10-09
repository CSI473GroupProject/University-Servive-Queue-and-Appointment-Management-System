# Framework Decision and Dependency Management

**Project:** Student Registration Queue Management System — Team 17
**Lab:** CSI473 Laboratory 9
**Branch:** lab-09
**Date:** 9 October 2026
**Decision:** Implement the layered monolith (ADR-001) with **Java and Spring Boot**.

**Status of the details below.** Java with Spring Boot is the team's confirmed choice. Items marked
**(proposed)** are recommendations the team has not yet confirmed (database product, build tool,
schema-migration tool, test database). Confirm or replace them before Phase 2; the architectural
rationale does not depend on them.

## Why a framework, and why this one

| Driver | How Spring Boot responds |
|---|---|
| **AD-03** secure, role-based access with limited team resources (QS-03) | Spring Security provides authentication, the three roles (Student, Registration Staff, Administrator) and method-level authorisation as framework features, so the team configures security instead of writing it. The account lockout after 5 failed attempts for at least 15 minutes (QS-03) is implemented as a small, tested component on top of it, using `failed_login_count` and `locked_until` from the Lab 8 data model. |
| **ADR-001** layered monolith, one deployable | A single executable Spring Boot application with an embedded web server matches the one application node in the Lab 8 deployment diagram. |
| **AD-02 / QS-04** reliable state changes | Declarative transactions (`@Transactional`) over one relational database give the single-transaction booking the Lab 8 failure-recovery design depends on. |
| **Lab 8 API contract** | Spring Web maps the contract's endpoint, Bean Validation produces `400 VALIDATION_ERROR`, and one exception handler maps each stable `errorCode` to its HTTP status. |
| **Team and schedule** (Phase 1 constraint) | The team has chosen Java; Spring Boot has extensive documentation and a conventional project layout, which lowers onboarding cost for five members in one semester. |

## Framework boundary (what may depend on Spring)

| Package | May import Spring / JPA / HTTP? | Reason |
|---|---|---|
| `ports` | **No** | Interfaces owned by the core must not leak framework types. |
| `domain` | **No** | Plain records, enums and exceptions; business rules (BR-08 transitions) testable without a container. |
| `application` | Annotation-only (`@Transactional`, optionally `@Service`) | Metadata that can be removed without changing logic. Verified: the skeletons in `src/interfaces-or-ports/` compile with the JDK alone. |
| `adapter.web` | Yes | `@RestController`, `@ControllerAdvice`, Bean Validation. |
| `adapter.persistence` | Yes | Spring Data JPA entities, mappers, exception translation. |
| `adapter.events`, `adapter.notification` | Yes | Event publishing, in-app and email senders. |
| `config` | Yes | Wires ports to adapters and builds `AppointmentService` / `NotificationService` as beans. |

**Rule:** dependencies point inward. Adapters import ports; ports never import adapters.

## Dependency-management approach

1. **One build file, one BOM.** Use **Maven (proposed)** with the Spring Boot parent so Spring-managed library versions come from one tested set; the team does not pin them by hand. If the repository was already initialised with Gradle, keep Gradle; the rationale is identical.
2. **Java 17 or later.** Spring Boot 3.x requires Java 17+. Commit the chosen release in the build file so every member and CI use the same one.
3. **Starters instead of loose libraries.** Add a dependency only with a named need, recorded in the table below. A pull request that adds a dependency updates this table.
4. **Commit the build file; never commit `target/` or secrets.** Database passwords and gateway credentials come from environment variables or an untracked profile file, not from the repository.
5. **Check the tree before merging a dependency change** (`mvn dependency:tree`) to catch unexpected transitive libraries.

### Approved dependencies for Phase 2

| Dependency | Needed for | Source requirement |
|---|---|---|
| `spring-boot-starter-web` | REST adapter for the Lab 8 contract | FR-01–FR-10, Lab 8 API |
| `spring-boot-starter-validation` | `400 VALIDATION_ERROR` on missing or malformed `studentId`/`slotId` | Lab 8 API |
| `spring-boot-starter-data-jpa` | Persistence adapter, transactions | QS-04, AD-02, PD-1 |
| `spring-boot-starter-security` | Authentication, roles, lockout hook | QS-03, AD-03 |
| Relational database driver — **PostgreSQL (proposed)** | The single datastore from ADR-001 / Lab 8 | AD-02 |
| Flyway **(proposed)** | Versioned schema so the Lab 8 unique constraints exist in every environment | BR-02, BR-04, BR-06 |
| `spring-boot-starter-test` (JUnit 5, Mockito) | Unit and integration tests | QS-04 verification |
| A real-database test setup, e.g. Testcontainers **(proposed, needs Docker)** or an embedded database in PostgreSQL mode | The unique-constraint and concurrency tests | PD-1 tests, Lab 8 concurrency test |

**Caution on the test database.** An embedded database can behave differently from the production
database on locking and constraint timing. The concurrency test in Lab 8 is only trustworthy if run
against the same product as production, or if its limits are stated when reported.

## How the framework meets the key design decisions

These sketches show intent and are **not compiled in this lab**; they belong to Phase 2.

**Persistence adapter translates the constraint violation (PD-1):**
```java
@Repository
class JpaAppointmentRepository implements AppointmentRepository {
    public Appointment save(Appointment a) {
        try {
            return mapper.toDomain(jpa.saveAndFlush(mapper.toEntity(a)));   // flush so the constraint fires here
        } catch (DataIntegrityViolationException ex) {
            throw new BookingException.SlotAlreadyBooked();                 // core never sees Spring's exception
        }
    }
    // ... other methods
}
```

**One handler gives every error a stable meaning (Lab 8 contract):**
```java
@RestControllerAdvice
class ApiExceptionHandler {
    @ExceptionHandler(BookingException.class)
    ResponseEntity<ErrorResponse> handle(BookingException e) {
        HttpStatus status = switch (e.errorCode()) {
            case "VALIDATION_ERROR" -> HttpStatus.BAD_REQUEST;
            case "SLOT_NOT_FOUND"   -> HttpStatus.NOT_FOUND;
            default                 -> HttpStatus.CONFLICT;   // SLOT_UNAVAILABLE, SLOT_ALREADY_BOOKED, DUPLICATE_ACTIVE_APPOINTMENT
        };
        return ResponseEntity.status(status).body(new ErrorResponse(e.errorCode(), e.getMessage()));
    }
}
```
(`ACCOUNT_LOCKED` / 423 is produced by the security layer before the request reaches this handler, as in the contract.)

**Notification after commit (PD-2):**
```java
@Component
class NotificationEventListener {
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    void on(AppointmentBooked event) { notificationService.onAppointmentBooked(event); }
}
```
A listener running after commit that needs to write to the database (to persist the `NOTIFICATION` row) must do so in a new transaction (`REQUIRES_NEW`), because the original one has completed.

## Explicit rejection decision

### Rejected: Spring Cloud, a message broker and reactive stack (WebFlux)
- **What was considered:** the Spring Cloud family (service discovery, gateway, config server), a message broker for events, and Spring WebFlux for non-blocking request handling, as ways to handle the 500-user peak (QS-06).
- **Why rejected:** ADR-001 already chose a single deployable. Spring Cloud and a broker exist to coordinate several services and add servers whose availability counts against QS-02. WebFlux changes the programming model for every layer (reactive types, harder debugging and testing) to solve a throughput problem the project does not have: 95% of requests within 3 seconds at 500 concurrent users is within reach of a conventional blocking server with indexed queries and a connection pool. The cost lands on five students with one semester (AD-03); the benefit is not needed at this scale. The message broker is analysed in more depth as R-1 in `docs/pattern-decisions.md`.
- **Consequence accepted:** a ceiling on concurrent connections per node. Throughput must be measured in Phase 2 against QS-01 and QS-06 rather than assumed.
- **Revisit if:** load tests show blocking request threads are the bottleneck (ADR-001 reconsideration trigger 1), or the system is split into separately deployed services.

## Traceability
Requirements and quality scenarios served: FR-06, FR-07, FR-08; UC-06, UC-08, UC-09, UC-11; BR-01, BR-02, BR-04, BR-05, BR-08; QS-01, QS-02, QS-03, QS-04, QS-06; AD-02, AD-03. Related artefacts: `decisions/ADR-001-architecture.md` (Lab 7), `docs/api-contracts/core-operation.yaml` and `models/deployment.mmd` (Lab 8), `docs/pattern-decisions.md`, `decisions/ADR-002.md`, `src/interfaces-or-ports/`.
