# User Profile and Task Scenarios

**Project:** Student Registration Queue Management System — Team 17
**Lab:** CSI473 Laboratory 10 — User-interface, accessibility and usability design
**Branch:** lab-10
**Applies to:** the team's own approved project (Phase 1, approved 13/08/2026)

**Evidence note.** The profiles below are built from the Phase 1 stakeholder table (S2), scope
assumptions and use cases. Phase 1 lists student interviews/questionnaires (04 Aug 2026) and
observation of the registration process (28 Jul 2026) as evidence.

## Primary user profile — Student seeking registration help

| Aspect | Description | Source |
|---|---|---|
| Who | A University of Botswana student who needs registration assistance (for example course registration or a registration enquiry), especially in busy registration periods. | Phase 1  and 2 stakeholders |
| Needs | A convenient way to get registration help without spending long periods in a physical line; to know when they will be served. | Phase 1 and 2 (need, concern) |
| Main frustration | Long waiting and uncertainty about when they will be served; waiting interferes with classes and study. | Phase 1 and 2 problem statement |
| Device | A phone, computer or other internet-enabled device. Mostly a phone **(assumption)**, so the design is mobile-first and must also work on a desktop browser. | Phase 1 and 2 assumptions |
| Experience | Often a first-time user of the system, with no training (QS-05 context); comfortable with common apps but not with this one. | QS-05 |
| Context of use | At or near the registration service point, possibly standing, one hand on the phone, in daylight or a crowded corridor **(assumption)**. Interrupted easily, so state must survive leaving the screen. | **(assumption)** |
| Constraints | Time pressure; may have patchy or costly data **(assumption)**, so screens must be light and recover from lost connections. | **(assumption)** |

**Design implications:** few steps to the key action; large touch targets; status shown in words
as well as colour; every message states what happened and what to do next; the screen tells the
student when its information was last refreshed.

## Secondary user profile — Registration Staff

| Aspect | Description | Source |
|---|---|---|
| Who | A member of registration staff operating a counter. | Phase 1 §2, glossary (Counter) |
| Needs | Manage many waiting students efficiently; call the next eligible student by the service rule; record the outcome quickly. | Phase 1 §2, UC-12–UC-15 |
| Main concern | Difficulty managing large numbers of students at once during busy periods. | Phase 1 §2 |
| Device | A desktop or laptop at the counter **(assumption)**. | **(assumption)** |
| Context of use | Repeating the same cycle (call, serve, complete) many times an hour while talking to a student, so actions must be fast, predictable and hard to trigger by accident. | UC-14, UC-15 frequency "Very high" |
| Data shown | Student ID only, not names or other details, in line with the Phase 1 constraint to avoid unnecessary personal data. | Phase 1 §2 constraints, UC-13 |

(The Administrator, UC-16–UC-18, is not wireframed in this lab; the minimum standard asks for at
least two roles or task perspectives, and this lab covers Student and Registration Staff.)

## Tasks

### Task A — Student joins the virtual queue (primary task)

**Scenario.** It is the first week of registration. Thandi has a question about her course
registration and sees a long line at the registration office. She opens the system on her phone,
signs in, and wants to get in line without standing there.

**Use cases and requirements:** UC-01 Select Registration Service, UC-02 Join Virtual Queue,
UC-03 Generate Queue Number, UC-04 Monitor Queue Status; FR-01, FR-02, FR-03, FR-04; BR-04.

**Starting point:** the student is signed in and on the services list (UC-01 precondition).

**Success criteria**
1. The student reaches a confirmation showing **queue number, position and estimated wait** (UC-02 step 4).
2. They complete it **unaided** and, for the formal requirement, **within 3 minutes** (QS-05: at least 90% of first-time users).
3. The confirmation appears within the performance limits (QS-01: position shown within 2 seconds under normal load).
4. No duplicate entry is created (BR-04); if one exists the student sees the existing entry (UC-02 AF-02).
5. The student can say **what happens next** (they will be notified when called) without being told.

**Shortest reasonable path (design target):** 1 action after the services list (press **Join queue**
on the service row); the status screen is the result. Details in `models/navigation.mmd`.

### Task B — Student books an appointment (vertical-slice task)

**Scenario.** Thandi prefers a fixed time. She wants a slot for course registration tomorrow
morning, and the slot she picks may have just been taken by someone else.

**Use cases and requirements:** UC-01, UC-06 Book Appointment, UC-07 Check Appointment
Availability; FR-05, FR-06, FR-07; BR-01, BR-02, BR-04; the Lab 8 `bookAppointment` contract.

