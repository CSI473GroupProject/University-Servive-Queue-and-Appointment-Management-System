# UC-01: Join Virtual Queue

## Primary Actor
Student

## Goal
Allow a student to join the virtual queue for a selected registration service.

## Preconditions
1. The student can access the system.
2. The selected registration service is available.
3. The virtual queue for the service is open.
4. The student is not already in the same queue.

## Trigger
The student selects a registration service and requests to join its virtual queue.

## Main Success Flow
1. The student selects a registration service.
2. The system displays the current queue information.
3. The student selects "Join Virtual Queue".
4. The system checks the student's eligibility.
5. The system creates a queue entry for the student.
6. The system assigns a unique queue number.
7. The system calculates the student's position in the queue.
8. The system displays the queue number and current position.
9. The system stores the student's queue status.

## Alternative Flows

### AF-01: Queue Is Full
1. The system determines that the queue has reached its maximum capacity.
2. The system prevents the student from joining.
3. The system informs the student that the queue is full.
4. No queue entry is created.

### AF-02: Student Already Has an Active Queue Entry
1. The system detects that the student already has an active queue entry.
2. The system prevents a duplicate queue entry.
3. The system displays the student's existing queue information.

### AF-03: Registration Service Is Unavailable
1. The system determines that the selected service is unavailable.
2. The system informs the student.
3. The system does not create a queue entry.
4. The student may select another available service.

## Postconditions

### Success
- A queue entry exists for the student.
- A unique queue number has been assigned.
- The student's queue position is recorded.
- The student's queue status can be monitored.

### Failure
- No invalid or duplicate queue entry is created.
- The student is informed of the reason for failure.

## Business Rule
A student must not have more than one active queue entry for the same registration service.
