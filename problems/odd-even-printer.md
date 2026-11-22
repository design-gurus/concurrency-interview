# Odd-Even Printer

Two threads share a counter. One thread prints the odd numbers and the other prints the even numbers, and together they must print 1 to N in order.

This is the repeated form of [print in order](print-in-order.md). A one-time gate is not enough, because the turn passes back and forth N times.

## Key idea

Keep the next number to print in shared state behind a lock. Each thread waits until the next number has its parity, prints it, advances the counter, and wakes the other thread.

Two details decide whether the program ends cleanly:

1. The wait condition must include the limit. A thread that only waits for its turn sleeps forever after the last number.
2. The thread that prints the last number must still wake the other one, so that it can observe the limit and return.

| Step | Counter | Odd thread | Even thread |
|---|---|---|---|
| 1 | 1 | Prints 1, wakes the other | Waiting |
| 2 | 2 | Waiting | Prints 2, wakes the other |
| 3 | 3 | Prints 3, wakes the other | Waiting |
| 4 (N = 3) | 4 | Sees 4 > 3 and returns | Sees 4 > 3 and returns |

## Java

```java
import java.util.function.IntConsumer;

class OddEvenPrinter {
    private final int limit;
    private int next = 1;

    OddEvenPrinter(int limit) {
        this.limit = limit;
    }

    void printOdd(IntConsumer out) throws InterruptedException {
        printWhen(1, out);
    }

    void printEven(IntConsumer out) throws InterruptedException {
        printWhen(0, out);
    }

    private synchronized void printWhen(int parity, IntConsumer out) throws InterruptedException {
        while (true) {
            while (next <= limit && next % 2 != parity) {
                wait();
            }
            if (next > limit) {
                return;
            }
            out.accept(next++);
            notifyAll();
        }
    }
}
```

## Python

```python
import threading


class OddEvenPrinter:
    def __init__(self, limit):
        self._limit = limit
        self._next = 1
        self._cond = threading.Condition()

    def print_odd(self, out):
        self._print_when(1, out)

    def print_even(self, out):
        self._print_when(0, out)

    def _print_when(self, parity, out):
        with self._cond:
            while True:
                while self._next <= self._limit and self._next % 2 != parity:
                    self._cond.wait()
                if self._next > self._limit:
                    return
                out(self._next)
                self._next += 1
                self._cond.notify_all()
```

## Go

Go has a more direct tool. Two unbuffered channels pass the turn between two goroutines, and no lock or shared counter is needed.

```go
package main

import "sync"

func PrintOddEven(limit int, out func(int)) {
	if limit < 1 {
		return
	}
	oddTurn := make(chan struct{})
	evenTurn := make(chan struct{})
	var wg sync.WaitGroup
	worker := func(start int, mine, other chan struct{}) {
		defer wg.Done()
		for n := start; n <= limit; n += 2 {
			<-mine
			out(n)
			if n < limit {
				other <- struct{}{}
			}
		}
	}
	wg.Add(2)
	go worker(1, oddTurn, evenTurn)
	go worker(2, evenTurn, oddTurn)
	oddTurn <- struct{}{}
	wg.Wait()
}
```

The `if n < limit` check matters. Without it, the goroutine that prints the last number would send a turn that nobody receives, and it would block forever.

## Follow-up questions

- How does the solution change for three threads that print 1 to N in rotation? (Replace the parity check with `next % 3`.)
- Why is `notifyAll` safer than `notify` once there are more than two threads?
- How would you solve it in Java with two semaphores, one starting at 1 and one starting at 0?
- What happens to the Java and Python versions when N is 0?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-odd-even-printer) covers the alternating-thread family of questions, including the three-thread versions.
- Lesson: [Problem 8: ZeroEvenOdd Multithreading Problem](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/problem-8-zeroevenodd-multithreading-problem?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-odd-even-printer), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Problem 7: FizzBuzz Multithreading Problem](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/problem-7-fizzbuzz-multithreading-problem?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-odd-even-printer), in Grokking Multithreading and Concurrency for Coding Interviews
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-odd-even-printer) explains unbuffered channels and how a send hands control to a receiver.
- Related pages: [condition variables and monitors](../concepts/condition-variables-and-monitors.md).
