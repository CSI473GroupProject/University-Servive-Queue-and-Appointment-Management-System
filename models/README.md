# Lab 07 Evidence — Architecture Alternatives and Component Structure

**Project:** Student Registration Queue Management System — Team 17
**Team members:** Motoya Keimetswe (202105048), Phenyo Morapedi (202105212)
**Branch:** lab-07
**Date:** 25 September 2026

## Applied-project confirmation

All evidence below is built directly from Team 17 approved project (Phase 1, approved 13/08/2026):
the Student Registration Queue Management System, its functional requirements FR-01–FR-10, use cases
UC-01–UC-18, quality scenarios QS-01–QS-07, business rules BR-01–BR-08, domain model and Phase 1 
architecture work.

## Evidence index

| File | What it shows | Named traceability |
|---|---|---|
| `docs/quality-to-architecture.md` | Each quality scenario (QS-01–QS-07) translated into a design obligation, driver (AD-01–AD-03), and owning architecture element | QS-01–QS-07, AD-01–AD-03, UC-01–UC-18, FR-01–FR-10 |
| `docs/architecture-options.md` | Three realistic architecture alternatives compared on the same criteria; decision rationale | AD-01–AD-03, QS-01–QS-07, UC-02, UC-06, UC-18 |
| `models/component-architecture.mmd` / `.svg` | Component/module view: responsibilities, dependency direction, external system (Notification Gateway), data store, security and failure boundaries | UC-01–UC-18, FR-01–FR-10, QS-03/AD-03 |
| `decisions/ADR-001-architecture.md` | Context, decision, alternatives, positive and negative consequences, and an explicit reconsideration trigger tied to measurable evidence | QS-01–QS-04, QS-06, FR-02/03/09/10, UC-01–UC-04, UC-14, UC-15, UC-18, BR-06 |

## Exit record

**Which quality requirement most influenced the architecture?**
**QS-04 (Reliability)** — the requirement that 99.9% of queue-joining transactions (UC-02) be recorded
correctly, with no data loss or duplication — was the single strongest factor in selecting the layered
monolith over microservices. A distributed-transaction split between the Queue Service and Notification
Service would have put this reliability target at direct risk (see `docs/architecture-options.md`,
Alternative 2 assessment against AD-02). QS-01/QS-06 (performance/scalability) and QS-03 (security within
team resources) were also decisive, but QS-04 is what most directly ruled out the microservices
alternative.

**What evidence would cause the team to revise the decision?**
As recorded in ADR-001's reconsideration trigger: measured queue-position response times exceeding the
QS-01 2-second target, or fewer than 95% of requests completing within 3 seconds at 500 concurrent users
(QS-06) — specifically when this coincides with UC-18 reporting activity — would indicate the accepted
shared-runtime risk has materialised and the reporting workload needs to be isolated, or the architecture
reconsidered.

## Commit note

Editable model source (`component-architecture.mmd`) and its readable export (`.svg`) are both committed,
per the lab requirement that screenshots alone are not acceptable source evidence.
