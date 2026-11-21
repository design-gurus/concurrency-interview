# Problems

Nine classic concurrency interview problems, each solved in Java, Python, and Go. Every solution on these pages was compiled and run under a multi-threaded test before publication, and the Go solutions also passed the Go race detector.

| Problem | Difficulty | Main primitive | What it tests |
|---|---|---|---|
| [Print in order](print-in-order.md) | Easy | Latch, event, or closed channel | Making one thread wait for another |
| [Thread-safe singleton](thread-safe-singleton.md) | Easy | Double-checked locking, `sync.Once` | Safe publication and memory visibility |
| [Odd-even printer](odd-even-printer.md) | Easy | Condition variable, or two channels | Passing a turn between threads |
| [Bounded buffer](bounded-buffer.md) | Medium | Mutex with two condition variables | Waiting for a state without a busy loop |
| [Dining philosophers](dining-philosophers.md) | Medium | Lock ordering | Preventing deadlock |
| [Token bucket rate limiter](token-bucket-rate-limiter.md) | Medium | Mutex and a monotonic clock | Making "check, then update" atomic |
| [Thread-safe LRU cache](thread-safe-lru-cache.md) | Medium | Mutex around a map and a list | Seeing that a read is also a write |
| [Readers-writers lock](readers-writers-lock.md) | Hard | Condition variable with a waiting count | Preventing starvation |
| [Concurrent web crawler](concurrent-web-crawler.md) | Hard | Thread pool, concurrent set, pending count | Bounding work and detecting completion |

## How to practice

1. Read the problem statement and the "Key idea" section, then close the page.
2. Write the solution in your interview language within 20 minutes.
3. Explain aloud one interleaving of two threads that would break a version without the lock.
4. Compare with the solution, then answer the follow-up questions without notes.

Step 3 is the one most candidates skip, and it is the part of the answer that an interviewer weighs most.

## Go deeper

These nine problems are a sample. [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-readme) covers the full set of interview problems with step-by-step explanations in several languages.