**Success criteria**
1. The student chooses a date and an available slot and receives a booking confirmation (UC-06 step 5, `201 Booked`).
2. If the slot was just taken (`409 SLOT_ALREADY_BOOKED`, UC-06 exception 4a), the student understands why, sees a refreshed list, and books another slot **without re-entering anything unnecessary**.
3. The student can find the cancellation rule before confirming (FR-08, UC-09).

**Shortest reasonable path (design target):** 3 actions after the services list: **Book appointment**,
choose a time, **Confirm booking** (plus one for choosing a different date).

### Task C — Registration staff call the next student and complete service (secondary-user task)

**Scenario.** It is mid-morning at Counter 3. A member of staff finishes with one student and
needs to call the next one for Course registration, then record the outcome.

**Use cases and requirements:** UC-12 View Queues, UC-14 Call Next Eligible Student, UC-15 Complete
Student Service; FR-09, FR-10; the Phase 1 service rule (a due appointment first, otherwise the earliest waiting queue number).

**Success criteria**
1. Staff call the next student with **one action** from the dashboard, and the screen shows who was called (student ID, queue number or appointment time) and from where (queue or appointment).
2. Staff can record **Complete** in **one action**, and **No-show** in one action plus a confirmation (No-show is terminal and cannot be reopened, BR-08); the two cannot be confused (different wording and position, not colour alone).
3. When no one is waiting or due, the screen says so plainly instead of failing (UC-14 exception 1a).
4. The student's identity shown is limited to the student ID.

**Shortest reasonable path (design target):** 2 actions per student (**Call next**, then **Complete**;
**No-show** adds one confirmation). *The target of 2 actions is the team's design goal; it is not a requirement
stated in Phase 1.*

## Design decisions about navigation and structure

| Decision | Alternative considered | Reason and consequence accepted |
|---|---|---|
| Show **Join queue** and **Book appointment** inline on each service row | A separate "next options" screen after selecting a service (UC-01 step 4 as written) | One fewer step on the shortest path, which helps the QS-05 3-minute target. Consequence: each service row is busier, so row layout and labels must stay clear. The use case behaviour (presenting the next options) is preserved, only the screen differs. |
| Staff dashboard is the **home screen** after sign-in and Call next sits on it | Staff navigate from a menu to a "Call next" page | Task C happens many times an hour (UC-14/UC-15 "Very high"); no navigation between cycles. Consequence: the dashboard must stay uncluttered. |
| **Call next is disabled while a student is called at that counter**, with the reason shown | Allow calling again at any time | Prevents leaving a called record unresolved (UC-15 business rule: only a Called record at that counter can be completed). **Design assumption not stated in Phase 1; confirm with the team.** |
| After a **booking conflict** the time list refreshes and the old selection is cleared | Keep the selection and retry the same slot | Matches the Lab 8 design (no retry of the same slot) and UC-06 exception 4a. |



## Traceability — screens to requirements

| Screen (wireframe frame) | Task | Use cases | Functional requirements | Quality scenarios / rules |
|---|---|---|---|---|
| S1 Services list | A, B | UC-01 | FR-01 | QS-05, QS-01 |
| S2 Queue status | A | UC-02, UC-03, UC-04, UC-05, UC-11 | FR-02, FR-03, FR-04 | QS-01, QS-05, QS-04; BR-04, BR-06 |
| S3 Choose appointment time | B | UC-06, UC-07 | FR-05, FR-06 | QS-07 (availability changes appear within 1 minute), BR-01 |
| S4 Booking conflict error | B | UC-06 exception 4a | FR-07 | QS-04; BR-02; Lab 8 `409 SLOT_ALREADY_BOOKED` |
| T1 Staff dashboard with a called student | C | UC-12, UC-14, UC-15 | FR-09, FR-10 | QS-01, QS-06; BR-07, BR-08 |
| T2 Staff empty and stale states | C | UC-12, UC-14 exception 1a | FR-09, FR-10 | QS-02 |
| Sign-in and locked-account message | A, B, C | UC-01 | — | QS-03 (lock after 5 failed attempts for at least 15 minutes) |

UI-to-API mapping for Task B (Lab 8 contract): `400 VALIDATION_ERROR` → field error; `404
SLOT_NOT_FOUND` → refresh list; `409 SLOT_UNAVAILABLE` → "no longer open", refresh; `409
SLOT_ALREADY_BOOKED` → conflict alert (S4); `409 DUPLICATE_ACTIVE_APPOINTMENT` → existing
appointment message; `423 ACCOUNT_LOCKED` → locked-account message. Full message text is in
`docs/ui-states.md`.

## Related artefacts
`models/navigation.mmd` / `.svg` · `prototype/wireframes-student.svg`, `prototype/wireframes-staff.svg` · `docs/ui-states.md` · `docs/accessibility-checklist.md` · `evidence/usability-test.md`
