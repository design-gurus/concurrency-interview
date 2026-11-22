# Race Conditions

A race condition is a bug in which the result of a program depends on the order in which threads happen to run. The program can pass 10,000 test runs and fail on the next one, which is why interviewers treat the topic as a test of reasoning instead of testing.

## The standard example

The statement `count++` looks like one step. It is three: read the value, add 1, and write the result. Two threads can interleave those steps.

| Step | Thread A | Thread B | `count` in memory |
|---|---|---|---|
| 1 | Reads 5 | | 5 |
| 2 | | Reads 5 | 5 |
| 3 | Writes 6 | | 6 |
| 4 | | Writes 6 | 6 |

Two increments ran and the counter rose by 1. This is called a lost update. With 2 threads that each increment 100,000 times, the final value is often well below 200,000.

## Two terms that are often confused

| Term | Meaning | Example |
|---|---|---|
| Data race | Two threads access the same memory at the same time, at least one writes, and nothing synchronizes them | The `count++` above |
| Race condition | The correctness of the program depends on timing | Two threads both see "seat 14A is free" and both book it |

A program can have a race condition without a data race. If each step of "check the seat, then book the seat" takes a lock separately, every memory access is synchronized, and two customers still receive the same seat.

## Check-then-act

Most race conditions in application code have the same shape: a thread checks a condition and then acts on it, and another thread changes the condition between the two steps.

```java
if (!map.containsKey(key)) {   // check
    map.put(key, compute());   // act
}
```

Two threads can both pass the check. The fix is to make the check and the action one atomic step, which means a step that other threads can never observe half finished. In this case, `map.computeIfAbsent(key, k -> compute())` on a `ConcurrentHashMap` does that.

## A critical section

A critical section is a block of code that only one thread may execute at a time. The usual tools for protecting one are below.

| Fix | How it works | Typical use |
|---|---|---|
| Mutex | One thread holds the lock, and the others wait | Several variables that must change together |
| Atomic operation | The CPU performs read, change, and write as one instruction | One counter or one flag |
| Immutability | Data that never changes needs no protection | Configuration, messages |
| Confinement | Only one thread ever touches the data | Thread-local state, one goroutine that owns a map |
| Message passing | Threads send copies instead of sharing | Go channels, queues between workers |

## Finding races

Reading the code carefully is the main tool in an interview. In practice, tools help.

- Go has a built-in race detector: `go test -race`.
- Java and C++ have thread sanitizers and static analysis tools.
- A stress test with many threads and a final assertion makes a rare race more frequent, but it never proves that a race is absent.

## How this is asked

- "Is `i++` thread-safe? Why not?"
- "What is the difference between a data race and a race condition?"
- "This method uses a concurrent map. Is it thread-safe?" (Often no, because of check-then-act.)
- "Find the bug in this code" followed by a class with one unsynchronized field.

A strong answer names the exact interleaving that fails, as the table above does, instead of saying that the code "is not thread-safe".

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-race-conditions) teaches how to find and fix races through worked problems.
- Lesson: [Race Conditions and Critical Sections](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/critical-section-and-race-condition?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-race-conditions), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Data Races](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/data-races?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-race-conditions), in Grokking Concurrency in Go
- Lesson: [Race Conditions in Python Code](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/race-conditions-in-python-code?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-race-conditions), in Grokking Concurrency in Python
- Next pages: [locks, mutexes, and semaphores](locks-mutexes-and-semaphores.md) and [atomics and memory visibility](atomics-and-memory-visibility.md).
- Practice: the [thread-safe singleton](../problems/thread-safe-singleton.md) is a check-then-act race in its smallest form.
- Free answer: [What Is a Race Condition?](https://www.designgurus.io/answers/detail/what-is-race-conditions?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-race-conditions), a short definition with an example.
- Free article: [Thread Safety 101: Designing Code for Concurrency](https://www.designgurus.io/blog/thread-safety-concurrency?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-race-conditions), which covers the ways to make shared state safe.
