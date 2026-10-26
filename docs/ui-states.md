# UI States — Loading, Empty, Validation, Error, Unauthorised, Unavailable / Stale / Offline

**Project:** Student Registration Queue Management System — Team 17
**Lab:** CSI473 Laboratory 10
**Branch:** lab-10
**Applies to:** Task A (join queue), Task B (book appointment), Task C (staff call next and complete), plus sign-in.

**How to read the "Source" column**
- **UC** = a use case flow or exception from Phase 1 §4 (for example UC-06 exc. 4a).
- **API** = an error code in the Lab 8 contract (`docs/api-contracts/core-operation.yaml`).
- **Proposed** = needed by the interface but **not** yet defined in Phase 1 or the Lab 8 contract. Each one must be added to the use cases or the API before implementation, or dropped.

## Message rules (apply to every row)
1. Say **what happened** and **what the person can do next**, in plain words. No error codes, stack traces or blame.
2. Show errors **next to the thing they concern** and, for forms or conflicts, also in a summary at the top.
3. Never use colour alone: every state has a **symbol and words** (`!` for problems, a tick for success, an `x` for unavailable).
4. **Announce** changes: urgent problems use `role="alert"` and move focus to the message; passive updates use `role="status"` (polite).
5. Keep the person's work: do not clear fields or selections unless the state itself requires it (S4 clears the taken slot, and says so).
6. Do not promise what the system cannot guarantee. After a timeout during booking, tell the student to check **My appointments** before trying again, so they do not book twice.

## Sign-in (all roles) — UC-01, QS-03

| State | Source | What the user sees | Recovery | Accessibility behaviour |
|---|---|---|---|---|
| Loading | — | Button reads "Signing in..." and is inactive | — | `aria-busy`; button inactive prevents repeat submits |
| Validation | UC-01 | "Enter your student ID and password." beside the empty field(s) | Fill the field | Error linked to the field (`aria-describedby`); focus to the first invalid field |
| Error | UC-01 | "Student ID or password is incorrect." (does not say which) | Retry; "Forgot password" if the team provides it | `role="alert"` |
| Unauthorised: locked | API 423 `ACCOUNT_LOCKED`, QS-03 | "Too many failed attempts. Your account is locked until 10:57. Try again after that time." | Wait; contact the registry if urgent | `role="alert"`; time written out |
| Unauthorised: session ended | UC-01 exc. 1a | "Your session ended. Sign in again to continue." | Sign in; return to the same screen | `role="alert"`; the previous page is remembered |
| Unauthorised: wrong role | Proposed | "This page is for registration staff." / "This page is for students." | Link to the person's own home screen | `role="alert"`; no data from the other role is shown |

## S1 Services list — UC-01 (Tasks A, B)

| State | Source | What the user sees | Recovery | Accessibility behaviour |
|---|---|---|---|---|
| Loading | — | "Loading services..." with placeholder rows | — | `role="status"`; placeholders hidden from screen readers (`aria-hidden`) |
| Empty | UC-01 exc. 2a | "No registration services are open right now. Please check again later." | Come back later | `role="status"` |
| Empty filter result | UC-01 ext. 3a | "No services match 'xyz'. Clear the filter to see all services." with a **Clear filter** button | Clear filter | Announced politely; focus stays in the filter |
| Error loading | Proposed | "We couldn't load the services. Check your connection and try again." with **Try again** | Retry | `role="alert"`; focus to the message |
| Unavailable: queue closed | UC-02 exc. 1b, AF-03 | On the row: symbol + "Queue closed · appointments only". Join queue stays visible but inactive | Book an appointment instead | Row text is read with the button; inactive button remains focusable with `aria-disabled` so the reason is discoverable |
| Unavailable: queue full | UC-02 AF-01 | "This queue is full right now. You can book an appointment instead." | Book an appointment | `role="alert"` after the attempt |
| Unavailable: service deactivated | UC-16 ext. 3a | The service disappears from the list; if the student was on it: "This service is no longer available." | Choose another service | `role="alert"` |

## S2 Queue status — UC-02, UC-03, UC-04, UC-05, UC-11 (Task A)

