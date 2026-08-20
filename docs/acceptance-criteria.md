# Acceptance Criteria

## UC-01: Join Virtual Queue

### AC-01: Successfully Join Queue

Given the student has selected an available registration service and is not already in its queue,

When the student chooses to join the virtual queue,

Then the system shall create a queue entry, assign a unique queue number, and display the student's current queue position.


### AC-02: Queue Is Full

Given the selected registration service's virtual queue has reached its maximum capacity,

When the student attempts to join the queue,

Then the system shall prevent the student from joining and inform the student that the queue is full.


### AC-03: Duplicate Queue Entry

Given the student already has an active queue entry for the selected registration service,

When the student attempts to join the same queue again,

Then the system shall prevent a duplicate queue entry and display the student's existing queue information.
