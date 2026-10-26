# Accessibility Checklist

**Project:** Student Registration Queue Management System — Team 17
**Lab:** CSI473 Laboratory 10
**Branch:** lab-10
**Basis:** WCAG 2.2 Level AA success criteria (referenced by number). The lab's required areas are: keyboard access, visible focus, persistent labels, non-colour cues, clear errors, and readable responsive layout.

## What this checklist is and is not
This is a **design-stage** checklist. "Designed" means the wireframes and `docs/ui-states.md`
specify the behaviour. **Nothing here has been verified on a running interface**, because none
exists yet; each row says how it will be verified after implementation. The one exception is the
colour-contrast figures, which were **calculated** from the exact colours used in the wireframes.

Status key: **Designed** = specified in the wireframes or states table. **Verify** = must be tested on the implemented screens. **Decision needed** = the team must choose.

## The six required areas

| # | Area | WCAG | How the design meets it | Shown in | Verify after implementation | Status |
|---|---|---|---|---|---|---|
| 1 | **Keyboard access** | 2.1.1 | Every action is a real button, link, input or radio group, reachable by Tab and operated with Enter/Space. Radio groups use arrow keys. Dialogs trap focus and close with Esc. No action depends on hover or touch gestures. | All frames; red focus numbers | Complete Tasks A, B and C using the keyboard only, no mouse. | Designed · Verify |
| 2 | **Focus order and visible focus** | 2.4.3, 2.4.7, 2.4.11 | Focus order follows reading order (numbers on each frame). A strong focus ring (at least 2 px, high contrast, not removed) is required on every control. Sticky headers and banners must not hide the focused control. | Red numbers on S1–S4, T1, T2 | Tab through every screen; confirm the ring is always visible, including at 200% zoom and with the offline banner shown. | Designed · Verify |
| 3 | **Persistent labels** | 1.3.1, 3.3.2, 2.5.3 | Every field has a visible label that stays on screen ("Filter services", "Date", "Service note (optional)", "Counter"). Placeholder text is only an example, never the label. Buttons that repeat across rows carry the service name in their accessible name and still contain their visible words ("Join queue for Course registration"). | S1, S3, S4, T1 | Check each input's accessible name with the browser accessibility tree; try voice-control by saying the visible button text. | Designed · Verify |
| 4 | **Non-colour cues** | 1.4.1, 1.4.11 | Status always uses symbol + words (open, closed, waiting, called, stale). Selection uses a filled radio, a thick border and a text summary. Errors use `!` symbol, bold heading and a thick border. Disabled actions show the reason in text. | S1, S2, S3, S4, T1, T2 | View every screen in greyscale or with a colour-blindness simulator; each state must stay distinguishable. | Designed · Verify |
| 5 | **Clear errors** | 3.3.1, 3.3.3, 3.3.4, 4.1.3 | Each message names the problem and the next step (`docs/ui-states.md`). Urgent problems use `role="alert"` and move focus; passive updates use `role="status"`. Destructive or terminal actions (Cancel appointment, Mark no-show) ask for confirmation, with the safe choice as the default. | S4, T2, ui-states.md | Trigger each error with a screen reader running and confirm it is announced once, with the right wording. | Designed · Verify |
| 6 | **Readable, responsive layout** | 1.4.4, 1.4.10, 1.4.12, 1.3.4 | Single-column mobile-first layout that reflows without sideways scrolling at 320 CSS px; text can be enlarged to 200% without loss; layout works in portrait and landscape; spacing does not break if the user overrides text spacing. | Student frames (phone width) | Test at 320 px width, 200% zoom and with a text-spacing bookmarklet; check both orientations. | Designed · Verify |

## Additional checks relevant to this project

| # | Check | WCAG | Design response | Verify after implementation | Status |
|---|---|---|---|---|---|
| 7 | Text contrast at least 4.5:1 | 1.4.3 | Calculated from the wireframe colours: white on primary button 8.87:1; secondary button text 8.87:1; error text on alert fill 8.92:1 (10.20:1 on white); secondary text #444 on white 9.74:1; status text on its chip 6.03:1; inactive-button text #666 on #eee 4.95:1; note grey #555 on white 7.46:1. All pass. | Re-measure with a contrast tool on the built screens (colours may change). | **Calculated for wireframes** · Verify |
| 8 | Component outlines at least 3:1 | 1.4.11 | Control borders (#333, #555) are 7.46:1 or higher. **Finding:** the dashed outline of an inactive button (#999) is 2.85:1. Inactive controls are exempt, but ours stay focusable and carry the reason, so the outline should be **#767676 or darker (about 4.5:1)** in the build. | Measure on the built screens. | Design change recorded · Verify |
| 9 | Touch targets large enough | 2.5.8 | Buttons and radio rows are 40–46 px high (minimum required is 24 px; about 44 px is good practice on phones). | Measure on a real phone. | Designed · Verify |
| 10 | Status messages announced | 4.1.3 | Queue position changes announce politely and **only when meaningful** (for example "You are now 4th", "You have been called"), not on every poll (UC-11). | Screen-reader test on S2 with a simulated position change. | Designed · Verify |
| 11 | Auto-updating content | 2.2.2 | Queue position refreshes every 5–10 s (report ADR-02). Updates never move focus or reflow the page, and **Refresh now** gives manual control. The updates are essential to the task, so the pause/stop exception may apply. | Confirm the page stays stable while updating; **the team should decide** whether a pause control is also needed. | Decision needed |
| 12 | Session timeout | 2.2.1 | Session expiry (UC-01 exc. 1a) shows a clear message and returns the person to the same screen after sign-in. | Check that a warning or clear recovery is in place before data is lost. | Designed · Verify |
| 13 | Redundant entry | 3.3.7 | After a booking conflict the date is kept and the student is not asked to re-enter anything (S4). | Test the S4 flow end to end. | Designed · Verify |
| 14 | Accessible sign-in | 3.3.8 | Sign-in must allow password managers and paste, and must not require remembering or transcribing a code from memory. Account lockout (QS-03) is unaffected. | Test with a password manager. | Decision needed |
| 15 | Page titles, headings and language | 2.4.2, 2.4.6, 3.1.1 | Each screen has a unique title ("Queue status – Course registration") and one main heading; the page language is set. | Check the title bar and heading outline of every screen. | Designed · Verify |
| 16 | Bypass blocks and consistent navigation | 2.4.1, 3.2.3 | A "Skip to main content" link is the first focus stop; the same navigation (Services, My appointments, Alerts) appears in the same place on every student screen. | Tab from the page top; compare screens. | Designed · Verify |

## What was actually checked in Lab 10
- The wireframes were reviewed against this checklist by the design author (this document records the outcome as "Designed").
- Contrast ratios in row 7–8 were calculated from the colours in `prototype/wireframes-*.svg`.
- The usability sessions in `evidence/usability-test.md` record whether any participant used a keyboard only or a zoomed display. If none did, that remains untested and is carried to implementation.

## Traceability
Quality scenario QS-05 (first-time student completes queue joining unaided within 3 minutes) is the usability requirement this serves; accessibility supports it because a first-time user may use a keyboard, a screen reader or a zoomed phone. Related requirements: QS-03 (lockout message), QS-01 (position display), UC-01, UC-02, UC-04, UC-06, UC-14, UC-15. Related artefacts: `docs/ui-states.md`, `prototype/wireframes-student.svg`, `prototype/wireframes-staff.svg`, `models/navigation.mmd`.