| State | Source | What the user sees | Recovery | Accessibility behaviour |
|---|---|---|---|---|
| Loading: joining | UC-02 | Join queue reads "Joining..." and is inactive | — | Inactive button prevents a second tap (BR-04 also blocks a duplicate on the server) |
| Loading: first status | UC-04 | "Getting your place in the queue..." | — | `role="status"` |
| Success | UC-02 step 4 | Number, position, estimated wait, status "Waiting" with symbol, "Updated hh:mm:ss" | — | Focus moves to the queue number heading |
| Already in this queue (not an error) | UC-02 AF-02 | "You are already in this queue. Here is your place." Shows the existing number | None needed | `role="status"` |
| Validation | — | None: the student supplies no free-text input in this task | — | — |
| Error: join failed | Proposed | "We couldn't add you to the queue. Nothing was changed. Check your connection and try again. You can't be added twice." with **Try again** | Retry | `role="alert"` |
| Unavailable: estimate missing | UC-02 ext. 4a | "Estimated wait is not available yet." Number and position still shown | None | Read in order after position |
| Unavailable: position not available | UC-05 exc. 1a | "We can't show your position right now. Your place is kept. Try again shortly." with **Refresh now** | Refresh | `role="status"` |
| Stale / offline | Report ADR-02 (polling every 5–10 s) | Last values kept, banner: "Last updated 10:41:50. We can't refresh right now. Your position may have changed." with **Try again** | Retry; restores automatically | `role="status"` on first appearance; timestamp in text |
| Called | UC-14, UC-11 | Status "Called" with symbol and "Go to Counter 3"; grace-period text | Go to the counter | Polite live region announces; also delivered as a notification |
| Ended (Served / No-show / Cancelled) | UC-04 exc. 1a, state model | "This queue entry has ended (no-show). Join the queue again if you still need help." | Join again | `role="status"` |
| Unauthorised: someone else's entry | UC-10 business rule (own records only) | "You can only view your own queue place." | Back to services | `role="alert"` |

## S3 / S4 Choose appointment time — UC-06, UC-07 (Task B)

| State | Source | What the user sees | Recovery | Accessibility behaviour |
|---|---|---|---|---|
| Loading times | UC-07 | "Loading available times..." | — | `role="status"` |
| Empty: no slots | UC-06 exc. 3a, UC-07 exc. 1a | "No times are available on 27 Oct. Try another date, or join the queue instead." with **Choose another date** and **Join queue** (if the queue is open) | Other date or queue | `role="status"`; focus to the date field |
| Empty: no schedule | UC-07 exc. 1a | "Appointments are not available for this service yet." | Join the queue | `role="status"` |
| Validation: no time chosen | API 400 `VALIDATION_ERROR` (prevented on the client) | "Choose a time to continue." beside an inactive **Confirm booking** | Choose a time | Text linked to the button; button uses `aria-disabled` |
| Loading: booking | UC-06 step 4 | Confirm reads "Booking..." and is inactive | — | Prevents a double submit |
| Success | API 201, UC-06 step 5 | "Booking confirmed" tick, date, time, service; "We sent a confirmation in the app" (and by email if used) | **My appointments** | `role="status"`; focus to the confirmation heading |
| Success, notification failed | UC-11 exc. 3a, report ADR-04 | "Your booking is confirmed. We couldn't send a notification, so your appointment is saved in My appointments." | Open My appointments | `role="status"`; the booking is not undone |
| Error: slot just taken | API 409 `SLOT_ALREADY_BOOKED`, UC-06 exc. 4a, BR-02 | **S4:** "That time was just booked. 10:30 – 11:00 was taken by someone else a moment ago. Please choose another time." List refreshed, selection cleared, date kept | Choose another time | `role="alert"`; focus moves to the alert |
| Error: slot not open | API 409 `SLOT_UNAVAILABLE`, BR-01 | "That time is no longer open for booking. Please choose another time." List refreshed | Choose another time | `role="alert"` |
| Error: slot missing | API 404 `SLOT_NOT_FOUND` | "That time no longer exists. The list has been updated." | Choose another time | `role="alert"` |
| Error: already has appointment | API 409 `DUPLICATE_ACTIVE_APPOINTMENT`, BR-04 | "You already have an appointment for this service." with **View my appointment** | UC-10 | `role="alert"` |
| Error: request failed or timed out | Proposed | "We couldn't confirm your booking. Check **My appointments** before trying again, so you don't book twice." | Check My appointments, then retry | `role="alert"`; link in the message |
| Unauthorised: locked or session ended | API 423 `ACCOUNT_LOCKED`; UC-01 exc. 1a | See Sign-in rows | See Sign-in | See Sign-in |
| Stale / offline | UC-07 (availability must be current), QS-07 | "Times last updated 10:41. Refresh to see current times." with **Refresh times**; offline: "You're offline. Reconnect to see current times." and **Confirm booking** inactive | Refresh or reconnect | `role="status"`; inactive button explains why |

