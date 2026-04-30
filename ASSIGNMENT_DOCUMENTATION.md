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
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

**Conclusion**: 

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 

**Results**: 

**What this proves**: 

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 

**Actual values**: 

**Analysis**: 

---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 

**Results**: 

**What I learned**: 

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[6-8 sentences about key concepts, challenges, insights]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 

**Example 2**: 

---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 

**Commit messages**: 
1. 
2. 
3. 
4. 

---

## Summary

**Total time spent on assignment**: 

**Key takeaways**: 
1. 
2. 
3. 

**Most challenging aspect**: 

**What I'm most proud of**: 

---

**End of Documentation**
