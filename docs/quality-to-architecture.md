# Quality-to-Architecture Traceability

**Project:** Student Registration Queue Management System;Team 17
**Lab:** CSI473 Laboratory 7 ;Architecture alternatives and component structure
**202105048 M.W.M Keimetswe**
**202105212 P Morapedi**
**Branch:** lab-07
**Source:** Quality scenarios QS-01–QS-07 (Phase 1 report, Section 5); use cases UC-01–UC-18 and functional requirements FR-01–FR-10 (Phase 1 report, Section 4)


| Quality Scenario | Measurable Response Level | Design Obligation | Driver | Architecture Element(s) Responsible | Related Use Cases / FRs |
|---|---|---|---|---|---|
| **QS-01** Performance ;logged in student requests queue position under normal load | Response displayed within 2 seconds | Queue position calculation must be cheap to run on every request; must not require a full queue scan under normal concurrency | AD-01 | Queue Service (position calculation logic); QueueRepository (indexed lookup on service + status) | UC-04 Monitor Queue Status, UC-05 Get Queue Position, FR-04 |
| **QS-02** Availability ;student attempts to access the system during a registration period | ≥99% availability during registration periods | Architecture must avoid a single point of failure that takes down queue and appointment access together | AD-02 | Layered monolith runtime (ADR-001) deployed with redundancy; Presentation layer front ends remain reachable even if a background service (e.g. reporting) degrades | UC-02, UC-04, UC-06, UC-09; FR-02, FR-05 |
| **QS-03** Security ; repeated failed login attempts against a student account | Account locked after 5 consecutive failed attempts, minimum 15 minutes | Authentication must be framework-provided, not custom-built, and must be enforced before any request reaches business logic | AD-03 | Authentication and Access Control component (shared, cross-cutting across Presentation and Application/service layers) | UC-01 (login precondition); all role-gated use cases (UC-12–UC-18) |
| **QS-04** Reliability ; student successfully submits a request to join a queue | 99.9% of queue-joining transactions recorded correctly, no data loss or duplication | Queue entry creation must be an atomic, transactional operation; queue number generation must not create duplicates under concurrent requests | AD-02 | Queue Service + QueueRepository (atomic increment, per ADR-001/UC-03 concurrency handling); single relational database (no distributed transaction) | UC-02 Join Virtual Queue, UC-03 Generate Queue Number, FR-02, FR-03, BR-06 |
| **QS-05** Usability ;first-time student joins a virtual queue unaided | ≥90% of first-time users complete the process within 3 minutes | Queue-joining workflow (UC-01 → UC-02 → UC-03) must be a short, linear front-end flow with no unnecessary steps | (UI/UX concern, not structural) | Student Web/Mobile Interface (Presentation layer) | UC-01, UC-02, FR-01, FR-02 |
| **QS-06** Scalability ;up to 500 concurrent students request queue positions or book appointments | 95% of requests complete within 3 seconds at 500 concurrent users | Application/service layer must not let one slow operation (e.g. reporting) block queue/appointment request handling; read-heavy operations should be isolatable from write-heavy ones | AD-01 | Queue Service, Appointment Service (kept independent of Administration Service reporting queries per ADR-001 consequence) | QS-01 (compounding), UC-04, UC-06, UC-07 |
| **QS-07** Modifiability ; administrator changes a service's appointmentslot duration or availability window | Change reflected in new availability queries within 1 minute, no restart, no effect on existing bookings | Availability configuration must be read fresh (or cache-invalidated) on each query rather than baked in at startup | Secondary driver (treated as a constraint on the Appointment Service) | Appointment Service + AvailabilityService logic; ServiceRepository | UC-17 Manage Appointment Availability, UC-07 Check Appointment Availability |


