# Thread-Safe LRU Cache

Build a cache that holds at most N entries and removes the least recently used (LRU) entry when it is full. Many threads must be able to call `get` and `put` at the same time without corrupting it.

The single-threaded version is a standard coding question. The concurrent version adds one insight that many candidates miss.

## Key idea

An LRU cache is a hash map plus a doubly linked list. The map finds an entry in constant time, and the list keeps entries in order of use.

The insight is that **`get` is a write**. Every read moves an entry to the front of the list, so two concurrent readers both change the list. A readers-writers lock therefore gives no benefit, and one plain mutex around every operation is the correct first answer.

| Approach | Correct | Parallel reads | When to offer it |
|---|---|---|---|
| One mutex around `get` and `put` | Yes | No | First answer |
| Readers-writers lock | No, because `get` changes the list | Not applicable | Never, and say why |
| Several independent segments, each with its own mutex | Yes | Across segments | When asked to scale |

With 16 segments chosen by the hash of the key, two threads contend only when their keys fall into the same segment. The cost is that eviction becomes least recently used per segment, which is an approximation.

## Java

```java
import java.util.LinkedHashMap;
import java.util.Map;

class LruCache<K, V> {
    private final int capacity;
    private final LinkedHashMap<K, V> map;

    LruCache(int capacity) {
        this.capacity = capacity;
        this.map = new LinkedHashMap<>(16, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > LruCache.this.capacity;
            }
        };
    }

    synchronized V get(K key) {
        return map.get(key);
    }

    synchronized void put(K key, V value) {
        map.put(key, value);
    }

    synchronized int size() {
        return map.size();
    }
}
```

The third constructor argument, `true`, asks `LinkedHashMap` to keep entries in access order. Some interviewers ask for the linked list written by hand, as the Go version below does.

## Python

```python
import threading
from collections import OrderedDict


class LruCache:
    def __init__(self, capacity):
        self._capacity = capacity
        self._items = OrderedDict()
        self._lock = threading.Lock()

    def get(self, key, default=None):
        with self._lock:
            if key not in self._items:
                return default
            self._items.move_to_end(key)
            return self._items[key]

    def put(self, key, value):
        with self._lock:
            self._items[key] = value
            self._items.move_to_end(key)
            if len(self._items) > self._capacity:
                self._items.popitem(last=False)
```

The lock is required even though the interpreter runs one thread at a time. Each method performs several steps, and a thread switch between two of them would leave the cache in an inconsistent state.

## Go

```go
package main

import (
	"container/list"
	"sync"
)

type entry[K comparable, V any] struct {
	key   K
	value V
}

type LruCache[K comparable, V any] struct {
	mu       sync.Mutex
	capacity int
	order    *list.List
	items    map[K]*list.Element
}

func NewLruCache[K comparable, V any](capacity int) *LruCache[K, V] {
	return &LruCache[K, V]{capacity: capacity, order: list.New(), items: map[K]*list.Element{}}
}

func (c *LruCache[K, V]) Get(key K) (V, bool) {
	c.mu.Lock()
	defer c.mu.Unlock()
	el, ok := c.items[key]
	if !ok {
		var zero V
		return zero, false
	}
	c.order.MoveToFront(el)
	return el.Value.(entry[K, V]).value, true
}

func (c *LruCache[K, V]) Put(key K, value V) {
	c.mu.Lock()
	defer c.mu.Unlock()
	if el, ok := c.items[key]; ok {
		el.Value = entry[K, V]{key, value}
		c.order.MoveToFront(el)
		return
	}
	c.items[key] = c.order.PushFront(entry[K, V]{key, value})
	if c.order.Len() > c.capacity {
		oldest := c.order.Back()
		c.order.Remove(oldest)
		delete(c.items, oldest.Value.(entry[K, V]).key)
	}
}
```

## Follow-up questions

- Why does a readers-writers lock not help this cache?
- Two threads miss on the same key and both compute the value, which takes 2 seconds. How would you make the second thread wait for the first result instead?
- How would you add a time to live of 60 seconds per entry?
- How do segments change the eviction guarantee, and when is that acceptable?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-thread-safe-lru-cache) covers thread-safe data structures and the locking choices behind them.
- [Grokking the Coding Interview](https://www.designgurus.io/course/grokking-the-coding-interview?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-thread-safe-lru-cache) covers the single-threaded data structure patterns that this problem builds on.
- Related pages: [locks, mutexes, and semaphores](../concepts/locks-mutexes-and-semaphores.md) and the [readers-writers lock](readers-writers-lock.md).
