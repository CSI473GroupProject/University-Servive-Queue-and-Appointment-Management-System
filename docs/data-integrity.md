# Data Integrity — Constraints and Traceability

**Project:** Student Registration Queue Management System Group 17
**Lab:** CSI473 Laboratory 8 Data, API, deployment and failure aware design
**Branch:** lab-08
**Date:** 2 October 2026

## Scope

This document records the integrity constraints enforced by the logical data model
(`models/logical-data-model.mmd`), how each is protected at the database and application level, and the
test needed to verify it — closing the loop from Lab 7's architecture to this lab's data/API/deployment
design.

## Integrity constraints

| Constraint | Enforcement mechanism | Named rule / scenario | Enforced at | Test |
|---|---|---|---|---|
| A student has at most one active (Waiting/Called) QueueEntry per service | Partial unique index on `(student_id, queue_id)` filtered to `status IN ('Waiting','Called')`, re-checked in `QueueService.checkExistingEntry` before insert (UC-02 step 2) | BR-04, UC-02 AF-02 | Database + application | Attempt two `joinQueue` calls for the same student/service; assert the second returns the existing queue entry, not a new one |
| A queue number is unique per service per calendar day | Unique index on `(queue_id, queue_number, booking_date)`, generated via atomic increment (UC-03) | BR-06, UC-03 | Database (atomic increment) | Fire concurrent `joinQueue` requests for the same service; assert all assigned queue numbers are distinct with no gaps or duplicates |
| An appointment slot can have at most one active Appointment | Unique constraint on `appointment.slot_id` for active (Booked/Called) rows | BR-02, UC-06 exception 4a | Database (final arbiter) + application re-check (ADR-003) | See `models/failure-recovery.mmd` concurrency test |
| A student has at most one active Appointment per service | Composite check on `(student_id, service_id via slot)` restricted to Booked/Called status | BR-04 | Application (`AppointmentService.reserveSlot`) | Attempt to book two slots for the same student/service; assert the second is rejected with `DUPLICATE_ACTIVE_APPOINTMENT` |
| A completed Appointment or QueueEntry cannot be modified or reopened | State machine transition guard — only Waiting→Called/Cancelled and Called→Served/No-show are valid transitions (Phase 1 §7.2 state models) | BR-08 | Application (status transition logic) | Attempt to cancel or re-call a `Served` appointment; assert rejection, state unchanged |
| Queue-joining and appointment-booking transactions do not lose or duplicate data | Single relational database, single transaction per operation (no distributed transaction) — ADR-001 | QS-04 (99.9% correctness) | Database (ACID transaction) | Load test: N concurrent `joinQueue`/`bookAppointment` calls; assert row count matches successful-response count exactly |
| Account lockout after repeated failed logins | `failed_login_count` incremented per failure, `locked_until` set after 5th failure, checked before any business logic runs | QS-03 | Application (Authentication & Access Control) | Attempt 6 failed logins; assert 6th and subsequent attempts are rejected with `ACCOUNT_LOCKED` even with a correct password, until `locked_until` elapses |
| Notification delivery does not block or roll back the originating transaction | `NotificationService.sendBookingConfirmation` is called *after* the Appointment/QueueEntry commit, not within the same transaction; failure is logged and falls back to the next channel (ADR-004) | UC-11, QS-04 | Application (post-commit, async) | Simulate Notification Gateway failure during a booking; assert the Appointment is still committed and visible, and a fallback/log entry is recorded |

## Consistency with the domain model

No entity, relationship, or cardinality in `models/logical-data-model.mmd` differs from the domain model
established in Phase 1 §6 (multiplicities A–H). This lab adds only persistence-level detail: primary
keys, foreign keys, status enumerations, and the explicit unique constraints above that implement business
rules BR-01 through BR-08. Where a business rule required a mechanism not explicit in the domain model
(e.g. the partial unique index for BR-04, or the post-commit notification pattern for UC-11), that
addition is named and justified above rather than left implicit.

## Exit record

**Integrity/failure risk:** Two students can attempt to book the same appointment slot at nearly the same
instant (UC-06 exception 4a). Without protection, this risks a double-booking — two Appointment rows
referencing one slot, violating BR-02.

**Design mechanism that addresses it:** A database-level unique constraint on `appointment.slot_id` for
active appointments, combined with an application-level availability re-check immediately before commit
(ADR-003's optimistic concurrency approach). The database constraint is the final arbiter: even if the
application-level check is stale, the second `INSERT` is rejected by the database itself, guaranteeing
exactly one active Appointment per slot at all times. Full flow in `models/failure-recovery.mmd`.

**Test needed to verify the mechanism:** A concurrency test that fires two `bookAppointment` requests for
the same `slotId` from separate threads/clients with no artificial delay between them, then asserts: (1)
exactly one `201 Booked` response is returned across both calls; (2) exactly one `409
SLOT_ALREADY_BOOKED` response is returned; (3) the Appointment table contains exactly one row for that
`slot_id` after both calls complete; (4) no Appointment row exists in a partial or invalid state.
