# Atomics and Memory Visibility

Two separate problems hide behind the phrase "thread-safe". Atomicity asks whether an operation can be interrupted halfway. Visibility asks whether a write by one thread is ever seen by another. A lock solves both, and the lighter tools on this page solve one or both for a single variable.

## Visibility

Modern CPUs keep values in registers and per-core caches, and compilers reorder instructions. As a result, a write by one thread is not guaranteed to reach another thread unless the program asks for that.

```java
class Worker {
    private boolean stopped = false;

    void run() {
        while (!stopped) {
            doWork();
        }
    }

    void stop() {
        stopped = true;
    }
}
```

The loop may never end. The compiler is allowed to read `stopped` once and reuse the result, because nothing tells it that another thread writes the field. Declaring the field `volatile` fixes this program.

## What `volatile` does and does not do

| Guarantee | `volatile` | Lock | Atomic variable |
|---|---|---|---|
| A write is visible to later reads in other threads | Yes | Yes | Yes |
| Read, change, and write happen as one step | No | Yes | Yes |
| Several variables change together | No | Yes | No |

`volatile` is enough when one thread writes and the others only read, as with a stop flag. It is not enough for `count++`, which reads and writes.

## Atomic variables and compare-and-swap

An atomic variable performs a read, a change, and a write as one indivisible operation. It is built on a CPU instruction called compare-and-swap (CAS), which says: "set this location to the new value only if it still holds the value I expect, and tell me whether that worked".

```java
AtomicInteger count = new AtomicInteger();

int next() {
    while (true) {
        int current = count.get();
        if (count.compareAndSet(current, current + 1)) {
            return current + 1;
        }
    }
}
```

If another thread changes the value between the read and the CAS, the CAS fails and the loop retries. No thread ever blocks, which is why such code is called lock-free. In practice, `count.incrementAndGet()` does all of this in one call.

| Language | Atomic counter | Visibility for a flag |
|---|---|---|
| Java | `AtomicInteger`, `AtomicLong`, `LongAdder` | `volatile` |
| Go | `atomic.Int64` from `sync/atomic` | `atomic.Bool` |
| Python | No atomic type; use a `threading.Lock` | A plain attribute is visible, but compound updates still need a lock |

## Happens-before

Language memory models define visibility with a relation called happens-before. If action A happens-before action B, then B is guaranteed to see the effects of A. The rules that matter in interviews are short.

- Releasing a lock happens-before the next acquisition of the same lock.
- A write to a `volatile` field happens-before every later read of it.
- Starting a thread happens-before anything that thread does.
- A send on a Go channel happens-before the matching receive completes.

Without one of these links between a write and a read, the program has a data race, and its behavior is not defined by the language.

## The ABA problem

A CAS checks only that the value is the same, not that nothing happened. A value can change from A to B and back to A, and the CAS still succeeds. This matters for lock-free structures that reuse nodes. The usual fix pairs the value with a version number, as Java's `AtomicStampedReference` does.

## When to use which

| Need | Tool |
|---|---|
| A stop flag written by one thread | `volatile` or an atomic boolean |
| One counter | An atomic variable |
| A counter updated by dozens of threads | `LongAdder` in Java, which spreads updates across cells |
| Two or more fields that must agree | A lock |

## How this is asked

- "What does `volatile` do? Does it make `i++` safe?"
- "What is compare-and-swap?"
- "What is the difference between `synchronized` and `AtomicInteger`?"
- "Why is double-checked locking broken without `volatile`?" See the [thread-safe singleton](../problems/thread-safe-singleton.md).

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-atomics-and-memory-visibility) covers the memory model, atomics, and lock-free techniques with worked problems.
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-atomics-and-memory-visibility) covers `sync/atomic` and the Go memory model.
- Related page: [race conditions](race-conditions.md).
