# Condition Variables and Monitors

A mutex answers the question "may I touch this data now?". A condition variable answers a different question: "the data is not in the state I need, so how do I sleep until it is?". A monitor is a lock and its condition variables packaged with the data they protect.

## Why a lock alone is not enough

A consumer that finds an empty queue has two poor options with only a mutex. It can loop, checking the queue again and again, which wastes a CPU core. It can sleep for a fixed time, which adds delay. A condition variable gives a third option: release the lock, sleep, and be woken exactly when another thread changes the state.

## The three operations

| Operation | Java (`Object`) | Java (`Condition`) | Python | Go |
|---|---|---|---|---|
| Wait | `wait()` | `await()` | `cond.wait()` | `cond.Wait()` |
| Wake one waiter | `notify()` | `signal()` | `cond.notify()` | `cond.Signal()` |
| Wake all waiters | `notifyAll()` | `signalAll()` | `cond.notify_all()` | `cond.Broadcast()` |

`wait` does three things as one atomic step: it releases the lock, puts the thread to sleep, and reacquires the lock before it returns. A thread must hold the lock when it calls any of the three operations.

## The pattern

Every correct use has the same shape.

```python
with cond:
    while not ready():      # 1. check in a loop
        cond.wait()         # 2. sleep with the lock released
    use_the_state()         # 3. act while still holding the lock
```

And on the other side:

```python
with cond:
    change_the_state()
    cond.notify_all()
```

## Why the check is a loop

The `while` is the detail interviewers look for. There are three reasons for it.

1. **Another thread took the item first.** Two consumers are woken for one item. The first takes it, and the second must wait again.
2. **Spurious wake-ups.** Operating systems may wake a waiting thread with no signal at all, and the language specifications allow it.
3. **A broadcast for a different reason.** With one condition variable for several states, a thread can be woken by a change it does not care about.

With an `if`, each of these cases lets a thread continue while its condition is false.

## Lost wake-ups

A condition variable has no memory. If a thread signals when nobody is waiting, the signal disappears. A correct program therefore never waits for a signal alone. It waits for a state, such as "the queue is not empty", that is stored in a variable and checked under the lock. When a one-time event must be remembered, a latch or an event object is the right tool, as in [print in order](../problems/print-in-order.md).

## One wake-up or all

| Use | When |
|---|---|
| Wake one | Every waiter waits for the same condition, and one state change satisfies one waiter |
| Wake all | Waiters wait for different conditions on the same variable, or one change can satisfy several |

Waking one is more efficient, and waking all is safer. When in doubt, wake all, and state the cost: with 50 waiters, 49 wake, check, and sleep again.

## Monitors

A monitor is an object whose methods all run under one lock, with condition variables for waiting. Every Java object is a monitor: `synchronized` supplies the lock, and `wait` and `notify` supply one condition. The [bounded buffer](../problems/bounded-buffer.md) is the standard monitor, with two conditions on one lock.

## How this is asked

- "Why must `wait` be called inside a loop?"
- "What is the difference between `notify` and `notifyAll`?"
- "What is the difference between `wait` and `sleep` in Java?" (`wait` releases the lock and `sleep` keeps it.)
- "What is a spurious wake-up?"
- "Implement a blocking queue."

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-condition-variables-and-monitors) applies this pattern across its practice problems.
- Lesson: [Condition Variables](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/4-condition-variables?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-condition-variables-and-monitors), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Cond: Waiting for a Condition](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/cond-waiting-for-a-condition?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-condition-variables-and-monitors), in Grokking Concurrency in Go
- Lesson: [Conditions, Events, Semaphores, and Barriers](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/conditions-events-semaphores-and-barriers?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-condition-variables-and-monitors), in Grokking Concurrency in Python
- [Grokking Concurrency in Python](https://www.designgurus.io/course/grokking-concurrency-in-python?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-condition-variables-and-monitors) covers `threading.Condition`, `Event`, and `queue.Queue`.
- Practice: the [bounded buffer](../problems/bounded-buffer.md) and the [odd-even printer](../problems/odd-even-printer.md).
