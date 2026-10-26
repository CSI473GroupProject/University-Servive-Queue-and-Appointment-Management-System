# Usability Test Record

**Project:** Student Registration Queue Management System — Group 17
**Lab:** CSI473 Laboratory 10
**Branch:** lab-10
**Prototype tested:** `prototype/wireframes-student.svg` and `prototype/wireframes-staff.svg`
**Tasks:** A (join queue), B (book appointment), C (staff call next and complete) — see `docs/user-profile-and-tasks.md`

> ## STATUS: PROTOCOL READY — RESULTS NOT YET RECORDED
> Sections 1–3 and 7 are prepared. **Sections 4–6 and 8 are blank on purpose and must be filled in by
> the team from what really happened in the session.** The lab marks "at least one design change
> follows from observed user difficulty", so invented observations would be both dishonest and
> self-defeating. Delete this banner once the results are in.

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

## 4. Observations (to be filled in during the session)

One row per observation. **Severity:** 1 = small annoyance; 2 = hesitation or detour but completed; 3 = could not continue without help.

| Participant | Task | Frame | What happened (exact action, words said, where they hesitated) | Severity |
|---|---|---|---|---|
| | | | | |
| | | | | |
| | | | | |
| | | | | |
| | | | | |
| | | | | |

## 5. Results per task (to be filled in)

| Participant | Task | Result (Success / Assisted / Failed) | Time (mm:ss) | Wrong taps or detours | Help asked? | Ease rating (1–7) |
|---|---|---|---|---|---|---|
| P1 | A | | | | | |
| P1 | B | | | | | |
| P2 | A | | | | | |
| P2 | C | | | | | |
| P3 | A | | | | | |
| P3 | B | | | | | |

**Success criteria used** (from `docs/user-profile-and-tasks.md`): Task A — reaches the confirmation, unaided, within 3 minutes, and can say what happens next. Task B — books a slot and recovers from the conflict without re-entering the date. Task C — calls next and completes, and does not confuse Complete with No-show.

**Summary (write after the session):** how many succeeded unaided on each task; the median time for Task A; the most serious problem (highest severity, most participants).

## 6. Revision log (to be filled in)

At least one row must come from a difficulty that was **actually observed**. Name the evidence.

| # | Observed difficulty (participant, task, frame, what happened) | Design change made | Artefact and commit changed | How it will be verified after implementation |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |

**To make the revision visible for marking:** commit the **tested** wireframes first, then change the wireframes (and `docs/ui-states.md` or the navigation model if affected) in a **separate commit** whose message names the problem, for example "Lab 10: revise S2 after P2 did not see Leave queue". The two commits together show the revision.

## 7. Where to look — risks in the current design (hypotheses, not results)

These are the places where the design is most likely to cause trouble, drawn from the wireframes. They tell the observer what to watch for. **They are not findings.** If one is observed, the right-hand column suggests a change. If it is not observed, do not claim it.

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
| **Most important usability problem observed** (with evidence: participant, task, frame) | |
| **Interface change made** (file and commit) | |
| **How it will be verified after implementation** | |

**Suggested ways to verify** (choose the ones that fit the change): re-run the same task with **three new first-time users** on the built screen and require every one to finish unaided within the task limit; add an **end-to-end test** that asserts the changed element is present and reachable by keyboard; check the changed element against `docs/accessibility-checklist.md` rows 1–6; for Task A, record the completion time of each tester to feed the QS-05 measurement in Section 13 of the report.

## 9. Academic integrity note
The test protocol and the wireframes were prepared with AI assistance. The observations, ratings
and decisions recorded in Sections 2 and 4–8 must come from the team's own session. Disclose the
assistance in Section 16 of the project report, and keep the original notes (photos of paper
notes, or the observer's file) as source evidence.

## Traceability
QS-05 (first-time usability), QS-01 (position shown quickly), QS-03 (lockout message); UC-01, UC-02, UC-04, UC-06, UC-14, UC-15; FR-02, FR-04, FR-06, FR-10. Related artefacts: `docs/user-profile-and-tasks.md`, `docs/ui-states.md`, `docs/accessibility-checklist.md`, `prototype/wireframes-student.svg`, `prototype/wireframes-staff.svg`, `models/navigation.mmd`.