## My appointments and cancellation — UC-08, UC-09, UC-10

| State | Source | What the user sees | Recovery | Accessibility behaviour |
|---|---|---|---|---|
| Loading | — | "Loading your appointments..." | — | `role="status"` |
| Empty | UC-10 exc. 2a | "You have no appointments yet." with **Book an appointment** | Book | `role="status"` |
| Confirm cancel | UC-08 | Dialog: "Cancel your 10:30 appointment on 27 Oct?" Buttons **Cancel appointment** and **Keep appointment** (default focus on Keep) | Choose | Modal dialog; focus trapped, Esc keeps the appointment |
| Cancelled | UC-08 step 4 | "Appointment cancelled. The time is available to others again." | — | `role="status"` |
| Error: past cut-off | UC-08 exc. 2a, UC-09 | "This appointment can no longer be cancelled online. The cut-off was 08:00 today." | Contact the service | `role="alert"`; cut-off written out |
| Error: already cancelled / completed | UC-08 exc. 1a, BR-05, BR-08 | "This appointment is already cancelled." / "...already completed." | None | `role="status"` |

## Registration staff dashboard — UC-12, UC-13, UC-14, UC-15 (Task C)

| State | Source | What the user sees | Recovery | Accessibility behaviour |
|---|---|---|---|---|
| Loading | — | "Loading queues..." | — | `role="status"` |
| Empty | UC-12 exc. 2a, UC-14 exc. 1a | "No students are waiting or due for an appointment at this counter." Call next inactive: "nothing to call" | Wait; checks again every 10 s | `role="status"`; not styled as an error |
| Loading: calling | UC-14 | Call next reads "Calling..." | — | Inactive while running |
| Success: called | UC-14 step 5 | Panel with ID, queue number or appointment time, source, counter, grace period | Complete or mark no-show | Focus moves to the called-student panel heading |
| Validation: nothing to complete | UC-15 exc. 1a | Complete and No-show inactive with "No student is called at this counter." | Call next | `aria-disabled`, reason in text |
| Confirm no-show | UC-15 ext. 1a, BR-08 | "Mark queue 17 as no-show? This cannot be undone." **Mark no-show** / **Keep waiting** | Choose | Modal dialog; default focus on Keep |
| Grace period ended | UC-14 exc. 3a, state model | "Grace period ended. Queue 17 was marked as no-show." Counter freed | Call next | `role="alert"` |
| Error: call failed | Proposed | "Couldn't call the next student. Nothing was changed. Try again." If a student already shows as called, continue with them | Retry | `role="alert"` |
| Error: student called elsewhere | Proposed (not in Phase 1) | "That student was just called at another counter. Showing the next student." | Continue | `role="alert"` |
| Unauthorised: service not at this counter | UC-12 business rule, BR-07 | Not shown in the list. If reached directly: "You don't have access to this service. Ask an administrator to assign it to your counter." | Contact administrator | `role="alert"` |
| Stale / offline | Report ADR-02, QS-02 | Banner "Connection lost. Showing data from 10:41:50. Actions are paused until the connection returns." with **Retry now**; data labelled "STALE" | Retry; resumes automatically | `role="alert"` once; Call next, Complete and No-show inactive |

## Decisions and consequences
- **No offline caching and no queued actions.** The offline behaviour is to show the last data, label it stale and **pause actions**. Queued offline calls could call the same student twice or reorder the queue (BR-06, report ADR-03). Consequence: a staff member or student who loses connection simply cannot act until it returns. This follows the lab's warning not to add offline features the project does not need.
- **Messages after timeouts point to My appointments** because a booking may have been saved even though the reply was lost.
- **Items marked Proposed** (queue-join failure, "called elsewhere", "wrong role", request timeout) need a decision by the team: add to the use cases or API contract, or remove from the interface.

## Traceability
Use cases: UC-01, UC-02, UC-04–UC-15. API: Lab 8 `bookAppointment` codes. Quality scenarios: QS-01, QS-02, QS-03, QS-05, QS-07. Business rules: BR-01, BR-02, BR-04, BR-05, BR-06, BR-07, BR-08. Related: `prototype/wireframes-student.svg`, `prototype/wireframes-staff.svg`, `docs/accessibility-checklist.md`.
