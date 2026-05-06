# Assignment 3 - Complete Documentation

**Student Name**: [Your Full Name]  
**Student ID**: [Your ID]  
**Date Submitted**: [Submission Date]

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

### Entry 1 - [Date, Time]
**What I implemented**: 
Set up shared resources, locks, and semaphores for synchronization.
**Challenges encountered**: 
Identifying which shared variables required protection.
**How I solved it**: 
Used separate ReentrantLocks and a semaphore to prevent race conditions.
**Testing approach**: 
Ran multiple simulations to verify stable counter values.
**Time spent**: 
1 hours
---

### Entry 2 - [Date, Time]
**What I implemented**: 
Implemented the process execution logic, including quantum handling and progress bars.
**Challenges encountered**: 
Avoiding mixed or overlapping console output from multiple threads.
**How I solved it**: 
Used cpuSemaphore to ensure only one process prints and executes at a time.
**Testing approach**: 
Observed console output to confirm clean, non-overlapping progress updates.
**Time spent**: 
30 minute
---

### Entry 3 - [Date, Time]
**What I implemented**: 
Added final statistics: context switches, completed processes, waiting times, and log summary.
**Challenges encountered**: 
Calculating accurate waiting time for each process.
**How I solved it**: 
Computed waiting time using (completionTime - creationTime) - burstTime and protected updates with locks.
**Testing approach**: 
Manually compared calculated waiting times with printed results.
**Time spent**: 
1 hour
---



## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:
One race condition existed in the shared counter contextSwitchCount because multiple threads increment it without protection in the original version. Without locking, two threads could read the same value and write back the same incremented value, causing lost updates.
Another race condition affected executionLog, where multiple threads could add log entries at the same time, corrupting the list or causing inconsistent ordering.
Example from the code:
contextSwitchCount++; and executionLog.add(message);  
Both required locks to ensure atomic updates.

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
A ReentrantLock provides exclusive access to a specific shared variable, ensuring only one thread modifies it at a time. I used ReentrantLocks for the counters and the log list because each resource needed strict mutual exclusion.
A Semaphore controls access to a shared resource with a limited number of permits. I used cpuSemaphore with one permit to ensure only one process executes on the “CPU” at a time.
Locks protected data; the semaphore controlled scheduling behavior.
---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:
Deadlock occurs when two or more threads wait forever because each is holding a resource the other needs. One prevention technique is using try-finally to guarantee locks are always released, which I applied in all shared-resource methods.
Another technique is consistent lock ordering, but since each counter has its own lock and threads never acquire multiple locks at once, circular waiting is avoided.
These choices ensured no thread could hold a lock indefinitely.
---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:
I used separate locks for each counter (fine‑grained locking) because the counters are independent. This improves concurrency by allowing different threads to update different counters at the same time without blocking each other.
A single lock (coarse‑grained) would simplify the design but reduce performance because all updates would be serialized.
Fine‑grained locking has slightly more complexity but provides better parallelism.
Since the counters do not depend on each other, separate locks give the highest concurrency and avoid unnecessary blocking.
---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
contextSwitchCount, completedProcessCount, totalWaitingTime
**Why they need protection**: 
Multiple threads update these counters at the same time, which can cause lost updates or incorrect values.
**Synchronization mechanism used**: 
A separate ReentrantLock for each counter
**Code snippet**:
```java
contextSwitchLock.lock();
try {
    contextSwitchCount++;
} finally {
    contextSwitchLock.unlock();
}

```

**Justification**: 
Each counter is independent, so using separate locks prevents race conditions while allowing higher concurrency.
---

### Critical Section #2: Execution Log

**What resource**: 
The shared list executionLog.
**Why it needs protection**: 
Multiple threads may add log entries simultaneously, which can corrupt the list or cause inconsistent ordering.
**Synchronization mechanism used**: 
A dedicated ReentrantLock (logLock)
**Code snippet**:
```java
logLock.lock();
try {
    executionLog.add(message);
} finally {
    logLock.unlock();
}

```

