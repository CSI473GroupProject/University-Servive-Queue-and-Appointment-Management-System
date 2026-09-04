# Candidate Domain Concepts — Extraction & Classification
**University of Botswana Service Queue and Appointment Management System — CSI473 Lab 04, Studio Step 1 (00–18 min)**
**Source:** Laboratory 3, Group 17 (functional requirements FR-01–FR-10, actor goals, UC-01 "Join Virtual Queue," acceptance criteria, use case diagram)

This worksheet does two things the brief asks for in the first studio block:
1. **Extracts** every candidate noun, role, event, and rule mentioned or implied in the Lab 03 artefacts.
2. **Classifies** each one, and explicitly **rejects** the terms that describe the interface or the technology rather than the problem domain, so they don't leak into the domain model in the 18–50 min block.

---

## 1. Candidate concepts, by domain-modelling category

Categories follow the standard "list of category names" technique (things, roles, events, transactions, rules, containers) so nothing is missed and nothing is admitted just because it's a noun.

| # | Candidate term | Where it comes from | Category | Verdict | Rationale |
|---|---|---|---|---|---|
| 1 | **Student** | Actor; FR-01–08 | Role | ✅ Keep | Primary actor who initiates queueing and appointment behaviour — has domain responsibilities (join, cancel, hold a queue position). |
| 2 | **Registration Staff** | Actor; FR-09–10 | Role | ✅ Keep | Acts on queues/appointments; distinct responsibilities from Administrator. |
| 3 | **Registration/Service Administrator** | Actor; diagram | Role | ✅ Keep | Configures services and appointment availability — a distinct role from Staff. |
| 4 | **Registration Service** | FR-01, FR-05; "Select Registration Service" | Concept (description) | ✅ Keep | The thing being queued for/booked against (e.g. transcript requests, ID replacement). Core domain concept, not a UI screen. |
| 5 | **Virtual Queue** | FR-02; UC-01 | Concept (container) | ✅ Keep | Holds an ordered set of Queue Entries for one service; has a capacity and open/closed state. |
| 6 | **Queue Entry** | UC-01 main flow step 5; postconditions | Concept (transaction) | ✅ Keep | The record created when a student joins — distinct from the Student and from the Queue itself. |
| 7 | **Queue Number** | FR-03; UC-01 step 6 | Concept (value/identifier) | ✅ Keep | A unique identifier assigned to a Queue Entry — a domain value, not just a UI label. |
| 8 | **Queue Position** | FR-04; UC-01 step 7 | Concept (value, derived) | ✅ Keep | Calculated rank of a Queue Entry within a Virtual Queue — worth an attribute/derived value, not a screen field. |
| 9 | **Queue Status** | FR-04; UC-01 step 9 | Concept (value/state) | ✅ Keep | State of a Queue Entry (e.g., waiting, called, completed) — needed for the state-based business rules in block 50–70. |
| 10 | **Appointment Slot** | FR-05–07 | Concept (thing) | ✅ Keep | A bookable time slot for a service — exists independently of any booking. |
| 11 | **Appointment** | FR-06, FR-08; diagram | Concept (transaction) | ✅ Keep | The booking itself, linking a Student to an Appointment Slot — distinct from the Slot. |
| 12 | **Cancellation Rule / Cancellation Policy** | FR-08; "Validate Cancellation Policy" | Rule/Policy | ✅ Keep | Named business policy governing when an Appointment can be cancelled — becomes one of the ≥6 business rules. |
| 13 | **Eligibility (student eligibility for a queue/service)** | UC-01 step 4; "Call Next Eligible Student" | Rule/Policy | ✅ Keep | A named rule concept, not a UI check — governs who may join a queue or be called next. |
| 14 | **Service Rule (call-next rule)** | FR-10 | Rule/Policy | ✅ Keep | Distinct policy from the cancellation rule — decides call order (e.g., FIFO, priority). Needs its own name once the team defines it. |
| 15 | **Notification (queue/appointment update)** | Diagram: "Receive Notifications" | Concept (event/message), borderline | ⚠️ Keep, narrowly | Keep the *business event* "student is notified of a status change." Drop the delivery mechanism (SMS/email/push) — that's technology, see §2. |
| 16 | **Report / Activity Record** | "View Reports and Activity" | Concept, borderline | ⚠️ Defer | Only keep if the team's requirements actually define a persisted report entity. As written it reads as a UI/query feature, not a domain thing — revisit once revised requirements are on hand. |
| 17 | **One-active-entry-per-service rule** | UC-01 "BUSINESS RULE" | Rule/Policy | ✅ Keep | "A student cannot have more than one active queue entry for the same registration service" — an invariant, feeds block 50–70 directly. |
| 18 | **Join Virtual Queue (event)** | UC-01 | Event | ✅ Keep | Noteworthy domain event — creates a Queue Entry. |
| 19 | **Book Appointment (event)** | FR-06 | Event | ✅ Keep | Creates an Appointment against a Slot. |
| 20 | **Cancel Appointment (event)** | FR-08 | Event | ✅ Keep | Governed by Cancellation Rule. |
| 21 | **Call Next Eligible Student (event)** | FR-10 | Event | ✅ Keep | Changes Queue Entry status; governed by the Service Rule. |
| 22 | **Complete Student Service (event)** | Diagram | Event | ✅ Keep | Closes out a Queue Entry/Appointment. |

---

## 2. Terms rejected as interface or technology terms

These appeared in the Lab 03 wording but describe **the software's interface, an action a user performs on a screen, or an implementation mechanism** — not a problem-domain idea — so they're deliberately left out of the domain model, per the brief's own minimum standard ("not merely a screen, controller, framework type or database table").

| Rejected term | Where it appears | Why it's excluded |
|---|---|---|
| **System** | Every FR ("the system shall…") | The application itself — a boundary/actor in the use-case sense, not a domain class. |
| **Select** (as in "Select Registration Service") | FR-01, actor goal | A UI interaction verb. The underlying domain concept it points to — *Registration Service* — is kept; the click/tap isn't. |
| **Display / View** (as in "system displays…", "View Queues", "View Appointments", "View Appointment Details") | FR-04, FR-09, diagram | Presentation behaviour — showing data on a screen — not a domain responsibility. The underlying data (Queue Position, Appointment) is already captured above. |
| **Screen / Screenshot** | Implied by "display" language; explicitly ruled out in the Lab 04 brief itself | Interface artefact, not a domain concept. |
| **Login / access the system** | UC-01 precondition 1 ("The student can access the system") | Authentication/authorisation mechanism — infrastructure, not domain. |
| **Button / menu / form field** (implied by "selects", "chooses") | UC-01 flow | UI widget vocabulary, not present as nouns but worth flagging so the team doesn't introduce them while diagramming. |
| **Notification delivery channel (SMS/email/push)** | Not stated explicitly, but implied by "Receive Notifications" | If the team's revised requirements specify a channel, that's a technology/integration detail — keep the *event* (student notified), drop the *transport*. |

---
