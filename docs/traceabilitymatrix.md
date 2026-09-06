# Traceability Matrix
Student Registration Queue Management System — Lab 04

Covers: requirement → use case → analysis element → verification, with the governing business rule shown first.
All eight business rules (BR-01–BR-08) are now traced; BR-04 required a wording revision and a new FR-11.

| Business rule | Req | Requirement | Use case | Analysis element | Verification |
|---|---|---|---|---|---|
| — | FR-01 | Select a registration service | Select Registration Service | `Student`, `Registration Service` | Select each available service; confirm correct context is set for subsequent actions |
| BR-03 | FR-02 | Join a virtual queue | Join Virtual Queue | `Student`, `Virtual Queue`, `Queue Entry` | Join a queue for an available service; confirm `Queue Entry` created. Attempt to join a closed/unavailable queue; confirm rejection |
| BR-06 | FR-03 | Assign a unique queue number | Generate Queue Number *(«include»)* | `Virtual Queue`, `Queue Entry` | Join queue with multiple students; confirm queue numbers/positions are unique and follow join order |
| BR-06 | FR-04 | View current queue position/status | Monitor Queue Status → Get Queue Position | `Queue Entry` | Query position after another entry ahead is called/completed; confirm position updates correctly |
| BR-01 | FR-05 | View available appointment slots | Book Appointment → Check Appointment Availability | `Appointment Slot` | Query slots for a service; confirm only available (unbooked) slots are returned |
| BR-01 | FR-06 | Book an available slot | Book Appointment | `Appointment`, `Appointment Slot` | Book an available slot; confirm `Appointment` created and slot marked unavailable |
| BR-02 | FR-07 | Prevent booking an unavailable/already-booked slot | Book Appointment → Check Appointment Availability | `Appointment Slot`, `Appointment` | Attempt to double-book the same slot concurrently; confirm second request rejected |
| BR-05, BR-08 | FR-08 | Cancel an appointment per cancellation rules | Cancel Appointment → Validate Cancellation Policy | `Appointment` | Cancel a not-yet-completed appointment (succeeds); attempt to cancel a completed one (rejected) |
| BR-07 | FR-09 | Staff view active queues and appointments | View Queues, View Appointments | `Virtual Queue`, `Queue Entry`, `Appointment`, `Registration Staff` | Log in as staff for Service A; confirm no queue/appointment data for Service B is visible |
| BR-06 | FR-10 | Staff call the next eligible student | Call Next Eligible Student | `Virtual Queue`, `Queue Entry`, `Registration Staff` | Call next; confirm entry with earliest queue position and "waiting" status is selected, then transitions to "called" |
| BR-04 (revised) | FR-11 | Prevent a student from holding more than one active appointment or queue entry for the same service at the same time | Join Virtual Queue; Book Appointment *(constraint enforced within both)* | `Student`, `Queue Entry`, `Appointment`, `Registration Service` | Attempt to join a second queue for a service with an active `Queue Entry`; confirm rejection. Attempt to book a second appointment for a service with an active `Appointment`; confirm rejection |

## Resolved: BR-04 wording

BR-04 originally read: "A student cannot have two active appointments for the same service at the same time."
It is revised to: "A student cannot have two active appointments **or queue entries** for the same service at the same time,"
so it correctly covers the rule the CRC cards were already citing it for. FR-11 above states this explicitly as a
numbered requirement, closing the gap between the business rules and the functional requirements list.

## Not yet covered (no FR/NFR written yet)

The use-case diagram includes the following use cases with no corresponding numbered requirement:
Complete Student Service, View Appointment Details, Receive Notifications (Queue/Appointment Updates),
Manage Registration Services, Manage Appointment Availability, View Reports and Activity.

No quality/non-functional requirements have been supplied yet — these rows are still outstanding.
