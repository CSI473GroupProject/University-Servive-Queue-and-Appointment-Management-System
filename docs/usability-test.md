# Usability Test Record

**Project:** Student Registration Queue Management System — Group 17
**Lab:** CSI473 Laboratory 10
**Branch:** lab-10
**Prototype tested:** `prototype/wireframes-student.svg` and `prototype/wireframes-staff.svg`
**Tasks:** A (join queue), B (book appointment), C (staff call next and complete) — see `docs/user-profile-and-tasks.md`

## 1. Purpose and limits
A small, formative, task-based check of whether people can complete the named tasks on the
wireframes, to find problems early and change the design.

**Limit:** three people can reveal problems but cannot establish a rate. QS-05 asks that **at least
90% of first-time users** complete queue joining **within 3 minutes**; that needs a larger sample on
the built interface (Phase 2, Section 13).

## 2. Participants

| ID | Who (role they play) | Is a team member? | Familiar with the system? | Device or input used | Tasks |
|---|---|---|---|---|---|
| 202204313 | Student | **No** | **No** |phone | A, B |
| 202204319 | Student | **No** | **No** |personal computer | A, C (as staff) |
| 202105155 | Student | **No** | **No** |desktop| A, B,C |

## 3. Method (14-minute slot: about 4–5 minutes per person)
1. **Set-up.** Show the wireframes one frame at a time (printed, or on a screen). One team member is the **facilitator** and also plays the system (shows the next frame when the participant "presses" something). One team member is the **observer** and writes everything down. Do not ask the participant to look at the notes column.

2. **Give the scenario only** (no hints about which button to press):
   - **Task A:** "It is the first week of registration. You have a question about your course registration and there is a long line. You want to get in line without standing there. Show me what you would do." *Done when:* they say they have a place in the queue and what happens next.
   - **Task B:** "You prefer a fixed time. Book a time for course registration tomorrow morning." *Done when:* they say the booking is confirmed. (**Facilitator: when the participant confirms 10:30, show frame S4** to test the conflict, then see whether they recover.)
   - **Task C (staff role):** "You are at Counter 3 and have just finished with a student. Call the next student for Course registration, then record that you have finished with them." *Done when:* they say the student is recorded as served.
3. **Stop rules.** Task A: stop at 3 minutes (the QS-05 limit). Tasks B and C: stop at 5 minutes. Offer help only after the participant has clearly given up, and record that it was **assisted**.
4. **After each task** ask: "How easy or difficult was that, from 1 (very difficult) to 7 (very easy)?" and "What did you expect to happen after that last step?"
5. **Keyboard or zoom check (if time):** ask one participant to repeat Task A using only the keyboard on a laptop, or with the browser zoomed to 200%. Record whether this was done; if not, it stays untested.

## 4. Observations 

One row per observation. **Severity:** 1 = small annoyance; 2 = hesitation or detour but completed; 3 = could not continue without help.

| Participant | Task | Frame | What happened (exact action, words said, where they hesitated) | Severity |
|---|---|---|---|---|
|202204313 |A |queue status |The user correctly pressed on the join queue button and was succesfully allocated position 1 in the que, "wait this aint right" hesitated between book apointment and join queue |2 |
|202204313 |B| Choose appointment time|the user choose an already active time slot from our pseudo entries,"ooops" no hesitations |1 |
|202204319 |A|queue status|the user clicked on the refresh button and the screen refreshed to give him his new status queued, "done" no hesitation |1 |
|202204319 |C|staff call next and complete |the user lost connection and waited for service recovery indefinitely, "No connection", no hesitation  |3 |
|202105155 |A|queue status |The user wanted to leave the queue so he clicked on the leave button and successfully left,"left", no hesitation |1 |
|202105155 |B| Choose appointment time |the user wanted to select an apointment time suitable for his timetable, he struggled with the date format, "what is the format for date?" on the date format to use |  3 |
|202105155|C|staff call next and complete  |The user succesfully alternated between caling for both appointments and registration queries addressing them successfull without starving the other, "Easy peezy"  |1  |   
## 5. Results per task 

