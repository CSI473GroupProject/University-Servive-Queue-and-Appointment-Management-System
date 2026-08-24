# Domain Model: Multiplicities and Assumptions

## 1. Multiplicities in the Domain Model

Multiplicity shows **how many instances of one class can be associated with one instance of another class**.

### A. RegistrationStaff — manages — VirtualQueue

**Multiplicity:**

* RegistrationStaff → VirtualQueue: **0..***
* VirtualQueue → RegistrationStaff: **0..***

**Explanation:**
A registration staff member can manage **zero or many virtual queues**. A virtual queue can also be managed by **zero or many registration staff members**, for example, different staff members may manage the queue at different times.

### B. VirtualQueue — contains — QueueEntry

**Multiplicity:**

* VirtualQueue → QueueEntry: **0..***
* QueueEntry → VirtualQueue: **1**

**Explanation:**
A virtual queue can contain **zero or many queue entries**. However, each queue entry must belong to **exactly one virtual queue**.

**Example:**
A newly created queue may have no students yet, while a busy queue may contain many students.

### C. Student — creates — QueueEntry

**Multiplicity:**

* Student → QueueEntry: **0..***
* QueueEntry → Student: **1**

**Explanation:**
A student can create **zero or many queue entries**, depending on how many services they use. However, each queue entry belongs to **exactly one student**.

### D. Student — books — Appointment

**Multiplicity:**

* Student → Appointment: **0..***
* Appointment → Student: **1**

**Explanation:**
A student can make **zero or many appointments**. Each appointment, however, is associated with **exactly one student**.

### E. RegistrationService — has — VirtualQueue

**Multiplicity:**

* RegistrationService → VirtualQueue: **0..***
* VirtualQueue → RegistrationService: **1**

**Explanation:**
A registration service can have **zero or many virtual queues**. Each virtual queue belongs to **exactly one registration service**.

**Example:**
The Student Registration service could have separate queues for different registration activities.

### F. ServiceAdministrator — configures — RegistrationService

**Multiplicity:**

* ServiceAdministrator → RegistrationService: **0..***
* RegistrationService → ServiceAdministrator: **1**

**Explanation:**
A service administrator can configure **zero or many registration services**. Each registration service is configured by **one service administrator**.

### G. RegistrationService — reserves — AppointmentSlot

**Multiplicity:**

* RegistrationService → AppointmentSlot: **0..***
* AppointmentSlot → RegistrationService: **1**

**Explanation:**
A registration service can have **zero or many appointment slots**. Each appointment slot belongs to **exactly one registration service**.

**Example:**
A service could provide appointment slots at 09:00, 10:00, 11:00, etc.

### H. AppointmentSlot — reserves — Appointment

**Multiplicity:**

* AppointmentSlot → Appointment: **0..1**
* Appointment → AppointmentSlot: **1**

**Explanation:**
An appointment slot can have **zero or one appointment** because a slot can either be available or already booked. Each appointment must be associated with **exactly one appointment slot**.

**Important:** `0..1` means **zero or one**, not zero or many.

## 2. Assumptions

1. **Each student can have only one active queue entry for a particular registration service at a time.**

2. **Each queue entry belongs to exactly one student and exactly one virtual queue.**

3. **Queue numbers are generated automatically according to the order in which students enter the queue.**

4. **Queue positions are updated automatically when students are served, removed, or when the queue changes.**

5. **A student can book multiple appointments, but each individual appointment belongs to only one student.**

6. **An appointment slot can be booked by only one appointment at a time.**

7. **Students can cancel appointments, and cancelled slots can become available for another student.**

8. **Queue entries and appointments have statuses that indicate their current state, such as Waiting, Served, Cancelled, or Confirmed.**

9. **A registration service can operate multiple virtual queues, depending on the type and volume of services provided.**

10. **Only authorised service administrators can configure registration services and their service rules.**

11. **Registration staff are responsible for managing queues and processing students who are waiting for services.**

12. **Appointment slots have a specific date, start time, and end time and are created for a particular registration service.**

13. **Students must satisfy applicable eligibility rules before they can access certain registration services or appointments.**

14. **Cancellation policies determine whether an appointment can be cancelled and under what conditions.**

15. **Service rules define the operational rules and order in which a registration service handles students.**

16. **Notifications are optional and may be used to inform students about events such as appointment confirmations, cancellations, or queue updates.**
