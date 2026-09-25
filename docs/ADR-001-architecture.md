# ADR-001: Overall Architecture Style

**Status:** Accepted
**Date:** 25 September 2026
**Project:** Student Registration Queue Management System - Team 17
**202105048 M.Keimetswe**
**202105212 P.Morapedi**

## Context

The system must let students select a registration service and either join a virtual queue (UC-02) or
book an appointment (UC-06), let registration staff call and serve students (UC-14, UC-15), and let
administrators configure services and view reports (UC-16-UC-18).

Three named quality scenarios most strongly shape the architecture:

- **QS-01 / QS-06 (performance & scalability):** queue position must return within 2 seconds; 95% of
  requests must complete within 3 seconds at 500 concurrent users.
- **QS-02 / QS-04 (availability & reliability):** >99% availability during registration periods; 99.9%
  of queue-joining transactions (UC-02) must be recorded correctly, with no loss or duplication.
- **QS-03 (security), read together with the team-resource constraint:** accounts lock after 5 failed
  logins; the team must rely on well-understood, framework-provided security rather than build its own.

## Decision

Adopt a **layered monolithic web application** - presentation, application/service, domain and data
access layers, one relational database - as detailed in `docs/architecture-options.md` (Alternative 1)
and diagrammed in `models/component-architecture.mmd` / `.svg`.

## Alternatives considered

1. **Microservices architecture** (Queue, Appointment, Notification, Authentication, Reporting as
   separate deployable services, each with its own datastore).
2. **Serverless / function-based architecture** (per-request cloud functions with a managed database).

## Consequences

**Positive:**
- UC-02 (Join Virtual Queue) and its queue-number generation (UC-03, business rule BR-06) stay within a
  single database transaction, directly supporting QS-04's 99.9% correctness target — there is no
  distributed-transaction failure mode to design around.
- One authentication implementation covers all three roles (Student, Registration Staff, Administrator;
  UC-01 precondition, UC-12-UC-18 role checks), which is realistic for a five-person team and satisfies
  AD-03 without building custom security infrastructure.
- Development, testing and deployment stay within tooling the team already knows, reducing schedule risk
  for the one-semester timeline.

**Negative (accepted):**
- All functionality shares one runtime. An unoptimised reporting query (UC-18 View Reports and Activity)
  could in principle compete for database and application resources with queue operations (UC-04, QS-01)
  if reporting is not kept read-only and isolated from the live transactional tables.
- The architecture does not allow the Queue Service to scale independently of the rest of the system,
  which would matter at a scale beyond a single university's registration periods — judged acceptable
  because QS-06's target (500 concurrent users) does not require it.

## Reconsideration trigger

This decision should be revisited if any of the following evidence appears during Phase 2 performance
testing or production use:

1. **Load-test evidence against QS-01/QS-06:** if measured queue-position response time exceeds the
   2-second target (QS-01), or fewer than 95% of requests complete within 3 seconds at 500 concurrent
   users (QS-06), specifically while UC-18 reporting queries are running concurrently with UC-04/UC-05
   queue operations. This would indicate the shared-runtime risk named above has materialised.
2. **Reliability evidence against QS-04:** if queue-joining transactions (UC-02) show any measured data
   loss or duplication under concurrent load, indicating the single-database transactional approach is
   insufficient - though this is the least likely trigger, since Alternative 1 was chosen specifically to
   avoid this failure mode.
3. **Scale evidence against QS-06:** if actual registration-period concurrency is shown (via UC-18
   activity reports) to regularly exceed the 500-user design point this decision was scoped against.

If trigger 1 occurs, the first mitigation to try is isolating reporting reads onto a replica or cached
read model (already flagged as a candidate mitigation in `docs/architecture-options.md`) before
considering a structural change to Alternative 2 or 3.

## Traceability

Named requirements/use cases informing this decision: FR-02, FR-03, FR-09, FR-10; UC-01, UC-02, UC-03,
UC-04, UC-14, UC-15, UC-18; BR-06. Named quality scenarios: QS-01, QS-02, QS-03, QS-04, QS-06.
