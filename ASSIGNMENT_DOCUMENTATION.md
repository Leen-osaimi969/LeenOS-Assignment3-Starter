# Assignment 3 - Complete Documentation

**Student Name**: [Leen Lafi ALosaimi]  
**Student ID**: [444953969]  
**Date Submitted**: [4/ 5 / 2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [(https://drive.google.com/file/d/1_JyjgENj3L3IUWf1RDyAYA5RFMXB6Zux/view?usp=sharing)]

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

### Entry 1 - [30 Apr, 8:00]
**What I implemented**: 
I added synchronization mechanisms to the shared resources. I created separate ReentrantLock objects to protect the shared counters and execution log, and I added a Semaphore to control CPU access so only one process can execute the critical CPU section at a time.

**Challenges encountered**: 
The main challenge was identifying which variables were shared between threads and needed protection. The shared counters and ArrayList could cause race conditions if multiple threads accessed them at the same time.

**How I solved it**: 
I used ReentrantLock for mutual exclusion around each shared variable update. I also used a binary Semaphore with on

**Testing approach**: 
I ran the program and checked that the process execution output appeared correctly without race condition errors. I also verified that the counters and execution log were updated through synchronized methods.

**Time spent**: 
Around 30 minutes.



#### Entry 2 - [30 Apr, 8:40 PM]

**What I implemented**:
I implemented thread-safe methods for updating shared data, including incrementing context switches, completed processes, and adding waiting time. I wrapped these operations using locks with try-finally blocks to ensure proper unlocking.

**Challenges encountered**:
The challenge was making sure locks are always released even if an exception occurs. Forgetting to release a lock could cause deadlocks and stop the program.

**How I solved it**:
I used try-finally blocks for every lock to guarantee that unlock() is always executed. I also separated locks for each shared variable to improve concurrency.

**Testing approach**:
I ran multiple processes and observed that counters updated correctly without inconsistent values. I checked that no deadlocks occurred during execution.

**Time spent**:
Around 35 minutes.

---


#### Entry 3 - [30 Apr, 9:20 PM]

**What I implemented**:
I implemented the main execution logic inside the run() method, including CPU scheduling using a semaphore, handling time quantum execution, tracking remaining time, and logging execution steps.

**Challenges encountered**:
Managing multiple responsibilities inside the run method was challenging, especially coordinating semaphore control, timing (sleep), and updating shared resources correctly.

**How I solved it**:
I structured the method into clear sections: acquiring the semaphore, executing the process in steps, updating progress, and finally releasing the semaphore. I also reused the synchronized methods from SharedResources.

**Testing approach**:
I executed the program and verified that processes run sequentially due to the semaphore. I checked progress output, context switches, and completion behavior.

**Time spent**:
Around 45 minutes.

---


## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

One race condition occurs with the shared variable contextSwitchCount. Multiple threads may execute contextSwitchCount++ at the same time, which is not an atomic operation. This can cause lost updates, where some increments are overwritten, leading to an incorrect total count.

Another race condition exists in the shared executionLog ArrayList. Since ArrayList is not thread-safe, multiple threads calling executionLog.add(message) concurrently can corrupt the internal structure or result in missing log entries.

Concurrent access is a problem because threads interleave execution unpredictably, especially during read-modify-write operations. Without synchronization, the program cannot guarantee consistency of shared data.

As a result, incorrect behavior such as wrong counters, incomplete logs, or even runtime errors may occur.

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

`ReentrantLock` is used for mutual exclusion, meaning only one thread can access a critical section at a time. I used it to protect shared variables such as `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`, and `executionLog`.

`Semaphore` is used to control access to a limited resource. In my code, I used `cpuSemaphore` to make sure only one process can access the CPU execution section at a time.

The reason I used both is that locks protect shared data, while the semaphore controls process execution access.

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

Deadlock happens when two or more threads are waiting for each other forever, so none of them can continue.

One prevention technique is using try-finally blocks. In my code, every lock is released inside finally, so the lock is unlocked even if an error happens.

Another prevention technique is avoiding holding multiple locks at the same time. In my implementation, each method uses one lock for one shared resource, which reduces the chance of circular waiting.

I also release the semaphore inside a finally block, so CPU access is always released after execution.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

I used separate locks for each shared counter, which is a fine-grained locking approach. For example, I used `contextSwitchLock`, `completedProcessLock`, and `waitingTimeLock` instead of using one lock for everything.

I made this choice because the counters are independent from each other. Updating the context switch count does not depend on updating the completed process count or waiting time.

Fine-grained locking provides better concurrency because different threads can update different counters without blocking each other unnecessarily. A coarse-grained lock is simpler because it uses one lock, but it reduces performance because only one thread can access any shared counter at a time.

The trade-off is that fine-grained locking is slightly more complex to manage, while coarse-grained locking is easier but less efficient. Since the three counters are independent, separate locks provide better concurrency and a cleaner design for this program.
---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**:
`contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`.

**Why they need protection**:
These variables are shared between multiple process threads. If more than one thread updates them at the same time, race conditions can happen and the final values may be incorrect.

**Synchronization mechanism used**:
I used separate `ReentrantLock` objects for each counter variable.

**Code snippet**:
```java
public static void incrementContextSwitch() {
    contextSwitchLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        contextSwitchLock.unlock();
    }
}

public static void incrementCompletedProcess() {
    completedProcessLock.lock();
    try {
        completedProcessCount++;
    } finally {
        completedProcessLock.unlock();
    }
}

public static void addWaitingTime(long time) {
    waitingTimeLock.lock();
    try {
        totalWaitingTime += time;
    } finally {
        waitingTimeLock.unlock();
    }
}

**Justification**:
I used separate locks because each counter is independent. This prevents race conditions while still allowing better concurrency than using one lock for all counters. 

---

### Critical Section #2: Execution Log
**What resource**:
`executionLog`, which is a shared `ArrayList<String>`.

**Why it needs protection**:
`ArrayList` is not thread-safe. If multiple process threads add log messages at the same time, the log could lose entries, store incorrect data, or cause runtime issues.

**Synchronization mechanism used**:
I used `ReentrantLock` through `logLock`.

**Code snippet**:
```java
public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}

**Justification**: 
The execution log is shared by all process threads, so access must be synchronized. Using logLock makes sure only one thread can add a message at a time, which keeps the log consistent and prevents race conditions.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**:
The semaphore is used to control access to the CPU, ensuring that only one process executes the critical CPU section at a time.

**Number of permits and why**:
I used a semaphore with 1 permit (`new Semaphore(1)`) to make it behave like a binary semaphore. This ensures mutual exclusion so only one thread can access the CPU at a time.

**Where implemented**:
It is implemented in the `run()` and `runToCompletion()` methods where each process acquires the semaphore before execution and releases it after finishing.

**Code snippet**:
```java
SharedResources.cpuSemaphore.acquire();
try {
    // process execution
} finally {
    SharedResources.cpuSemaphore.release();
}

**Effect on program behavior**: 
The semaphore ensures that processes do not execute the CPU section simultaneously. This prevents conflicts and simulates a real CPU where only one process runs at a time, resulting in correct and predictable execution order.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync


**Results**: 
(Show that running multiple times produces consistent, correct results)

The program ran correctly multiple times. The processes executed without crashing, the execution log was updated, and the final summary values were printed successfully. The shared counters such as context switches, completed processes, and waiting time stayed consistent because they were updated through synchronized methods.

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

Synchronization is necessary because multiple process threads access shared resources at the same time. Without locks, contextSwitchCount++ and completedProcessCount++ could lose updates because incrementing is not atomic. Also, totalWaitingTime could become incorrect if multiple threads add waiting time at the same time. The shared executionLog also needs protection because ArrayList is not thread-safe and concurrent writes could cause missing entries or runtime errors.

**Conclusion**: 

The consistency test shows that the synchronization mechanisms help the program produce stable and correct behavior across multiple runs.

---

### Test 2: Exception Testing
**Testing procedure**:
I ran the program with multiple threads adding log messages at the same time to the shared `executionLog`.

**Results**:
No `ConcurrentModificationException` occurred during execution. The log entries were added correctly without errors or missing data.

**What this proves**:
This proves that using `ReentrantLock` for the execution log successfully prevents concurrent modification issues and makes the `ArrayList` thread-safe.

---

### Test 3: Correctness Verification
**Expected values**:
- Context switches should increase based on process execution
- Completed processes should equal the total number of processes
- Waiting time should be a valid accumulated value

**Actual values**:
The output showed correct values for context switches, completed processes, and waiting time after execution.

**Analysis**:
The actual values matched the expected values, which confirms that synchronization mechanisms (locks and semaphore) correctly protected shared variables and ensured accurate updates.
---

### Test 4: Different Scenarios
**Scenario tested**:
I tested the program with a different time quantum value.

**Purpose**:
The purpose was to check if the scheduler still works correctly when the execution time slice changes.

**Results**:
The program still executed correctly. Processes used the CPU one at a time, context switches were updated, and completed processes were counted correctly.

**What I learned**:
Changing the time quantum affects how often processes switch, but synchronization keeps the shared data safe and consistent.
---

## Part 5: Reflection and Learning
### What I learned about synchronization:

I learned that synchronization is important when multiple threads share the same data. Without it, race conditions can happen and cause incorrect results. Using `ReentrantLock` helped me control access to critical sections more clearly than using synchronized blocks. I also learned how `Semaphore` can be used to limit access to a resource like CPU execution. One challenge was making sure locks are always released, which I solved using try-finally blocks. I also understood the difference between coarse-grained and fine-grained locking. Overall, synchronization improves program correctness and stability.

---

### Real-world applications:

**Example 1**: Banking systems where multiple users update the same account balance.

**Example 2**: Operating systems where multiple processes compete for CPU resources.

---

### How I would explain synchronization to others:

Synchronization is like organizing access to shared things. Imagine several people trying to use the same printer at once. Without rules, they may interrupt each other and cause errors. Synchronization makes sure only one person uses it at a time or controls how many can use it. In programming, it prevents threads from interfering with each other and keeps data correct.
---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Leen-osaimi969/LeenOS-Assignment3-Starter.git

**Number of commits**: 6 

**Commit messages**: 
1. set my student ID: 444953969
2. Add synchronization: ReentrantLocks, Semaphore, and protected counters
3. Add thread-safe methods for completed process count and waiting time
4. Add thread-safe logging using ReentrantLock

---

## Summary

**Total time spent on assignment**: 6 houres

**Key takeaways**: 
1. Synchronization is essential to prevent race conditions when multiple threads access shared data.
2. Using ReentrantLock gives better control compared to synchronized blocks, especially with try-finally.
3. Semaphores are useful for limiting access to resources (like CPU simulation), not just protecting data.

**Most challenging aspect**: 
Understanding where exactly race conditions could happen and deciding the correct place to apply locks without overcomplicating the code

**What I'm most proud of**: 
Successfully implementing a thread-safe scheduler with correct synchronization, and ensuring consistent results across multiple runs without errors.
---

**End of Documentation**
