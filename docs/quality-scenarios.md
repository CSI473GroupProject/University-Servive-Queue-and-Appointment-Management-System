# Quality Scenarios

## QS-01: Performance

Stimulus: A student requests their queue position.

Context: The system is operating under normal load.

Expected Response: The system displays the student's current queue position.

Measurable Response Level: The response shall be displayed within 2 seconds.


## QS-02: Availability

Stimulus: A student attempts to access the queue system during registration hours.

Context: The university registration period is active.

Expected Response: The system allows the student to access queue services.

Measurable Response Level: The system shall have at least 99% availability during registration periods.


## QS-03: Security

Stimulus: An unauthorised user repeatedly enters incorrect login credentials.

Context: The user attempts to access a student account.

Expected Response: The system prevents further login attempts temporarily.

Measurable Response Level: The account shall be locked after 5 consecutive failed attempts.


## QS-04: Reliability

Stimulus: A student successfully joins a virtual queue.

Context: The system is operating normally.

Expected Response: The system saves the student's queue entry and position.

Measurable Response Level: 99.9% of successful queue-joining transactions shall be recorded correctly.


## QS-05: Usability

Stimulus: A student wants to join a virtual queue for a registration service.

Context: The student is using the system for the first time.

Expected Response: The student is able to complete the queue-joining process without assistance.

Measurable Response Level: At least 90% of first-time users shall complete the process within 3 minutes.
