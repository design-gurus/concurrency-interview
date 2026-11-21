# Dining Philosophers

Five philosophers sit at a round table with one fork between each pair of neighbors, so there are five forks. A philosopher needs both adjacent forks to eat, and the program must never reach a state where every philosopher holds one fork and waits for the other.

The problem is a model of any system in which threads need two or more locks at once, such as a transfer between two bank accounts.

## Key idea

If every philosopher picks up the left fork first, all five can hold one fork and wait forever. That is a deadlock: a state in which each thread waits for a lock that another waiting thread holds.

The simplest fix is lock ordering. Number the forks from 0 to 4, and make every philosopher pick up the lower-numbered fork first. Philosopher 4 sits between fork 4 and fork 0, so that philosopher reaches for fork 0 first while the other four reach for their left fork. A cycle of waiting threads can no longer form, because every thread acquires locks in the same global order.

| Fix | How it prevents deadlock | Cost |
|---|---|---|
| Lock ordering | No cycle of waiting can form | None at run time |
| A semaphore that admits 4 of the 5 | At least one philosopher can always take both forks | One extra primitive |
| Try the second fork, release the first on failure | No thread holds one lock while blocked on another | Risk of livelock, where threads keep retrying without progress |

## Java

```java
import java.util.concurrent.locks.ReentrantLock;

class DiningPhilosophers {
    private final ReentrantLock[] forks;

    DiningPhilosophers(int count) {
        forks = new ReentrantLock[count];
        for (int i = 0; i < count; i++) {
            forks[i] = new ReentrantLock();
        }
    }

    void dine(int philosopher, Runnable eat) {
        int left = philosopher;
        int right = (philosopher + 1) % forks.length;
        int first = Math.min(left, right);
        int second = Math.max(left, right);
        forks[first].lock();
        try {
            forks[second].lock();
            try {
                eat.run();
            } finally {
                forks[second].unlock();
            }
        } finally {
            forks[first].unlock();
        }
    }
}
```

## Python

```python
import threading


class DiningPhilosophers:
    def __init__(self, count):
        self._forks = [threading.Lock() for _ in range(count)]

    def dine(self, philosopher, eat):
        left = philosopher
        right = (philosopher + 1) % len(self._forks)
        first, second = min(left, right), max(left, right)
        with self._forks[first]:
            with self._forks[second]:
                eat()
```

## Go

```go
package main

import "sync"

type DiningPhilosophers struct {
	forks []sync.Mutex
}

func NewDiningPhilosophers(count int) *DiningPhilosophers {
	return &DiningPhilosophers{forks: make([]sync.Mutex, count)}
}

func (d *DiningPhilosophers) Dine(philosopher int, eat func()) {
	first := philosopher
	second := (philosopher + 1) % len(d.forks)
	if first > second {
		first, second = second, first
	}
	d.forks[first].Lock()
	defer d.forks[first].Unlock()
	d.forks[second].Lock()
	defer d.forks[second].Unlock()
	eat()
}
```

## Follow-up questions

- Which of the four deadlock conditions does lock ordering remove? (Circular wait.)
- Lock ordering prevents deadlock, but can one philosopher still wait much longer than the others? How would you bound that wait?
- How does the same idea apply to `transfer(from, to)` when two threads transfer in opposite directions between the same two accounts?
- What changes if a philosopher needs three resources instead of two?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-dining-philosophers) covers the semaphore and try-lock versions with full explanations.
- Related pages: [deadlock, livelock, and starvation](../concepts/deadlock-livelock-and-starvation.md) and [locks, mutexes, and semaphores](../concepts/locks-mutexes-and-semaphores.md).
- Free article: [Deadlock vs Livelock: Key Differences and How to Prevent Both](https://www.designgurus.io/blog/deadlock-vs-livelock?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-dining-philosophers), the two failures that this problem demonstrates.
- Free answer: [Types of Deadlock in an Operating System](https://www.designgurus.io/answers/detail/what-are-the-4-types-of-deadlock?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-dining-philosophers), background on how deadlocks are classified.
