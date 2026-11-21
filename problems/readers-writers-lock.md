# Readers-Writers Lock

Build a lock that allows any number of readers at the same time but only one writer, and never a reader and a writer together. The lock must also keep a steady flow of readers from blocking a writer forever.

The first requirement is easy. The second one, which is called writer starvation, is what the interviewer is checking.

## Key idea

Keep three pieces of state behind one mutex: the number of active readers, whether a writer is active, and the number of writers that are waiting. A new reader waits when a writer is active or when any writer is waiting. That second check is the whole fix for starvation, because arriving readers stop joining once a writer has asked for the lock.

| Caller | May enter when | On exit |
|---|---|---|
| Reader | No writer is active and no writer is waiting | Wakes waiters if it was the last reader |
| Writer | No writer is active and 0 readers are active | Wakes all waiters |

This policy is called writer preference. Its cost is the opposite risk: a steady flow of writers can now delay readers. State that trade-off before the interviewer asks.

## Java

```java
class ReadWriteLock {
    private int readers = 0;
    private int waitingWriters = 0;
    private boolean writing = false;

    synchronized void lockRead() throws InterruptedException {
        while (writing || waitingWriters > 0) {
            wait();
        }
        readers++;
    }

    synchronized void unlockRead() {
        readers--;
        if (readers == 0) {
            notifyAll();
        }
    }

    synchronized void lockWrite() throws InterruptedException {
        waitingWriters++;
        try {
            while (writing || readers > 0) {
                wait();
            }
            writing = true;
        } finally {
            waitingWriters--;
        }
    }

    synchronized void unlockWrite() {
        writing = false;
        notifyAll();
    }
}
```

The production class is `java.util.concurrent.locks.ReentrantReadWriteLock`, which also offers a fair mode.

## Python

```python
import threading


class ReadWriteLock:
    def __init__(self):
        self._cond = threading.Condition()
        self._readers = 0
        self._waiting_writers = 0
        self._writing = False

    def lock_read(self):
        with self._cond:
            while self._writing or self._waiting_writers > 0:
                self._cond.wait()
            self._readers += 1

    def unlock_read(self):
        with self._cond:
            self._readers -= 1
            if self._readers == 0:
                self._cond.notify_all()

    def lock_write(self):
        with self._cond:
            self._waiting_writers += 1
            while self._writing or self._readers > 0:
                self._cond.wait()
            self._waiting_writers -= 1
            self._writing = True

    def unlock_write(self):
        with self._cond:
            self._writing = False
            self._cond.notify_all()
```

The Python standard library has no readers-writers lock, so Python candidates build one by hand.

## Go

```go
package main

import "sync"

type ReadWriteLock struct {
	mu             sync.Mutex
	cond           *sync.Cond
	readers        int
	waitingWriters int
	writing        bool
}

func NewReadWriteLock() *ReadWriteLock {
	l := &ReadWriteLock{}
	l.cond = sync.NewCond(&l.mu)
	return l
}

func (l *ReadWriteLock) RLock() {
	l.mu.Lock()
	defer l.mu.Unlock()
	for l.writing || l.waitingWriters > 0 {
		l.cond.Wait()
	}
	l.readers++
}

func (l *ReadWriteLock) RUnlock() {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.readers--
	if l.readers == 0 {
		l.cond.Broadcast()
	}
}

func (l *ReadWriteLock) Lock() {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.waitingWriters++
	for l.writing || l.readers > 0 {
		l.cond.Wait()
	}
	l.waitingWriters--
	l.writing = true
}

func (l *ReadWriteLock) Unlock() {
	l.mu.Lock()
	defer l.mu.Unlock()
	l.writing = false
	l.cond.Broadcast()
}
```

`sync.RWMutex` is the production answer in Go, and it already blocks new readers once a writer is waiting.

## Follow-up questions

- When is a readers-writers lock slower than a plain mutex? (When the critical section is short, because the extra bookkeeping costs more than the reads it allows in parallel.)
- How would you make the lock fair to both sides, so that neither readers nor writers can be delayed without limit?
- Can a reader upgrade to a writer without releasing the lock, and why do two upgrading readers deadlock?
- Why does every exit use a broadcast instead of a single signal?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-readers-writers-lock) works through reader preference, writer preference, and fair versions.
- [Grokking Concurrency in Python](https://www.designgurus.io/course/grokking-concurrency-in-python?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-readers-writers-lock) covers `threading.Condition` and the patterns built on it.
- Related pages: [deadlock, livelock, and starvation](../concepts/deadlock-livelock-and-starvation.md) and [locks, mutexes, and semaphores](../concepts/locks-mutexes-and-semaphores.md).
