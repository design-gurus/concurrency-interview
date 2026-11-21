# Print in Order

Three threads call `first`, `second`, and `third` on the same object, one method each, and the scheduler may start them in any order. The output must always be first, then second, then third.

The question tests whether a candidate can make one thread wait for an event in another thread without a busy loop.

## Key idea

Each method after the first needs a signal that the previous one has finished. The signal must also be remembered: if `first` finishes before `second` starts waiting, `second` must still pass.

A one-time gate has exactly this behavior. It starts closed, opens once, and stays open. Every language has one.

| Language | One-time gate | Wait | Open |
|---|---|---|---|
| Java | `CountDownLatch(1)` | `await()` | `countDown()` |
| Python | `threading.Event` | `wait()` | `set()` |
| Go | `chan struct{}` | receive with `<-ch` | `close(ch)` |

Two gates are enough: one opened by `first`, and one opened by `second`.

## Java

```java
import java.util.concurrent.CountDownLatch;

class OrderedPrinter {
    private final CountDownLatch firstDone = new CountDownLatch(1);
    private final CountDownLatch secondDone = new CountDownLatch(1);

    void first(Runnable printFirst) {
        printFirst.run();
        firstDone.countDown();
    }

    void second(Runnable printSecond) throws InterruptedException {
        firstDone.await();
        printSecond.run();
        secondDone.countDown();
    }

    void third(Runnable printThird) throws InterruptedException {
        secondDone.await();
        printThird.run();
    }
}
```

## Python

```python
import threading


class OrderedPrinter:
    def __init__(self):
        self._first_done = threading.Event()
        self._second_done = threading.Event()

    def first(self, print_first):
        print_first()
        self._first_done.set()

    def second(self, print_second):
        self._first_done.wait()
        print_second()
        self._second_done.set()

    def third(self, print_third):
        self._second_done.wait()
        print_third()
```

## Go

```go
package main

type OrderedPrinter struct {
	firstDone  chan struct{}
	secondDone chan struct{}
}

func NewOrderedPrinter() *OrderedPrinter {
	return &OrderedPrinter{
		firstDone:  make(chan struct{}),
		secondDone: make(chan struct{}),
	}
}

func (p *OrderedPrinter) First(printFirst func()) {
	printFirst()
	close(p.firstDone)
}

func (p *OrderedPrinter) Second(printSecond func()) {
	<-p.firstDone
	printSecond()
	close(p.secondDone)
}

func (p *OrderedPrinter) Third(printThird func()) {
	<-p.secondDone
	printThird()
}
```

A receive on a closed channel returns immediately, which is what makes a closed channel behave as an open gate.

## Follow-up questions

- Why does a plain condition variable fail here if `first` signals before `second` begins to wait? (The signal is lost, because a condition variable keeps no memory of past signals.)
- How would you solve it with two semaphores that start at 0?
- How does the design change for N methods that must run in order?
- Why is a loop that repeatedly checks a shared flag a poor answer, even when the flag is declared `volatile`?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-print-in-order) covers ordering problems and the signaling primitives behind them.
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-print-in-order) explains closed channels, `select`, and `sync.WaitGroup`.
- Related pages: the [odd-even printer](odd-even-printer.md), which repeats the hand-off many times, and [condition variables and monitors](../concepts/condition-variables-and-monitors.md).
