# Concurrent Web Crawler

Given a start URL and a function `fetch(url)` that returns the links on a page, return every reachable URL on the same host. The `fetch` call blocks on the network, so the crawler must run many fetches at once and must fetch each URL only once.

This is the largest problem in the set. It combines a shared set, a bounded number of workers, and the question of how the program knows it has finished.

## Key idea

Three decisions make up the solution.

| Decision | Requirement | Common answer |
|---|---|---|
| The visited set | "Check, then add" must be one atomic step | A concurrent set, or a mutex around a map |
| The worker limit | 10,000 links must not start 10,000 threads | A fixed pool, or a counting semaphore |
| Completion | Return only when no fetch is running and none is queued | A count of pending tasks that reaches 0 |

Completion is where most attempts fail. An empty queue does not mean the crawl is done, because a running fetch may be about to add 50 new links. The safe rule is to count a task as pending before it is submitted and to subtract it only after it has submitted its own children. The count can then reach 0 only when all work is finished.

## Java

```java
import java.net.URI;
import java.util.List;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.Function;

class WebCrawler {
    private final Set<String> seen = ConcurrentHashMap.newKeySet();
    private final ExecutorService pool = Executors.newFixedThreadPool(8);
    private final AtomicInteger pending = new AtomicInteger();
    private final CountDownLatch done = new CountDownLatch(1);
    private final Function<String, List<String>> fetch;
    private String host;

    WebCrawler(Function<String, List<String>> fetch) {
        this.fetch = fetch;
    }

    Set<String> crawl(String start) throws InterruptedException {
        host = URI.create(start).getHost();
        submit(start);
        done.await();
        pool.shutdown();
        return seen;
    }

    private void submit(String url) {
        if (!host.equals(URI.create(url).getHost()) || !seen.add(url)) {
            return;
        }
        pending.incrementAndGet();
        pool.execute(() -> {
            try {
                fetch.apply(url).forEach(this::submit);
            } finally {
                if (pending.decrementAndGet() == 0) {
                    done.countDown();
                }
            }
        });
    }
}
```

`seen.add(url)` returns `false` when the URL is already present, so the check and the insert are one atomic call. A separate `contains` followed by `add` would let two threads fetch the same page.

## Python

```python
from concurrent.futures import ThreadPoolExecutor
from urllib.parse import urlsplit


def crawl(start, fetch, workers=8):
    host = urlsplit(start).hostname
    seen = {start}
    frontier = [start]
    with ThreadPoolExecutor(max_workers=workers) as pool:
        while frontier:
            next_frontier = []
            for links in pool.map(fetch, frontier):
                for url in links:
                    if url not in seen and urlsplit(url).hostname == host:
                        seen.add(url)
                        next_frontier.append(url)
            frontier = next_frontier
    return seen
```

This version crawls one level at a time. The pool fetches every page of a level in parallel, and only the main thread touches `seen`, so the set needs no lock. The cost is that one slow page delays the start of the next level.

## Go

```go
package main

import (
	"net/url"
	"sync"
)

func hostOf(raw string) string {
	u, err := url.Parse(raw)
	if err != nil {
		return ""
	}
	return u.Hostname()
}

func Crawl(start string, fetch func(string) []string, workers int) []string {
	var mu sync.Mutex
	var wg sync.WaitGroup
	seen := map[string]bool{start: true}
	slots := make(chan struct{}, workers)
	host := hostOf(start)

	var visit func(u string)
	visit = func(u string) {
		defer wg.Done()
		slots <- struct{}{}
		links := fetch(u)
		<-slots
		for _, link := range links {
			if hostOf(link) != host {
				continue
			}
			mu.Lock()
			isNew := !seen[link]
			seen[link] = true
			mu.Unlock()
			if isNew {
				wg.Add(1)
				go visit(link)
			}
		}
	}
	wg.Add(1)
	go visit(start)
	wg.Wait()

	result := make([]string, 0, len(seen))
	for u := range seen {
		result = append(result, u)
	}
	return result
}
```

Goroutines are cheap enough to start one per URL. The buffered channel `slots` acts as a counting semaphore, so at most `workers` fetches run at the same time. `wg.Add(1)` runs before each `go` statement, which is the same counting rule as in the Java version.

## Follow-up questions

- Why is "the queue is empty" the wrong test for completion?
- How would you limit the crawler to 2 requests per second for each host? (See the [token bucket rate limiter](token-bucket-rate-limiter.md).)
- What happens when `fetch` throws an error or never returns, and how would you add a timeout of 5 seconds?
- How does the design change when the visited set no longer fits in the memory of one machine?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-concurrent-web-crawler) covers thread pools, concurrent collections, and task coordination.
- Lesson: [Problem 16: Multithreaded Web Crawler](https://www.designgurus.io/course-play/grokking-multithreading-and-concurrency-for-coding-interviews/doc/problem-16-scenario-multithreaded-web-crawler?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-concurrent-web-crawler), in Grokking Multithreading and Concurrency for Coding Interviews
- Lesson: [Case Study: A Concurrent Web Crawler](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/case-study-a-concurrent-web-crawler?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-concurrent-web-crawler), in Grokking Concurrency in Go
- Lesson: [A Concurrent Web Crawler](https://www.designgurus.io/course-play/grokking-concurrency-in-python/doc/a-concurrent-web-crawler?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-concurrent-web-crawler), in Grokking Concurrency in Python
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-concurrent-web-crawler) covers worker pools, `sync.WaitGroup`, and bounded concurrency with channels.
- [Grokking Concurrency in Python](https://www.designgurus.io/course/grokking-concurrency-in-python?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-concurrent-web-crawler) covers `concurrent.futures` and `asyncio` for network-bound work.
- Related pages: [thread pools](../concepts/thread-pools.md) and [async versus threads](../concepts/async-vs-threads.md).
- Free article: [How to Design a Web Crawler from Scratch](https://www.designgurus.io/blog/design-web-crawler?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-concurrent-web-crawler), the system design version of the same problem.
