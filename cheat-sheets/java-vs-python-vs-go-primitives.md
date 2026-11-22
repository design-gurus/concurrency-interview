# Java vs Python vs Go: Concurrency Primitives

The three languages offer the same ideas under different names. This table translates between them, which helps when an interviewer allows any language or asks how a solution would look in another one.

## Starting work

| Need | Java | Python | Go |
|---|---|---|---|
| Start a thread | `new Thread(task).start()` | `threading.Thread(target=fn).start()` | `go fn()` |
| Wait for it to finish | `thread.join()` | `thread.join()` | `sync.WaitGroup` |
| Thread pool | `ExecutorService` | `ThreadPoolExecutor` | N goroutines reading one channel |
| Result of a task | `Future`, `CompletableFuture` | `concurrent.futures.Future` | A channel that carries the result |
| Lightweight tasks | Virtual threads | `asyncio` tasks | Goroutines |
| Process pool | Rarely needed | `ProcessPoolExecutor` | Rarely needed |

## Mutual exclusion

| Need | Java | Python | Go |
|---|---|---|---|
| Mutex | `synchronized`, `ReentrantLock` | `threading.Lock` | `sync.Mutex` |
| Reentrant mutex | `synchronized`, `ReentrantLock` | `threading.RLock` | None; restructure the code |
| Readers-writers lock | `ReentrantReadWriteLock` | None in the standard library | `sync.RWMutex` |
| Try to lock | `lock.tryLock()` | `lock.acquire(blocking=False)` | `mu.TryLock()` |
| Guaranteed release | `try` and `finally` | `with lock:` | `defer mu.Unlock()` |

## Waiting and signaling

| Need | Java | Python | Go |
|---|---|---|---|
| Condition variable | `wait` and `notifyAll`, or `Condition` | `threading.Condition` | `sync.Cond` |
| Counting semaphore | `Semaphore` | `threading.Semaphore` | A buffered channel |
| One-time gate | `CountDownLatch(1)` | `threading.Event` | A channel that is closed |
| Wait for N tasks | `CountDownLatch(n)` | Joining each thread, or `concurrent.futures.wait` | `sync.WaitGroup` |
| Run code exactly once | Holder class, or `enum` | A module, or a lock with a double check | `sync.Once` |
| Blocking queue | `ArrayBlockingQueue`, `LinkedBlockingQueue` | `queue.Queue` | A buffered channel |
| Wait on several events | `CompletableFuture.anyOf` | `asyncio.wait`, `concurrent.futures.wait` | `select` |
| Cancellation and timeouts | `Future.cancel`, interrupts | `asyncio` cancellation, timeouts on calls | `context.Context` |

## Atomics and safe collections

| Need | Java | Python | Go |
|---|---|---|---|
| Atomic counter | `AtomicInteger`, `LongAdder` | A lock around the counter | `atomic.Int64` |
| Visible flag | `volatile boolean` | `threading.Event` | `atomic.Bool` |
| Concurrent map | `ConcurrentHashMap` | A `dict` with a lock | `sync.Map`, or a map with a mutex |
| Concurrent set | `ConcurrentHashMap.newKeySet()` | A `set` with a lock | A map with a mutex |
| Thread-local value | `ThreadLocal` | `threading.local()` | None; pass values explicitly |

## Differences that cause bugs

| Topic | Java | Python | Go |
|---|---|---|---|
| Default lock is reentrant | Yes | No (`Lock`), yes (`RLock`) | No |
| Parallel threads on several cores | Yes | Not in the standard interpreter, because of the global interpreter lock | Yes |
| Exception in a thread | Ends that thread only | Ends that thread only | A panic in any goroutine ends the whole program |
| Unhandled failure in a pool task | Stored in the `Future` | Stored in the `Future` | Not applicable |
| Built-in race detector | No | No | Yes: `go test -race` |
| Sending on a closed channel | Not applicable | Not applicable | Panics |

## The same task in each language

Run 10 tasks and wait for all of them:

```java
ExecutorService pool = Executors.newFixedThreadPool(4);
List<Future<Integer>> futures = new ArrayList<>();
for (int i = 0; i < 10; i++) {
    final int n = i;
    futures.add(pool.submit(() -> n * n));
}
for (Future<Integer> f : futures) {
    f.get();
}
pool.shutdown();
```

```python
with ThreadPoolExecutor(max_workers=4) as pool:
    results = list(pool.map(lambda n: n * n, range(10)))
```

```go
var wg sync.WaitGroup
results := make([]int, 10)
for i := 0; i < 10; i++ {
	wg.Add(1)
	go func(n int) {
		defer wg.Done()
		results[n] = n * n
	}(i)
}
wg.Wait()
```

The Go version needs no lock, because each goroutine writes a different element of the slice.

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-java-vs-python-vs-go-primitives) teaches the primitives through interview problems.
- Lesson: [Choosing a Synchronization Construct](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/synchronization-constructs?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-java-vs-python-vs-go-primitives), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Conditions, Events, Semaphores, and Barriers](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/conditions-events-semaphores-and-barriers?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-java-vs-python-vs-go-primitives), in Grokking Concurrency in Python
- Lesson: [Channels or Mutexes](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/channels-or-mutexes?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-java-vs-python-vs-go-primitives), in Grokking Concurrency in Go
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-java-vs-python-vs-go-primitives) covers the Go column in depth.
- [Grokking Concurrency in Python](https://www.designgurus.io/course/grokking-concurrency-in-python?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-java-vs-python-vs-go-primitives) covers the Python column in depth.
- Related page: [locks, mutexes, and semaphores](../concepts/locks-mutexes-and-semaphores.md).
- Free answer: [Which Java API Is Used for Multithreading and Concurrency?](https://www.designgurus.io/answers/detail/which-java-api-is-used-for-multithreading-and-concurrency?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-java-vs-python-vs-go-primitives), a tour of the Java concurrency packages.
- Free answer: [How to Understand Concurrency Models in Programming Languages](https://www.designgurus.io/answers/detail/how-to-understand-concurrency-models-in-programming-languages?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-java-vs-python-vs-go-primitives), a comparison of the models behind these primitives.
