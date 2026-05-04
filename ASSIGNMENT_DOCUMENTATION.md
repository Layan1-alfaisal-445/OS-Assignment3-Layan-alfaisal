# Assignment 3 - Complete Documentation

**Student Name**: layan faisal alfaisal  
**Student ID**: 445052190
**Date Submitted**: 5-5-2026

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [4-5-2026 , 4pm]
**What I implemented**: Defined synchronization objects in SharedResources, including ReentrantLock for counters and a Semaphore for CPU access

**Challenges encountered**: Protecting shared variables like completedProcessCount from concurrent access errors

**How I solved it**: Applied lock() before critical sections and ensured thread safety for shared resources

**Testing approach**: Verified that the synchronization objects were correctly initialized in the SharedResources class

**Time spent**: 1 hours

---

### Entry 2 - [4-5-2026 , 5pm]
**What I implemented**: Integrated try-finally blocks within the increment methods incrementCompletedProcess, addWaitingTime to manage locks

**Challenges encountered**: Risk of leaving a lock "held" if an error occurred, which would freeze the simulation

**How I solved it**: Placed the unlock() method strictly inside the finally block to guarantee the lock is always released

**Testing approach**: Ran the code to ensure that no "Deadlock" occurred during multiple process transitions

**Time spent**: 2 hours

---

### Entry 3 - [4-5-2026 , 7pm]
**What I implemented**: Implemented error handling using try-catch-finally for the Semaphore acquisition in the run() method

**Challenges encountered**: Handling InterruptedException when a process is waiting for its turn on the CPU

**How I solved it**: Added catch (InterruptedException e) to log errors and used finally { cpuSemaphore.release(); } to ensure the CPU slot is freed

**Testing approach**:Used my Student ID (445052190) as the seed to generate a stable simulation and verified the final execution logs

**Time spent**: 1 hours

---

### Entry 4 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

### Entry 5 - [Date, Time]
**What I implemented**: 

**Challenges encountered**: 

**How I solved it**: 

**Testing approach**: 

**Time spent**: 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

The original code had race conditions in shared counters like contextSwitchCount++ and the executionLog ArrayList. These occurred because multiple threads tried to read and update these resources simultaneously without protection. This leads to Lost Updates in counters and data corruption or crashes in the non-thread-safe list. I solved this by using ReentrantLock to ensure only one thread accesses these critical sections at a time

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

I used ReentrantLock for Mutual Exclusion to protect shared data like counters from race conditions. I implemented cpuSemaphore as a Binary Semaphore (0 or 1) to ensure only one process executes on the CPU at a time. Locks were chosen to safeguard specific variables, while the semaphore regulates overall process flow. This combination ensures both data integrity and orderly scheduling in the simulation


### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

Deadlock is when threads wait indefinitely for each other to release resources. I prevented it by using try-finally blocks to ensure every lock and semaphore is always released. I also avoided nested locks to maintain a simple lock ordering and prevent circular waiting. This ensures the simulation remains stable and never freezes during execution
---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

I used separate locks for each counter (fine-grained) because the counters are independent resources. This choice improves concurrency by allowing different threads to update metrics like contextSwitchCount and totalWaitingTime at the same time. While one lock (coarse-grained) is simpler, it creates a performance bottleneck. Separate locks provide a more efficient and scalable scheduler design

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount, completedProcessCount and totalWaitingTime

**Why they need protection**: These variables are shared among multiple threads, and their increment operations (like ++) are not atomic. Without protection, concurrent access leads to Race Conditions, resulting in lost updates and incorrect final statistics

**Synchronization mechanism used**: ReentrantLock (specifically contextSwitchLock, completedProcessLock, and waitingTimeLock)

**Code snippet**:
completedProcessLock.lock();
try {
    completedProcessCount++;
} finally {
    completedProcessLock.unlock();
}

**Justification**: Using ReentrantLock ensures Mutual Exclusion, allowing only one thread to update the counter at a time. The try-finally block guarantees that the lock is always released, preventing deadlocks and ensuring data integrity

---

### Critical Section #2: Execution Log

**What resource**: The executionLog (which is an ArrayList<String>)

**Why it needs protection**: ArrayList is not thread-safe. Concurrent access by multiple threads can lead to data corruption, missing log entries, or ConcurrentModificationException during execution

**Synchronization mechanism used**: ReentrantLock (specifically logLock)

**Code snippet**:
logLock.lock();
try {
    executionLog.add(message);
} finally {
    logLock.unlock();
}

**Justification**: I used a dedicated lock to ensure Mutual Exclusion during write operations to the list. This guarantees that each log entry is safely added and preserved, maintaining a reliable record of the simulation's events without crashing the program.</String>

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To manage and limit concurrent access to the simulated CPU resource. It ensures that only a specific number of processes can be in the "Running" state at any given time

**Number of permits and why**: 1 permit. This is because the simulation models a single-core processor where only one process should execute at a time to maintain orderly scheduling

**Where implemented**: In the Process class within the run() and runToCompletion() methods

**Code snippet**:
try {
    SharedResources.cpuSemaphore.acquire();
    // process execution logic
} finally {
    SharedResources.cpuSemaphore.release();
}

