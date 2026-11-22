# Deadlock, Livelock, and Starvation

These are the three ways in which a concurrent program stops making progress without crashing. In a deadlock, threads wait for each other forever. In a livelock, threads stay active but accomplish nothing. In starvation, one thread waits without limit while others proceed.

## The three compared

| Failure | Threads are | CPU use | Typical cause |
|---|---|---|---|
| Deadlock | Blocked | None | Two threads acquire two locks in opposite orders |
| Livelock | Running | High | Threads keep reacting to each other and retrying |
| Starvation | One blocked, others running | Normal | An unfair lock or a priority rule that always favors others |

## Deadlock

The smallest example needs two threads and two locks.

| Step | Thread 1 | Thread 2 |
|---|---|---|
| 1 | Locks A | Locks B |
| 2 | Waits for B | Waits for A |

Neither thread can continue, and neither will release what it holds.

## The four conditions

A deadlock can occur only when four conditions hold at the same time. They are known as the Coffman conditions, after Edward Coffman, who described them.

| Condition | Meaning | How to remove it |
|---|---|---|
| Mutual exclusion | A resource can be held by one thread only | Use lock-free or immutable data, where possible |
| Hold and wait | A thread holds one resource while waiting for another | Acquire all locks at once, or release everything and retry |
| No preemption | A resource cannot be taken away from its holder | Use `tryLock` with a timeout, and release on failure |
| Circular wait | A cycle of threads exists, each waiting for the next | Acquire locks in one global order |

Removing any one condition prevents deadlock. Lock ordering, which removes circular wait, is the cheapest and the most common answer.

## Lock ordering in practice

A transfer between two accounts needs both account locks. If one thread transfers from account 7 to account 12 while another transfers from 12 to 7, they can deadlock. The fix is to lock the account with the smaller identifier first, whatever the direction of the transfer.

```java
Account first = from.id < to.id ? from : to;
Account second = from.id < to.id ? to : from;
synchronized (first) {
    synchronized (second) {
        from.withdraw(amount);
        to.deposit(amount);
    }
}
```

The [dining philosophers](../problems/dining-philosophers.md) problem is the same idea with five locks.

## Livelock

In a livelock, every thread keeps running, but each one keeps undoing or repeating its work in response to the others. In code, livelock usually appears when threads use `tryLock`, fail, release, and retry at exactly the same moment. The fix is to wait for a random time before each retry, so that the threads no longer retry at the same instant.

## Starvation

Starvation appears when the rule for granting a lock can pass over the same thread repeatedly. A lock that always favors readers can delay a writer without limit. A fair lock, which serves waiting threads in arrival order, prevents it at the cost of lower throughput. The [readers-writers lock](../problems/readers-writers-lock.md) page shows one fix.

## Detecting a deadlock

- In Java, a thread dump (from `jstack` or `jcmd`) lists each thread, the lock it holds, and the lock it waits for, and it reports cycles directly.
- In Go, the runtime stops the program with "all goroutines are asleep" when every goroutine is blocked. It does not detect a deadlock among a subset.
- In Python, `faulthandler.dump_traceback()` prints the stack of every thread.

## How this is asked

- "What is a deadlock, and what conditions are required for one?"
- "How would you prevent a deadlock in a transfer between two accounts?"
- "What is the difference between a deadlock and a livelock?"
- "Your service has stopped responding and CPU use is 0 percent. What do you check first?"

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-deadlock-livelock-and-starvation) covers deadlock prevention with problems that fail in each of these three ways.
- Lesson: [Deadlock, Starvation, and Livelock](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/deadlock-starvation-and-livelock?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-deadlock-livelock-and-starvation), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Deadlocks](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/deadlocks?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-deadlock-livelock-and-starvation), in Grokking Concurrency in Go
- Lesson: [Livelocks and Starvation](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/livelocks-and-starvation?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-deadlock-livelock-and-starvation), in Grokking Concurrency in Go
- Related pages: [locks, mutexes, and semaphores](locks-mutexes-and-semaphores.md) and [common mistakes](../cheat-sheets/common-mistakes.md).
- Free article: [Deadlock vs Livelock: Key Differences and How to Prevent Both](https://www.designgurus.io/blog/deadlock-vs-livelock?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-deadlock-livelock-and-starvation), a longer comparison of the two failures.
- Free answer: [How to Avoid Deadlock in Java](https://www.designgurus.io/answers/detail/how-to-avoid-deadlock-in-java?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-deadlock-livelock-and-starvation), the prevention rules applied in one language.