**Justification**: 
The lock ensures only one thread modifies the list at a time, keeping the log consistent and thread‑safe.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
To ensure only one process uses the simulated CPU at a time.
**Number of permits and why**: 
permit — because the CPU can run only one 1 process at once.
**Where implemented**: 
Inside the run() and runToCompletion() methods of the Process class.
**Code snippet**:
```java
SharedResources.cpuSemaphore.acquire();
try {
    // process execution
} finally {
    SharedResources.cpuSemaphore.release();
}

```

**Effect on program behavior**: 
It prevents multiple threads from executing CPU work simultaneously, ensuring clean output and correct scheduling behavior.
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results
# Run at least 5 times


**Testing procedure**: 
```bash
# Run at least 5 times
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
), not from synchronization issues.
```

**Results**: 
All runs produced consistent behavior: same number of processes, correct context switch increments, and no corrupted output. Minor variations only came from randomness (burst times), not from synchronization issues.

**Why synchronization is necessary**: 
 
Without locks, counters like contextSwitchCount and shared lists like executionLog could produce lost updates, inconsistent values, or corrupted logs due to race conditions.

**Conclusion**: 
Synchronization ensured stable and predictable results across repeated executions.
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 
Ran the program under heavy load and observed log updates during multiple thread executions.

**Results**: 
No exceptions occurred because logLock ensures only one thread modifies the list at a time.
**What this proves**: 
The logging mechanism is thread‑safe and properly synchronized.
---

### Test 3: Correctness Verification
**What I tested**: Verified final statistics such as total context switches, completed processes, and waiting times.

**Expected values**: 

Completed processes = number of created processes

Context switches = increments equal to number of quantum executions

Waiting time = (completionTime - creationTime) - burstTime
**Actual values**: 
All final printed values matched expected behavior and were logically consistent.
**Analysis**: 
Correct synchronization ensured accurate counters and prevented lost updates.
---

### Test 4: Different Scenarios
**Scenario tested**: Changed time quantum and increased number of processes

**Purpose**: 
To verify that synchronization still works under different scheduling loads.
**Results**: 
Program behaved correctly: no deadlocks, no overlapping output, and all processes completed.
**What I learned**: 
The locking design scales well and maintains correctness even with higher concurrency.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:
I learned that synchronization is essential when multiple threads access shared data at the same time. Without proper locking, race conditions can easily corrupt counters, logs, and timing values. I also learned how different mechanisms—like ReentrantLocks and Semaphores—solve different types of concurrency problems. Implementing try‑finally blocks taught me the importance of always releasing locks to avoid deadlocks. I gained a better understanding of fine‑grained vs. coarse‑grained locking and how design choices affect performance. Overall, synchronization requires careful planning, especially when multiple threads interact with shared resources.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking systems where multiple transactions update the same account balance.
**Example 2**: 
Operating systems managing CPU scheduling, where multiple processes compete for limited hardware resources.
---

### How I would explain synchronization to others:
Synchronization is like having one person at a time use a shared tool so no one breaks it or causes mistakes. If two people try to write on the same paper at once, the result becomes messy—threads behave the same way. Locks act like “please wait your turn,” and semaphores act like “only a limited number of people can enter.” It ensures that shared data stays correct even when many threads run at the same time.
---

## Part 6: GitHub Repository Information

**Repository URL**: 
https://github.com/layan626/OS-Assignment3-Layan-ibrahim.git
**Number of commits**: 
commits 15
**Commit messages**: 
1. Add shared resources and synchronization locks
2. Implement process execution with CPU semaphore
3. Add final statistics and waiting time calculation
4. Improve error handling with try/catch/finally

---

## Summary

**Total time spent on assignment**: 
Approximately 5–6 hours
**Key takeaways**: 
1. Synchronization is essential to prevent race conditions
2. Locks and semaphores solve different concurrency problems
3. Fine‑grained locking improves performance when resources are independent.

**Most challenging aspect**: 
Ensuring correct waiting time calculation and preventing output overlap between threads.
**What I'm most proud of**: 
Building a fully synchronized CPU scheduler that runs smoothly without race conditions or exceptions.
---

**End of Documentation**