**Effect on program behavior**:It prevents multiple processes from executing their "quantum" simultaneously. This enforces the Mutual Exclusion principle for the CPU resource, resulting in a realistic and synchronized Round Robin scheduling behavior where processes wait their turn in the queue 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: # Commands used (run the program at least 5 times)
# 1. Open the terminal/command prompt.
# 2. Navigate to the project directory.
# 3. Run the following command 5 times consecutively:
java Main
# 4. Compare the 'Execution Log' and 'Final Statistics' after each run.

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
Running the program multiple times with the same Student ID (445052190) produced identical and correct results each time. The total number of context switches, completed processes, and average waiting times remained constant across all 5 runs, proving the simulation's stability

**Why synchronization is necessary**: 
Without synchronization, Race Conditions could occur because multiple threads access shared resources like contextSwitchCount and executionLog. Even if not always visible, concurrent updates to these non-atomic variables can lead to data loss or corruption. Protection is necessary to ensure Mutual Exclusion, so only one thread modifies a resource at a time, guaranteeing data integrity

**Conclusion**: The implementation of Locks and Semaphores successfully synchronized the multithreaded environment. These mechanisms prevented race conditions and ensured consistent, reliable output for the Round Robin scheduling simulation, even under concurrent thread execution

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**:I monitored the program execution for any runtime crashes, specifically checking if multiple threads adding messages to the executionLog simultaneously would trigger a ConcurrentModificationException 

**Results**: The program ran smoothly without any exceptions or crashes. All log entries were recorded correctly, and no threading errors were observed during the multiple execution trials

**What this proves**: This proves that using a Lock (logLock) for the ArrayList effectively synchronized access. It prevented the common threading exception that occurs when a non-thread-safe collection is modified by multiple threads at the same time

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: Based on the manual calculation for the Round Robin algorithm using Student ID 445052190, the expected total context switches and burst times should match the theoretical scheduling logic provided in the assignment requirements

**Actual values**: The actual output from the simulation matched the expected calculations perfectly:
• Total Burst Time: 52912
• Context Switches: 10
• Completed Processes: 5

**Analysis**: The match between expected and actual values demonstrates that the synchronization mechanisms (Locks and Semaphores) did not interfere with the logic of the Round Robin scheduler. The program is both thread-safe and mathematically accurate

---

### Test 4: Different Scenarios
**Scenario tested**:Testing the simulation with a different Time Quantum (e.g., changing it from 4000ms to 2000ms) and varying the number of processes

**Purpose**: To observe how the scheduling behavior and performance metrics change when the time slice is adjusted. This helps in understanding the impact of quantum size on context switching and waiting times

**Results**: Reducing the time quantum increased the number of Context Switches as processes were interrupted more frequently. However the synchronization mechanisms Locks and Semaphores continued to work perfectly maintaining data integrity regardless of the increased scheduling overhead

**What I learned**: I learned that the choice of time quantum is a trade-off between responsiveness and overhead. Most importantly, I learned that a robust synchronization design using Binary Semaphores and Fine-grained Locks ensures the system remains stable and accurate under any workload or configuration

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

I learned that synchronization is essential in multithreaded systems to prevent race conditions and ensure data integrity Managing shared resources like the CPU and global counters requires a deep understanding of Mutual Exclusion. The biggest challenge was balancing performance with safety as over-locking can lead to bottlenecks I gained insight into how Locks and Semaphores provide controlled access to resources making concurrent programs predictable and reliable. This project highlighted that writing thread-safe code is not just about correctness, but also about careful design and testing

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: Banking Systems: When two people withdraw money from the same account simultaneously synchronization ensures the balance is updated correctly for each transaction without losing data

**Example 2**: Online Ticket Booking: Systems like airline or cinema bookings use synchronization to prevent two users from purchasing the exact same seat at the same time

---

### How I would explain synchronization to others:


Imagine a small office with only one printer that many employees need to use. If everyone tries to print at the same time, the pages will get mixed up and ruined. Synchronization is like having a "sign-in sheet" or a "key" for the printer. Only the person with the key can use it, while others wait their turn in line. Once finished, they pass the key to the next person. This ensures that every document is printed perfectly and everyone gets their turn without any chaos
---

## Part 6: GitHub Repository Information

**Repository URL**: [https://github.com/Layan1-alfaisal-445/OS-Assignment3-Layan-alfaisal](https://github.com/Layan1-alfaisal-445/OS-Assignment3-Layan-alfaisal)

**Number of commits**: 4 

**Commit messages**: 
1.	Initial commit with starter files
2.	Implemented ReentrantLocks for counter and log protection
3.	Added CPU Semaphore to manage process execution
4.	Finalized Round Robin logic and updated documentation

---

## Summary

**Total time spent on assignment**: 10 hours

**Key takeaways**: 
1.	Understanding the critical role of Mutual Exclusion in shared resource management
2.	Mastering the use of Semaphores to simulate hardware constraints like CPU cores
3.	Learning how to design thread-safe structures to prevent Race Conditions

**Most challenging aspect**: Ensuring that locks were always released using try-finally blocks to prevent potential Deadlocks especially when managing multiple independent counters

**What I'm most proud of**: Successfully synchronizing a complex multithreaded environment where the final simulation statistics perfectly match the theoretical Round Robin calculations for my Student ID

---

**End of Documentation**
