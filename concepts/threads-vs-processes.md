# Threads vs Processes

A process is a running program with its own private memory. A thread is a path of execution inside a process, and all threads of one process share that memory. Almost every other concurrency question follows from this one difference.

## The comparison

| Property | Process | Thread |
|---|---|---|
| Memory | Private address space | Shared with the other threads of the process |
| Cost to create | Higher: the operating system builds a new address space | Lower: a new stack, about 1 MB by default on many systems |
| Communication | Pipes, sockets, shared memory segments | Ordinary variables and objects |
| Failure | A crash ends one process | A crash usually ends every thread in the process |
| Main risk | Slow, explicit communication | Two threads changing the same data at once |

Shared memory makes threads fast to coordinate and easy to get wrong. Private memory makes processes safer and more expensive to connect.

## Concurrency and parallelism

These two words are often confused, and interviewers ask for the difference.

- **Concurrency** means that several tasks are in progress during the same period. One CPU core can provide it by switching between tasks.
- **Parallelism** means that several tasks execute at the same instant. It requires more than one core.

A web server on one core that handles 1,000 open connections is concurrent but not parallel. A program that splits a matrix across 8 cores is parallel.

## Context switches

A context switch is the act of saving one thread's state and loading another's. It typically costs a few microseconds, plus the slower effect of cold CPU caches afterward. A program with 5,000 threads on 8 cores can spend more time switching than working, which is one reason [thread pools](thread-pools.md) exist.

## Lightweight threads

Several languages add a third option that the language runtime schedules instead of the operating system.

| Language | Name | Starting stack | Scheduled by |
|---|---|---|---|
| Go | Goroutine | A few kilobytes, and it grows on demand | The Go runtime, onto a small set of operating system threads |
| Java | Virtual thread | Small and stored on the heap | The Java runtime |
| Python | Coroutine (`asyncio` task) | No separate stack | An event loop on one thread |

A program can run 100,000 goroutines or virtual threads on a laptop. The same number of operating system threads would exhaust memory.

## When to choose which

| Situation | Usual choice | Reason |
|---|---|---|
| CPU-bound work in Java or Go | Threads or goroutines | Shared memory and real parallelism |
| CPU-bound work in Python | Processes | The global interpreter lock limits threads to one core |
| Thousands of slow network calls | Lightweight threads or an event loop | Waiting tasks cost almost nothing |
| Untrusted or crash-prone code | Processes | A failure stays inside one process |

The Python row is explained in [async versus threads](async-vs-threads.md).

## How this is asked

- "What is the difference between a process and a thread?"
- "What do threads of the same process share, and what does each thread own?" (They share the heap, global variables, and open files. Each owns its stack, its program counter, and its registers.)
- "Why is creating a thread cheaper than creating a process?"
- "What is a context switch, and why is it expensive?"

A strong answer names shared memory first, and then states the consequence: threads need synchronization, and processes need a channel for communication.

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-threads-vs-processes) starts from these fundamentals and builds up to full interview problems.
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-threads-vs-processes) explains goroutines and the Go scheduler.
- Next page: [race conditions](race-conditions.md).
