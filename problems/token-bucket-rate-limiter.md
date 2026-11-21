# Token Bucket Rate Limiter

Build a rate limiter that many threads call before doing work. It must allow a sustained rate of R requests per second and a short burst of up to B requests.

The question combines an algorithm with a concurrency requirement, and it leads naturally into a system design discussion.

## Key idea

A token bucket holds up to B tokens. Tokens are added at R per second, and each request removes one. A request is allowed only when a token is available.

No background thread is needed to add tokens. The limiter stores the time of the last update, and on each call it adds `elapsed time x R` tokens, capped at B. This is called lazy refill.

Worked example with B = 5 and R = 10 per second:

| Time | Event | Tokens after |
|---|---|---|
| 0 ms | Bucket starts full | 5 |
| 0 ms | 5 requests arrive, all allowed | 0 |
| 0 ms | A 6th request arrives and is rejected | 0 |
| 250 ms | Refill adds 0.25 x 10 = 2.5, one request allowed | 1.5 |
| 2,000 ms | Refill would add 17.5, capped at 5 | 5 |

The refill and the decision must happen as one atomic step. If two threads both read 1 token and both subtract it, the limiter allows two requests where it should allow one. One mutex around the whole method prevents that.

## Java

```java
class TokenBucket {
    private final double capacity;
    private final double refillPerNano;
    private double tokens;
    private long lastRefill;

    TokenBucket(double capacity, double refillPerSecond) {
        this.capacity = capacity;
        this.refillPerNano = refillPerSecond / 1_000_000_000.0;
        this.tokens = capacity;
        this.lastRefill = System.nanoTime();
    }

    synchronized boolean tryAcquire() {
        long now = System.nanoTime();
        tokens = Math.min(capacity, tokens + (now - lastRefill) * refillPerNano);
        lastRefill = now;
        if (tokens < 1) {
            return false;
        }
        tokens -= 1;
        return true;
    }
}
```

`System.nanoTime()` is a monotonic clock, which means it never moves backward. `System.currentTimeMillis()` can jump when the system clock is corrected, and a backward jump would produce a negative refill.

## Python

```python
import threading
import time


class TokenBucket:
    def __init__(self, capacity, refill_per_second):
        self._capacity = capacity
        self._rate = refill_per_second
        self._tokens = float(capacity)
        self._last = time.monotonic()
        self._lock = threading.Lock()

    def try_acquire(self):
        with self._lock:
            now = time.monotonic()
            elapsed = now - self._last
            self._tokens = min(self._capacity, self._tokens + elapsed * self._rate)
            self._last = now
            if self._tokens < 1:
                return False
            self._tokens -= 1
            return True
```

## Go

```go
package main

import (
	"math"
	"sync"
	"time"
)

type TokenBucket struct {
	mu       sync.Mutex
	capacity float64
	rate     float64
	tokens   float64
	last     time.Time
}

func NewTokenBucket(capacity, refillPerSecond float64) *TokenBucket {
	return &TokenBucket{capacity: capacity, rate: refillPerSecond, tokens: capacity, last: time.Now()}
}

func (b *TokenBucket) TryAcquire() bool {
	b.mu.Lock()
	defer b.mu.Unlock()
	now := time.Now()
	b.tokens = math.Min(b.capacity, b.tokens+now.Sub(b.last).Seconds()*b.rate)
	b.last = now
	if b.tokens < 1 {
		return false
	}
	b.tokens--
	return true
}
```

`time.Now()` in Go carries a monotonic reading, and `Sub` uses it, so the calculation is safe against clock corrections.

## Follow-up questions

- How would you add a blocking `acquire` that sleeps until a token is available, and how long should it sleep?
- How do you keep one bucket per user for 1 million users without holding 1 million buckets in memory forever?
- How does the design change when 20 servers must share one limit? (The state moves to a shared store, and the refill and decrement must run there as one atomic operation.)
- How does a token bucket differ from a fixed window counter at the boundary between two windows?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-token-bucket-rate-limiter) covers rate limiter implementations as coding problems.
- [Grokking the System Design Interview](https://www.designgurus.io/course/grokking-the-system-design-interview?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-token-bucket-rate-limiter) designs a distributed rate limiter as a full system.
- The [Grokking System Design repository](https://github.com/design-gurus/grokking-system-design) compares the rate limiting algorithms in one cheat sheet.
- Related pages: [atomics and memory visibility](../concepts/atomics-and-memory-visibility.md).
- Free article: [How to Design a Rate Limiter: Algorithms, Architecture, and Trade-offs](https://www.designgurus.io/blog/grokking-rate-limiters?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-token-bucket-rate-limiter), the system design version of the same problem.
- Free answer: [What Is Token Bucket vs Leaky Bucket?](https://www.designgurus.io/answers/detail/what-is-token-bucket-vs-leaky-bucket?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-token-bucket-rate-limiter), a comparison of the two algorithms.
