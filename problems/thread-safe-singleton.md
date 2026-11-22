# Thread-Safe Singleton

Create a class that has exactly one instance, built the first time it is needed. When 100 threads ask for it at the same moment, all 100 must receive the same fully constructed object.

The question is short, but it tests memory visibility, which is the hardest idea in this guide.

## Key idea

The naive version checks `if instance == null` and then creates the object. Two threads can both pass the check before either assigns, so two objects are built.

Locking every call fixes that, but every later read then pays for a lock it no longer needs. The classic answer is double-checked locking: check without the lock, take the lock only when the instance is missing, and check again inside.

In Java, the field must be declared `volatile`. Without it, a second thread can observe a non-null reference to an object whose constructor has not finished, because the runtime may publish the reference before the fields are written.

| Approach | Lazy | Lock cost after creation | Notes |
|---|---|---|---|
| Lock on every call | Yes | Every call | Correct but slow under contention |
| Double-checked locking with `volatile` | Yes | None | The answer most Java interviewers expect |
| Holder class (Java) | Yes | None | The class loader guarantees one initialization |
| `sync.Once` (Go) | Yes | One atomic read | The idiomatic Go answer |

## Java

```java
class Config {
    private static volatile Config instance;

    private Config() {
    }

    static Config getInstance() {
        Config local = instance;
        if (local == null) {
            synchronized (Config.class) {
                local = instance;
                if (local == null) {
                    local = new Config();
                    instance = local;
                }
            }
        }
        return local;
    }
}

class LazyConfig {
    private LazyConfig() {
    }

    private static class Holder {
        static final LazyConfig INSTANCE = new LazyConfig();
    }

    static LazyConfig getInstance() {
        return Holder.INSTANCE;
    }
}
```

`Config` shows double-checked locking. `LazyConfig` shows the holder idiom: the nested class is loaded on the first call to `getInstance`, and the language guarantees that class initialization runs once.

## Python

```python
import threading


class Config:
    _instance = None
    _lock = threading.Lock()

    def __init__(self):
        self.settings = {}

    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = cls()
        return cls._instance
```

A module is the simplest singleton in Python, because the interpreter imports each module once. Interviewers usually accept that answer and then ask for the class version.

## Go

```go
package main

import "sync"

type Config struct {
	settings map[string]string
}

var (
	instance *Config
	once     sync.Once
)

func GetConfig() *Config {
	once.Do(func() {
		instance = &Config{settings: map[string]string{}}
	})
	return instance
}
```

`sync.Once` runs the function exactly once, and every other caller waits until that first call has returned.

## Follow-up questions

- What can go wrong in the Java version if `volatile` is removed?
- Why does the Java version copy the field into a local variable? (It reads the `volatile` field once on the common path instead of twice.)
- How would you make the singleton safe against reflection and serialization in Java? (An `enum` with one value.)
- Why do many teams prefer passing the object in through a constructor instead of using a singleton, especially for testing?

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-thread-safe-singleton) explains safe publication and the Java memory model with worked problems.
- Lesson: [Once, OnceFunc, and OnceValue](https://www.designgurus.io/course-play/grokking-concurrency-in-go/doc/once-oncefunc-and-oncevalue?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-thread-safe-singleton), in Grokking Concurrency in Go
- [Grokking Concurrency in Go](https://www.designgurus.io/course/grokking-concurrency-in-go?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=problems-thread-safe-singleton) covers `sync.Once`, `sync.Mutex`, and the `sync/atomic` package.
- Related pages: [atomics and memory visibility](../concepts/atomics-and-memory-visibility.md) and [race conditions](../concepts/race-conditions.md).
