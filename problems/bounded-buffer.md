# Bounded Buffer (Producer and Consumer)

Build a queue with a fixed capacity that several threads share. A producer that calls `put` on a full queue must wait, and a consumer that calls `take` on an empty queue must wait.

This is the most common concurrency coding question, because it tests the one skill every other problem needs: waiting for a condition without wasting CPU time and without missing a wake-up.

## Key idea

Protect the queue with one lock, and attach two condition variables to that lock. A condition variable is an object that lets a thread release a lock and sleep until another thread signals it. One condition means "the queue is not full" and the other means "the queue is not empty".

Three rules make the solution correct:

1. Check the condition in a `while` loop, never in an `if`. A thread can wake up and find that another thread has already taken the item.
2. Change the queue and send the signal while holding the lock.
3. Signal the opposite condition: `put` signals "not empty", and `take` signals "not full".

| Operation | Waits while | Signals |
|---|---|---|
| `put` | The queue holds `capacity` items | Not empty |
| `take` | The queue holds 0 items | Not full |

## Java

```java
import java.util.ArrayDeque;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

class BoundedBuffer<T> {
    private final Queue<T> items = new ArrayDeque<>();
    private final int capacity;
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (items.size() == capacity) {
                notFull.await();
            }
            items.add(item);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    T take() throws InterruptedException {
        lock.lock();
        try {
            while (items.isEmpty()) {
                notEmpty.await();
            }
            T item = items.remove();
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

In production code, `java.util.concurrent.ArrayBlockingQueue` already does this. Say so in the interview, then write the version above.

## Python

```python
import threading
from collections import deque


class BoundedBuffer:
    def __init__(self, capacity):
        self._items = deque()
        self._capacity = capacity
        lock = threading.Lock()
        self._not_full = threading.Condition(lock)
        self._not_empty = threading.Condition(lock)

    def put(self, item):
        with self._not_full:
            while len(self._items) == self._capacity:
                self._not_full.wait()
            self._items.append(item)
            self._not_empty.notify()

    def take(self):
        with self._not_empty:
            while not self._items:
                self._not_empty.wait()
            item = self._items.popleft()
            self._not_full.notify()
            return item
```

Both conditions share one lock, so entering either `with` block locks the same queue. The standard library equivalent is `queue.Queue(maxsize=n)`.

## Go

```go
package main

import "sync"

type BoundedBuffer[T any] struct {
	mu       sync.Mutex
	notFull  *sync.Cond
	notEmpty *sync.Cond
	items    []T
	capacity int
}

func NewBoundedBuffer[T any](capacity int) *BoundedBuffer[T] {
	b := &BoundedBuffer[T]{capacity: capacity}
	b.notFull = sync.NewCond(&b.mu)
	b.notEmpty = sync.NewCond(&b.mu)
	return b
}

func (b *BoundedBuffer[T]) Put(item T) {
	b.mu.Lock()
	defer b.mu.Unlock()
	for len(b.items) == b.capacity {
		b.notFull.Wait()
	}
	b.items = append(b.items, item)
	b.notEmpty.Signal()
}

func (b *BoundedBuffer[T]) Take() T {
	b.mu.Lock()
	defer b.mu.Unlock()
	for len(b.items) == 0 {
		b.notEmpty.Wait()
	}
	item := b.items[0]
	b.items = b.items[1:]
	b.notFull.Signal()
	return item
}
```

A buffered channel, `make(chan T, capacity)`, is already a bounded buffer: a send blocks when it is full and a receive blocks when it is empty. Interviewers often accept the channel first and then ask for this version without channels.

## Follow-up questions

- Why is the condition checked in a loop instead of an `if` statement?
- What changes if `put` must give up after a timeout of 200 milliseconds?
- When is one condition variable with a broadcast enough, and what does it cost with 50 waiting threads?
- How would you shut the buffer down so that waiting consumers return instead of sleeping forever?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-bounded-buffer) covers this problem and its variants with step-by-step explanations.
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-bounded-buffer) explains channels, `select`, and when a mutex is the better choice.
- Related pages: [condition variables and monitors](../concepts/condition-variables-and-monitors.md) and the [readers-writers lock](readers-writers-lock.md).
