# Async vs Threads

Threads let the operating system switch between tasks at any moment. Asynchronous code, usually called async, runs many tasks on one thread and switches only at points the program marks with `await`. The choice between them depends on whether the work waits or computes.

## Two kinds of work

| Kind | The task spends its time | Examples |
|---|---|---|
| I/O-bound | Waiting for a network, a disk, or a database | HTTP calls, queries, file reads |
| CPU-bound | Computing | Compression, parsing, image processing |

A waiting task needs no CPU core. A computing task needs one for its whole duration. Every model below is judged by how it treats these two cases.

## The models

| Model | Switching | Memory per task | Good for | Weak at |
|---|---|---|---|---|
| Operating system threads | Preemptive: at any instruction | About 1 MB of stack | CPU-bound work on several cores | Tens of thousands of waiting tasks |
| Event loop with `async` and `await` | Cooperative: only at `await` | A few kilobytes | Very many waiting tasks | CPU-bound work, which blocks every other task |
| Lightweight threads (goroutines, Java virtual threads) | By the language runtime | A few kilobytes | Both, with ordinary blocking code | Little, although a limit on concurrency is still needed |
| Processes | By the operating system | A full address space | CPU-bound work that needs isolation | Sharing data |

## How an event loop works

An event loop is one thread that runs a task until the task reaches an `await`, parks it, and runs another task that is ready. When the awaited operation completes, the parked task becomes ready again.

```python
import asyncio

async def fetch(n):
    await asyncio.sleep(1)      # stands for a 1 second network call
    return n

async def main():
    results = await asyncio.gather(*(fetch(n) for n in range(100)))
    print(len(results))

asyncio.run(main())
```

The 100 calls finish in about 1 second in total, on one thread, because all of them wait at the same time.

The rule that follows is strict: **never block the event loop**. One call to `time.sleep(1)` or one 500 millisecond computation inside a coroutine stops every other task for that time. Blocking work belongs in a thread pool or a process pool.

## Python's global interpreter lock

The standard Python interpreter has a global interpreter lock (GIL): a mutex that allows only one thread at a time to execute Python bytecode. The consequences are specific.

- Threads do not speed up CPU-bound Python code. Four threads on four cores compute about as fast as one.
- Threads do help I/O-bound code, because a thread releases the GIL while it waits.
- Many numerical libraries release the GIL inside their compiled code, so they can use several cores.
- For CPU-bound work in pure Python, the usual answer is processes, through `multiprocessing` or `ProcessPoolExecutor`.

Newer Python releases offer an optional build without the GIL, called free-threaded Python. It allows real parallel threads, and it is not yet the default.

The GIL does not make Python code thread-safe. A thread can be switched out between any two bytecode instructions, so compound operations such as `count += 1` still need a lock.

## Choosing in each language

| Language | Many waiting tasks | CPU-bound work |
|---|---|---|
| Python | `asyncio`, or a thread pool for libraries that block | Processes |
| Java | Virtual threads, or a thread pool | Platform threads, one per core |
| Go | Goroutines | Goroutines; the runtime uses every core |

## Async does not remove races

Code between two `await` points runs without interruption, so many data races disappear. Race conditions remain. If a coroutine reads a balance, awaits a network call, and then writes the balance, another coroutine can change it during the await. `asyncio.Lock` exists for that reason.

## How this is asked

- "When would you use async instead of threads?"
- "What is the GIL, and how does it affect multithreading in Python?"
- "What happens if you call a blocking function inside a coroutine?"
- "What is the difference between a goroutine and a thread?"
- "Is async code free of race conditions?"

## Go deeper

- [Grokking Concurrency in Python](https://www.designgurus.io/course/grokking-concurrency-in-python?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-async-vs-threads) covers threads, the GIL, `multiprocessing`, and `asyncio` in depth.
- Lesson: [IO Bound Work, Asyncio or Threads](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/io-bound-work-asyncio-or-threads?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-async-vs-threads), in Grokking Concurrency in Python
- Lesson: [The Four Models and How to Size Them](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/the-four-models-and-how-to-size-them?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-async-vs-threads), in Grokking Concurrency in Python
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-async-vs-threads) covers goroutines, channels, and the scheduler.
- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-async-vs-threads) covers the thread-based interview problems.
- Related pages: [threads versus processes](threads-vs-processes.md) and [thread pools](thread-pools.md).
- Free article: [Asyncio vs Threading vs Multiprocessing in Python, With Measurements](https://www.designgurus.io/blog/asyncio-vs-threading-vs-multiprocessing?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-async-vs-threads), the same choice with measured results in Python.
- Free article: [The Python GIL Explained, and What Free-Threaded Python Changes](https://www.designgurus.io/blog/python-gil-explained?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-async-vs-threads), which explains why Python threads do not run CPU work in parallel.
