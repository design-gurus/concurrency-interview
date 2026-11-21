# Concurrency Interview

> The free, open companion to **Grokking Multithreading and Concurrency for Coding Interviews** by [DesignGurus.io](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme), created by Arslan Ahmad.

[![GitHub stars](https://img.shields.io/github/stars/design-gurus/concurrency-interview?style=social)](https://github.com/design-gurus/concurrency-interview/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/design-gurus/concurrency-interview)](https://github.com/design-gurus/concurrency-interview/commits/main)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](LICENSE)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

A concurrency interview does not test whether a candidate can start a thread. It tests whether the candidate can explain what happens when two threads reach the same data at the same moment, and prove that the code handles it. This repository is a short guide to that round: eight concept pages, nine classic problems solved in Java, Python, and Go, and three cheat sheets.

Every code solution here was compiled and run under a multi-threaded test, and the Go solutions also passed the Go race detector.

## Start here

| Your situation | Go to |
|---|---|
| The interview is tomorrow | The [question bank](cheat-sheets/question-bank.md), then [common mistakes](cheat-sheets/common-mistakes.md) |
| The topic is new to you | [Threads vs processes](concepts/threads-vs-processes.md), then the [concepts](concepts/README.md) in order |
| You know the theory and need coding practice | The [bounded buffer](problems/bounded-buffer.md), then the other [problems](problems/README.md) |
| You are switching interview language | [Java vs Python vs Go primitives](cheat-sheets/java-vs-python-vs-go-primitives.md) |
| You interview in Python and were asked about the GIL | [Async vs threads](concepts/async-vs-threads.md) |

## Concepts

| Page | What it covers |
|---|---|
| [Threads vs processes](concepts/threads-vs-processes.md) | Shared memory, concurrency against parallelism, context switches, lightweight threads |
| [Race conditions](concepts/race-conditions.md) | Lost updates, data races, check-then-act, and the five standard fixes |
| [Locks, mutexes, and semaphores](concepts/locks-mutexes-and-semaphores.md) | Which lock fits which problem, ownership, and reentrancy |
| [Deadlock, livelock, and starvation](concepts/deadlock-livelock-and-starvation.md) | The four Coffman conditions and how to remove each one |
| [Condition variables and monitors](concepts/condition-variables-and-monitors.md) | Waiting for a state, the `while` loop rule, and lost wake-ups |
| [Thread pools](concepts/thread-pools.md) | Pool sizing with a worked example, bounded queues, and rejection policies |
| [Atomics and memory visibility](concepts/atomics-and-memory-visibility.md) | `volatile`, compare-and-swap, happens-before, and the ABA problem |
| [Async vs threads](concepts/async-vs-threads.md) | Event loops, Python's global interpreter lock, goroutines, and virtual threads |

## Problems

Each page gives the key idea, a solution in Java, Python, and Go, and the follow-up questions interviewers ask.

| Problem | Difficulty | What it tests |
|---|---|---|
| [Print in order](problems/print-in-order.md) | Easy | Making one thread wait for another |
| [Thread-safe singleton](problems/thread-safe-singleton.md) | Easy | Safe publication and double-checked locking |
| [Odd-even printer](problems/odd-even-printer.md) | Easy | Passing a turn between two threads |
| [Bounded buffer](problems/bounded-buffer.md) | Medium | The producer and consumer pattern |
| [Dining philosophers](problems/dining-philosophers.md) | Medium | Preventing deadlock with lock ordering |
| [Token bucket rate limiter](problems/token-bucket-rate-limiter.md) | Medium | An atomic "check, then update" |
| [Thread-safe LRU cache](problems/thread-safe-lru-cache.md) | Medium | Seeing that a read is also a write |
| [Readers-writers lock](problems/readers-writers-lock.md) | Hard | Preventing writer starvation |
| [Concurrent web crawler](problems/concurrent-web-crawler.md) | Hard | Bounded workers and detecting completion |

## Cheat sheets

| Page | What it covers |
|---|---|
| [Question bank](cheat-sheets/question-bank.md) | 50 conceptual questions with one-line answers |
| [Java vs Python vs Go primitives](cheat-sheets/java-vs-python-vs-go-primitives.md) | A translation table for locks, latches, pools, and atomics |
| [Common mistakes](cheat-sheets/common-mistakes.md) | Ten errors in code and explanation, and three checks to run before saying "done" |

## What the round measures

| Signal | What a strong candidate does |
|---|---|
| Finds the shared state | Names every variable that more than one thread touches |
| Chooses the simplest correct tool | Starts with one mutex and refines only when asked |
| Reasons about interleavings | Describes a specific order of steps that would break an unprotected version |
| Handles waiting correctly | Waits in a loop, and never with a busy loop or a fixed sleep |
| Checks liveness | Explains why the code cannot deadlock and why every thread ends |

## FAQ

### What is asked in a concurrency interview?

The round usually has two parts. The first is conceptual: processes and threads, race conditions, deadlock, and the difference between a mutex and a semaphore. The second is a coding problem such as a bounded buffer, a rate limiter, or a thread-safe cache. The [question bank](cheat-sheets/question-bank.md) covers the first part, and the [problems](problems/README.md) cover the second.

### What are the most common multithreading interview questions?

Questions that appear in most rounds include the difference between a process and a thread, what a race condition is, the four conditions for deadlock, why `wait` is called in a loop, and what `volatile` does. Among coding problems, the producer and consumer problem (the [bounded buffer](problems/bounded-buffer.md)) is the standard one.

### Which language should I use for a concurrency interview?

Use the language you know best, unless the role names one. Java has the largest set of concurrency classes and the most interview material. Go solutions are often the shortest, because channels and goroutines are part of the language. Python is accepted for most problems, but you should be ready to explain the global interpreter lock.

### How do I solve the dining philosophers problem?

Number the forks and make every philosopher pick up the lower-numbered fork first. This rule is called lock ordering, and it prevents the cycle of waiting threads that a deadlock requires. The [dining philosophers](problems/dining-philosophers.md) page has the code in three languages and two alternative fixes.

### Does Python have real multithreading?

Python threads are real operating system threads, but the standard interpreter allows only one of them to run Python code at a time. Threads therefore help with work that waits on a network or a disk, and they do not speed up computation. [Async vs threads](concepts/async-vs-threads.md) explains the details and the alternatives.

### How long does it take to prepare?

A candidate who already writes multi-threaded code can cover this repository in about 6 to 8 hours: 2 hours for the concepts, 4 hours for the problems, and the rest for the cheat sheets. A candidate who is new to the topic should plan for 2 to 3 weeks with a structured course.

## Go deeper

This repository is a summary. The courses below teach the same material step by step, with many more problems and with explanations of every solution.

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the interview course for this round, from the fundamentals to the full set of classic problems.
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): goroutines, channels, `select`, the `sync` package, and concurrency patterns in Go.
- [Grokking Concurrency in Python](https://www.designgurus.io/course/grokking-concurrency-in-python?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): threads, the global interpreter lock, `multiprocessing`, and `asyncio`.
- [Grokking the Coding Interview](https://www.designgurus.io/course/grokking-the-coding-interview?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the coding patterns for the algorithm rounds of the same loop.

Related free repositories: [Grokking the Coding Interview patterns](https://github.com/design-gurus/grokking-coding-interview) and [Grokking System Design](https://github.com/design-gurus/grokking-system-design).

## Contributing

Corrections, clearer explanations, and solutions in other languages are welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

## License

The content is licensed under [CC BY 4.0](LICENSE). See [NOTICE.md](NOTICE.md) for what the license covers and how to attribute.
