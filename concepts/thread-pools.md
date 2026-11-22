# Thread Pools

A thread pool is a fixed set of worker threads that take tasks from a shared queue. It exists because creating a thread for every task is slow, and because an unlimited number of threads can exhaust memory.

## The parts

| Part | Role | Decision to make |
|---|---|---|
| Worker threads | Run the tasks | How many |
| Task queue | Holds tasks that no worker has taken yet | Bounded or unbounded |
| Rejection policy | Decides what happens when the queue is full | Block, drop, fail, or run in the caller |
| Shutdown | Stops the workers | Finish queued tasks or discard them |

A pool is a [bounded buffer](../problems/bounded-buffer.md) of tasks with consumers that never stop.

## Sizing a pool

The correct size depends on what the tasks wait for.

| Workload | Starting size | Reason |
|---|---|---|
| CPU-bound (hashing, image resizing) | Number of cores | More threads only add context switches |
| I/O-bound (database calls, HTTP calls) | Cores x (1 + wait time / compute time) | Threads that wait do not use a core |

Worked example: a server has 8 cores. Each task computes for 10 milliseconds and waits 90 milliseconds for a database. The formula gives 8 x (1 + 90 / 10) = 80 threads. For the CPU-bound case, the answer is 8.

The formula is a starting point. The real limit is often a downstream resource: 80 threads are of no use if the database accepts 20 connections.

## The queue

An unbounded queue is the most common production mistake with pools. When tasks arrive faster than workers finish them, the queue grows until the process runs out of memory, and every task in it is already late. A bounded queue forces a decision at the moment of overload.

| Rejection policy | Effect | Fits |
|---|---|---|
| Block the submitter | Slows the producer down | Internal pipelines |
| Fail fast with an error | The caller can retry or report | Request handlers |
| Drop the task | Work is lost | Metrics, logging |
| Run in the caller's thread | Slows the submitter naturally | Simple back pressure |

Back pressure is the general name for a slow consumer making its producer slow down.

## Pools in each language

| Language | Standard pool | Note |
|---|---|---|
| Java | `ExecutorService`, created by `Executors.newFixedThreadPool(n)` | The fixed pool uses an unbounded queue. Build a `ThreadPoolExecutor` directly to bound it |
| Python | `concurrent.futures.ThreadPoolExecutor(max_workers=n)` | Use `ProcessPoolExecutor` for CPU-bound work |
| Go | No pool type | Start N goroutines that read from one channel, or limit goroutines with a buffered channel |

Go rarely needs a pool for cost reasons, because goroutines are cheap. It still needs a limit, because the resources behind them are not.

## Results and failures

Submitting a task returns a future: an object that will hold the result later. Two points come up in interviews.

- An exception inside a task does not appear in the submitting thread. It is stored in the future and raised when the result is requested. A task whose future is never read can fail silently.
- A task that blocks forever occupies a worker forever. Every blocking call inside a pool needs a timeout.

## How this is asked

- "Why use a thread pool instead of creating threads?"
- "How many threads should a pool have?"
- "What happens when the queue is full?"
- "Design a thread pool." The expected answer is a bounded queue, N workers in a loop, a shutdown flag, and a rejection policy.
- "What is the difference between `submit` and `execute` in Java?" (`submit` returns a future and `execute` does not.)

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-thread-pools) covers executors, futures, and pool design as interview problems.
- Lesson: [Thread Pools and Executors](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/thread-pools-and-executors?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-thread-pools), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Worker Pools](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/worker-pools?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-thread-pools), in Grokking Concurrency in Go
- Lesson: [Build a Thread Pool](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/build-a-thread-pool?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-thread-pools), in Grokking Concurrency in Python
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-thread-pools) covers worker pools and pipelines built from channels.
- Practice: the [concurrent web crawler](../problems/concurrent-web-crawler.md) uses a pool in all three languages.
- Free answer: [What Is Connection Pooling for Databases or Threads?](https://www.designgurus.io/answers/detail/what-is-connection-pooling-for-databases-or-threads-and-how-does-it-improve-system-performance-and-resource-utilization?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-thread-pools), the pooling idea at the system level.