| Participant | Task | Result (Success / Assisted / Failed) | Time (mm:ss) | Wrong taps or detours | Help asked? | Ease rating (1(vey hard)–7(very easy)) |
|---|---|---|---|---|---|---|
| 202204313  | A |success |1:08 |1 |No |6 |
| 202204313 | B | assisted|1:30 |0 |yes |5 |
| 202204319 | A | success|0:58 |0 |no |7 |
| 202204319 | C |failed |4:03 |3 | yes|3 |
| 202105155 | A |success |1:20|0|no |6 |
| 202105155 | B |assisted |2:00|0|yes | 6|
|202105155|C|success |1:58  | 0 | yes|7|

**Success criteria used** (from `docs/user-profile-and-tasks.md`): Task A — reaches the confirmation, unaided, within 3 minutes. Task B — books a slot and recovers from the conflict without re-entering the date. Task C — calls next and completes, and does not confuse Complete with No-show.

## 6. Revision log 



| # | Observed difficulty (participant, task, frame, what happened) | Design change made | Artefact and commit changed | How it will be verified after implementation |
|---|---|---|---|---|
| 1 |Participant 202204313, Task A, Queue Status: The participant hesitated between Join Queue and Book Appointment, saying, “wait this ain't right.” Severity: 2. |Improve the distinction between joining the queue and booking an appointment by adding a short explanatory label under each button, such as “Join the queue now” and “Book a time slot.” |Queue interface wireframe in the prototype and docs/ui-states.md. |Retest Task A with new participants and check whether they choose the correct action without assistance within 3 minutes. |
| 2 |Participant 2022105155, Task B, Choose Appointment Time: The participant struggled with the date format and asked, “what is the format for date?” Severity: 3. |Provide a date picker and display the expected date format as a hint, for example, DD/MM/YYYY |Appointment booking wireframe in the prototype and docs/ui-states.md. |Retest Task B and check whether participants can select a valid appointment date and time without help.|


| # | Risk to watch for | Frame | If observed, consider |
|---|---|---|---|
| H1 | The participant confuses **Join queue** and **Book appointment**, or does not see that both are on the row | S1 | Add a one-line explanation under each (for example "Join now, wait in line" / "Pick a time"), or fall back to a separate options step (UC-01 step 4 as written) |
| H2 | After joining, the participant cannot say what happens next, or misreads "5th in line" / "about 20 min" as exact | S2 | Make the "we will notify you" text more prominent and place it directly under the position; wording such as "about" or "around" more visible |
| H3 | After the booking conflict, the participant does not notice the refreshed list, picks the removed time again, or thinks the whole booking failed | S4 | Keep the previously chosen time visible as "Taken" with a strike-through, or add "The list has been updated" under the alert |
| H4 | Staff press **Mark no-show** instead of **Complete**, or cannot tell why **Call next** is inactive | T1 after | Increase separation or change the order of the two buttons; move the reason text above the inactive button |
| H5 | A student does not understand what **Leave queue** does or fears losing their place by pressing Refresh | S2 | Rename ("Leave queue and lose my place"), and state on Refresh that it keeps their place |
| H6 | The participant does not find the cancellation rule before confirming | S3 | Move the cancellation line above **Confirm booking**, or into the summary box |

## 8. Exit record (to be completed after the session)

| Item | Entry |
|---|---|
| **Most important usability problem observed** (with evidence: participant, task, frame) |Participant 202204319, Task C, Frame: staff call next and complete. The user lost connection and waited for service recovery indefinitely (Severity 3). Secondary issue: Participant 202105155 (Task B) struggled with the date format, asking "what is the format for date?". |
| **Interface change made** (file and commit) | Modified ui-states.md and the prototype to add a clear error state/timeout message for lost connections, and added placeholder text "DD/MM/YYYY" to the date picker. |
| **How it will be verified after implementation** |1. Re-run tasks: Have three new first-time users attempt Task B and C on the built screen, requiring them to finish unaided within the task limit (specifically ensuring they don't get stuck on the date format).2. End-to-end test: Add an automated test that asserts the new "connection lost" alert is present and reachable, and that the date format placeholder is rendered. |

## Traceability
QS-05 (first-time usability), QS-01 (position shown quickly), QS-03 (lockout message); UC-01, UC-02, UC-04, UC-06, UC-14, UC-15; FR-02, FR-04, FR-06, FR-10. Related artefacts: `docs/user-profile-and-tasks.md`, `docs/ui-states.md`, `docs/accessibility-checklist.md`, `prototype/wireframes-student.svg`, `prototype/wireframes-staff.svg`, `models/navigation.mmd`.
