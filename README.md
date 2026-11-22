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

## Course lessons to start with

- [Race Conditions and Critical Sections](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/critical-section-and-race-condition?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): how a race happens and which code needs protection, in Grokking Multithreading and Concurrency for Coding Interviews
- [Mutex Lock](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/1-mutex-lock?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the basic lock that most interview solutions start from, in Grokking Multithreading and Concurrency for Coding Interviews
- [Condition Variables](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/4-condition-variables?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): how a thread waits for a condition and is woken, in Grokking Multithreading and Concurrency for Coding Interviews
- [Deadlock, Starvation, and Livelock](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/deadlock-starvation-and-livelock?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the three ways threads stop making progress, in Grokking Multithreading and Concurrency for Coding Interviews
- [Thread Pools and Executors](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/thread-pools-and-executors?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): how a fixed set of workers runs many tasks, in Grokking Multithreading and Concurrency for Coding Interviews
- [Problem 15: The Dining Philosophers](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/problem-15-the-dining-philosophers?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the classic deadlock problem solved in full, in Grokking Multithreading and Concurrency for Coding Interviews
- [Channels or Mutexes](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/channels-or-mutexes?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): how to choose between the two Go approaches, in Grokking Concurrency in Go
- [Rate Limiting with a Token Bucket](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/rate-limiting-with-a-token-bucket?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): a token bucket limiter built in Go, in Grokking Concurrency in Go
- [The Four Models and How to Size Them](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/the-four-models-and-how-to-size-them?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): threads, processes, asyncio, and interpreters compared, in Grokking Concurrency in Python
- [Producer and Consumer With a Queue](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/producer-and-consumer-with-a-queue?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the bounded buffer pattern in Python, in Grokking Concurrency in Python

## Free reading on DesignGurus.io

- [Thread Safety 101: Designing Code for Concurrency](https://www.designgurus.io/blog/thread-safety-concurrency?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the design habits that keep shared state safe.
- [Concurrency vs Parallelism: Why They're Not the Same Thing](https://www.designgurus.io/blog/concurrency-vs-parallelism?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the distinction that most concurrency rounds start with.
- [Deadlock vs Livelock: Key Differences and How to Prevent Both](https://www.designgurus.io/blog/deadlock-vs-livelock?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): a comparison of the two failures and their fixes.
- [Asyncio vs Threading vs Multiprocessing in Python, With Measurements](https://www.designgurus.io/blog/asyncio-vs-threading-vs-multiprocessing?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the three Python models compared with measured results.
- [The Python GIL Explained, and What Free-Threaded Python Changes](https://www.designgurus.io/blog/python-gil-explained?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): why Python threads do not run CPU work in parallel.
- [20 Concurrency and Parallelism Interview Questions (Answered)](https://www.designgurus.io/answers/detail/what-are-some-concurrency-and-parallelism-interview-questions?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): more questions with model answers.
- [Top Concurrency and Multithreading Considerations for System Design Interviews](https://www.designgurus.io/answers/detail/top-concurrency-and-multithreading-for-system-design-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): how the same ideas appear in the design round.

## Go deeper

This repository is a summary. The courses below teach the same material step by step, with many more problems and with explanations of every solution.

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the interview course for this round, from the fundamentals to the full set of classic problems.
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): goroutines, channels, `select`, the `sync` package, and concurrency patterns in Go.
- [Grokking Concurrency in Python](https://www.designgurus.io/course/grokking-concurrency-in-python?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): threads, the global interpreter lock, `multiprocessing`, and `asyncio`.
- [Grokking the Coding Interview](https://www.designgurus.io/course/grokking-the-coding-interview?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=readme): the coding patterns for the algorithm rounds of the same loop.

Related free repositories: [Grokking the Coding Interview patterns](https://github.com/design-gurus/grokking-coding-interview) and [Grokking System Design](https://github.com/design-gurus/grokking-system-design).

For the system design round, see [Grokking System Design](https://github.com/design-gurus/grokking-system-design), the free companion guide with 30 patterns, 59 question walkthroughs, and 35 cheat sheets.

## More free guides

Every guide below is free, follows the same short-page format, and comes from DesignGurus.io.

- **Core guides:** [Grokking System Design](https://github.com/design-gurus/grokking-system-design), [Grokking Coding Interview](https://github.com/design-gurus/grokking-coding-interview), [Grokking AI System Design](https://github.com/design-gurus/grokking-ai-system-design), [Design Patterns](https://github.com/design-gurus/design-patterns), [Best System Design Resources](https://github.com/design-gurus/best-system-design-resources), [Best System Design Courses](https://github.com/design-gurus/best-system-design-courses)
- **By interview round:** [System Design Numbers](https://github.com/design-gurus/system-design-numbers), [API Design Interview](https://github.com/design-gurus/api-design-interview), [Frontend System Design Interview](https://github.com/design-gurus/frontend-system-design-interview), [Mobile System Design Interview](https://github.com/design-gurus/mobile-system-design-interview), [Microservices Interview Questions](https://github.com/design-gurus/microservices-interview-questions)
- **By role:** [DevOps and SRE Interview](https://github.com/design-gurus/devops-sre-interview), [Data Engineering Interview](https://github.com/design-gurus/data-engineering-interview), [Forward Deployed Engineer Interview](https://github.com/design-gurus/forward-deployed-engineer-interview), [Solutions Architect Interview](https://github.com/design-gurus/solutions-architect-interview)
- **Seniority and behavioral:** [Staff Engineer Interview](https://github.com/design-gurus/staff-engineer-interview), [Engineering Manager Interview](https://github.com/design-gurus/engineering-manager-interview), [Amazon Leadership Principles](https://github.com/design-gurus/amazon-leadership-principles), [Software Engineer Levels](https://github.com/design-gurus/software-engineer-levels)
- **By company:** [Google Interview Guide](https://github.com/design-gurus/google-interview-guide), [Meta Interview Guide](https://github.com/design-gurus/meta-interview-guide), [Amazon Interview Guide](https://github.com/design-gurus/amazon-interview-guide), [Microsoft Interview Guide](https://github.com/design-gurus/microsoft-interview-guide), [Apple Interview Guide](https://github.com/design-gurus/apple-interview-guide), [Netflix Interview Guide](https://github.com/design-gurus/netflix-interview-guide), [OpenAI Interview Guide](https://github.com/design-gurus/openai-interview-guide), [Anthropic Interview Guide](https://github.com/design-gurus/anthropic-interview-guide)

## Contributing

Corrections, clearer explanations, and solutions in other languages are welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request.

## License

The content is licensed under [CC BY 4.0](LICENSE). See [NOTICE.md](NOTICE.md) for what the license covers and how to attribute.
