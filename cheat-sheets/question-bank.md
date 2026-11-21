# Concurrency Interview Question Bank

Fifty conceptual questions that appear in concurrency and multithreading interviews, each with a one-line answer. Use the answer to check recall, and follow the link in each section for the full explanation.

## Fundamentals

Explained in [threads vs processes](../concepts/threads-vs-processes.md).

| # | Question | Short answer |
|---|---|---|
| 1 | What is the difference between a process and a thread? | A process has private memory, and threads inside one process share memory. |
| 2 | What is the difference between concurrency and parallelism? | Concurrency is several tasks in progress, and parallelism is several tasks executing at the same instant. |
| 3 | What do threads of one process share? | The heap, global variables, and open files, but not the stack. |
| 4 | What is a context switch? | Saving the state of one thread and loading the state of another. |
| 5 | What is a daemon thread? | A background thread that does not keep the process alive when the other threads finish. |
| 6 | What is thread-local storage? | A variable of which every thread has its own separate copy. |
| 7 | What is a goroutine? | A lightweight thread that the Go runtime schedules onto operating system threads. |
| 8 | What is a Java virtual thread? | A lightweight thread scheduled by the Java runtime, cheap enough to create one per task. |

## Races and atomicity

Explained in [race conditions](../concepts/race-conditions.md) and [atomics and memory visibility](../concepts/atomics-and-memory-visibility.md).

| # | Question | Short answer |
|---|---|---|
| 9 | What is a race condition? | A bug in which correctness depends on the timing of threads. |
| 10 | What is a data race? | Two unsynchronized accesses to the same memory, at least one of them a write. |
| 11 | Is `i++` thread-safe? | No, because it is a read, an addition, and a write. |
| 12 | What is a critical section? | Code that only one thread may execute at a time. |
| 13 | What is a check-then-act bug? | Another thread changes the state between a check and the action that depends on it. |
| 14 | What does `volatile` guarantee in Java? | Visibility of writes to other threads, but not atomic compound updates. |
| 15 | What is compare-and-swap? | An atomic instruction that writes a new value only if the current value matches an expected one. |
| 16 | What is the ABA problem? | A value changes from A to B and back to A, so a compare-and-swap succeeds although the state changed. |
| 17 | What does "happens-before" mean? | A guarantee that one action's effects are visible to another action. |
| 18 | What does lock-free mean? | At least one thread always makes progress, and no thread waits on a lock. |

## Locks and semaphores

Explained in [locks, mutexes, and semaphores](../concepts/locks-mutexes-and-semaphores.md).

| # | Question | Short answer |
|---|---|---|
| 19 | What is a mutex? | A lock that one thread at a time can hold. |
| 20 | How does a mutex differ from a binary semaphore? | A mutex has an owner that must release it, and a semaphore can be released by any thread. |
| 21 | What is a counting semaphore used for? | Limiting how many threads use a resource at once. |
| 22 | What is a reentrant lock? | A lock that its owner can acquire again without blocking. |
| 23 | When does a readers-writers lock help? | When reads are much more frequent than writes and the critical section is long. |
| 24 | What is a spinlock? | A lock whose waiters loop instead of sleeping, suited to very short critical sections. |
| 25 | What is a fair lock? | A lock that serves waiting threads in arrival order, at some cost in throughput. |
| 26 | How does `synchronized` differ from `ReentrantLock`? | `ReentrantLock` adds timeouts, interruptible locking, fairness, and several conditions. |
| 27 | What is lock contention? | Threads spending time waiting for a lock that another thread holds. |
| 28 | What is optimistic locking? | Proceeding without a lock and retrying if a version check shows that the data changed. |

## Deadlock and liveness

Explained in [deadlock, livelock, and starvation](../concepts/deadlock-livelock-and-starvation.md).

| # | Question | Short answer |
|---|---|---|
| 29 | What is a deadlock? | Threads that wait for each other forever. |
| 30 | What are the four conditions for deadlock? | Mutual exclusion, hold and wait, no preemption, and circular wait. |
| 31 | What is the most common way to prevent deadlock? | Acquiring locks in one global order. |
| 32 | What is a livelock? | Threads that stay active and keep retrying but make no progress. |
| 33 | What is starvation? | One thread waits without limit while others proceed. |
| 34 | What is priority inversion? | A high-priority thread waits for a lock held by a low-priority thread that is not being scheduled. |
| 35 | How do you find a deadlock in a running Java service? | Take a thread dump, which lists held and awaited locks and reports cycles. |

## Waiting and signaling

Explained in [condition variables and monitors](../concepts/condition-variables-and-monitors.md).

| # | Question | Short answer |
|---|---|---|
| 36 | What is a condition variable? | An object that lets a thread release a lock and sleep until it is signaled. |
| 37 | Why is `wait` called in a loop? | A woken thread may find the condition false again. |
| 38 | What is a spurious wake-up? | A thread wakes from `wait` although nobody signaled it. |
| 39 | How does `notify` differ from `notifyAll`? | One wakes a single waiter, and the other wakes all of them. |
| 40 | How does `wait` differ from `sleep` in Java? | `wait` releases the lock and `sleep` keeps it. |
| 41 | What is a monitor? | A lock and its condition variables packaged with the data they protect. |
| 42 | What is a `CountDownLatch`? | A gate that opens once a counter reaches 0 and then stays open. |

## Pools, async, and languages

Explained in [thread pools](../concepts/thread-pools.md) and [async vs threads](../concepts/async-vs-threads.md).

| # | Question | Short answer |
|---|---|---|
| 43 | Why use a thread pool? | To reuse threads and to limit how many run at once. |
| 44 | How large should a pool be? | About the number of cores for CPU-bound work, and larger for work that waits on I/O. |
| 45 | What is a future? | An object that will hold the result of a task that has not finished yet. |
| 46 | What is back pressure? | A slow consumer making its producer slow down. |
| 47 | What is the global interpreter lock? | A lock in the standard Python interpreter that lets one thread at a time run Python bytecode. |
| 48 | When is async better than threads? | When there are very many tasks that mostly wait on I/O. |
| 49 | What happens if a coroutine makes a blocking call? | The event loop stops, and every other task waits. |
| 50 | How does a buffered Go channel differ from an unbuffered one? | A buffered send blocks only when the buffer is full, and an unbuffered send blocks until a receiver is ready. |

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-question-bank) gives the full explanation behind each of these answers, with coding problems.
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-question-bank) and [Grokking Concurrency in Python](https://www.designgurus.io/course/grokking-concurrency-in-python?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-question-bank) cover the language-specific questions.
- Coding practice: the [problems](../problems/README.md).
- Free answer: [20 Concurrency and Parallelism Interview Questions (Answered)](https://www.designgurus.io/answers/detail/what-are-some-concurrency-and-parallelism-interview-questions?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-question-bank), more questions with model answers.
- Free answer: [What Is Multithreading in Java, With Example Interview Questions](https://www.designgurus.io/answers/detail/what-is-multithreading-in-java-with-example-interview-questions?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-question-bank), a Java-specific set of questions.
