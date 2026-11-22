# Locks, Mutexes, and Semaphores

A mutex (short for mutual exclusion) is a lock that one thread at a time can hold. A semaphore is a counter of permits that up to N threads can hold. The two are asked about together because a semaphore with one permit looks like a mutex, and the differences are the point of the question.

## The primitives

| Primitive | What it allows | Who may release it | Typical use |
|---|---|---|---|
| Mutex | 1 thread | Only the thread that locked it | Protecting shared data |
| Reentrant lock | 1 thread, which may lock it again without blocking | The owner, once per acquisition | A locked method that calls another locked method |
| Readers-writers lock | Many readers or 1 writer | The holder | Data that is read far more often than it is written |
| Counting semaphore | Up to N threads | Any thread | Limiting concurrent use, such as 10 database connections |
| Binary semaphore | 1 thread | Any thread | Signaling between threads |
| Spinlock | 1 thread, and waiters loop instead of sleeping | The owner | Critical sections of a few instructions |

## Mutex versus binary semaphore

This is the most common question on the page. Both admit one thread, but they differ in ownership.

- A mutex has an owner. The thread that locks it must unlock it.
- A semaphore has no owner. One thread can acquire a permit and a different thread can release it.

That difference makes a semaphore a signaling tool. A producer can release a permit to announce that an item is ready, and a consumer that never acquired anything can then proceed.

## Reentrancy

A reentrant lock counts how many times its owner has locked it. Java's `synchronized` and `ReentrantLock` are reentrant, and so is Python's `threading.RLock`. Go's `sync.Mutex` and Python's `threading.Lock` are not: a goroutine or thread that locks one twice blocks forever.

## A semaphore as a limiter

The example below allows at most 3 of 20 threads to call a slow service at the same time.

```python
import threading

limit = threading.Semaphore(3)

def call_service(request):
    with limit:
        return send(request)
```

In Go, a buffered channel of capacity 3 does the same job: a send takes a slot and a receive returns it.

## Choosing a lock

| Question | If yes |
|---|---|
| Is it one counter or one flag? | Use an atomic variable instead of a lock |
| Are reads at least 10 times more frequent than writes, and is the critical section long? | Consider a readers-writers lock |
| Must at most N threads proceed? | Use a semaphore |
| Does a thread need to wait for a condition, not only for access? | Add a [condition variable](condition-variables-and-monitors.md) |
| None of the above | Use a plain mutex |

A plain mutex is the correct default. The other choices are optimizations that need a reason.

## Rules that prevent most bugs

1. Release in a `finally` block, a `with` statement, or a `defer`, so that an exception cannot leave the lock held.
2. Hold a lock for the shortest time that keeps the data consistent. Never hold one during a network call.
3. When a thread needs two locks, always acquire them in the same order. See [deadlock, livelock, and starvation](deadlock-livelock-and-starvation.md).
4. Document which lock protects which data.

## How this is asked

- "What is the difference between a mutex and a semaphore?"
- "What is a reentrant lock, and when do you need one?"
- "When is a spinlock better than a mutex?" (When the wait is shorter than the cost of putting a thread to sleep, and more than one core is available.)
- "What is the difference between `synchronized` and `ReentrantLock` in Java?" (The second adds timed and interruptible locking, fairness, and several conditions per lock.)

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-locks-mutexes-and-semaphores) covers each primitive with problems that require it.
- Lesson: [Mutex Lock](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/1-mutex-lock?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-locks-mutexes-and-semaphores), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Semaphore](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/3-semaphore?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-locks-mutexes-and-semaphores), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Mutex: Protecting Shared State](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/mutex-protecting-shared-state?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-locks-mutexes-and-semaphores), in Grokking Concurrency in Go
- Practice: the [readers-writers lock](../problems/readers-writers-lock.md) and the [dining philosophers](../problems/dining-philosophers.md).
- Reference: the [Java, Python, and Go primitives table](../cheat-sheets/java-vs-python-vs-go-primitives.md).
- Free answer: [What Is a Semaphore?](https://www.designgurus.io/answers/detail/what-is-a-semaphore?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-locks-mutexes-and-semaphores), a short explanation with examples.
- Free answer: [When to Use a Recursive Mutex](https://www.designgurus.io/answers/detail/when-to-use-recursive-mutex?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-locks-mutexes-and-semaphores), which covers the reentrant lock case.
