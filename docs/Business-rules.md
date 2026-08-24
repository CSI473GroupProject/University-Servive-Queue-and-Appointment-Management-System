# Business Rules

## BR-01: Appointment Availability
A student can only book an appointment with a service provider when
the requested time slot is available.

## BR-02: No Double Booking
A service provider cannot have more than one appointment scheduled
for the same time slot.

## BR-03: Queue Availability
A student can only join a queue for a service that is currently
available.

## BR-04: Unique Active Appointment
A student cannot have two active appointments for the same service
at the same time.

## BR-05: Appointment Cancellation
A student can only cancel an appointment that has not yet been
completed or served.

## BR-06: Queue Position
Each student in a queue must have a unique queue position.
Queue positions follow the order in which students joined the queue.

## BR-07: Service Provider Authorization
Only an authorized service provider can manage appointments for
their assigned service.

## BR-08: Completed Appointment
A completed appointment cannot be cancelled, modified, or returned
to the active queue.
