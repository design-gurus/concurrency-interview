# Common Mistakes in Concurrency Interviews

Most candidates who fail a concurrency round know the primitives. They lose the round on a small number of repeated errors, in the code or in how they explain it. The ten below are the ones to check for first.

## In the code

| # | Mistake | Why it fails | Fix |
|---|---|---|---|
| 1 | Checking a wait condition with `if` | A woken thread can find the condition false again | Use a `while` loop around every `wait` |
| 2 | Calling `notify` when waiters wait for different conditions | The single wake-up can go to a thread that cannot proceed, and the program stalls | Use `notifyAll`, or one condition variable per condition |
| 3 | Releasing a lock outside `finally`, `with`, or `defer` | An exception leaves the lock held forever | Always pair the acquire with a guaranteed release |
| 4 | Making each step atomic but not the sequence | "Check, then act" on a concurrent map is still a race | Use one atomic operation, such as `computeIfAbsent`, or one lock around both steps |
| 5 | Acquiring two locks in an order that depends on the arguments | Two threads with opposite arguments deadlock | Order the locks by a stable key, such as an identifier |
| 6 | Holding a lock during a slow call | Every other thread waits for the network or the disk | Copy what is needed, release the lock, and then make the call |
| 7 | Treating `volatile` as a lock | It gives visibility, not atomic updates | Use an atomic variable or a lock for read-change-write |
| 8 | Forgetting how the program ends | Threads wait forever after the last item | Include the end condition in every wait, and wake all waiters when finishing |

## In the explanation

| # | Mistake | What the interviewer concludes | Better |
|---|---|---|---|
| 9 | Saying "this is thread-safe" without a reason | The candidate is hoping, not reasoning | Name the shared state, the lock that protects it, and one interleaving that the lock prevents |
| 10 | Starting with the most advanced tool | The candidate cannot judge cost | Start with one mutex, state its cost, and refine only when asked |

## A worked example of mistake 4

The code below uses a thread-safe map and still has a bug.

```java
if (!cache.containsKey(key)) {
    cache.put(key, load(key));
}
return cache.get(key);
```

Two threads can both find the key missing, and both call `load`. If `load` takes 2 seconds and 50 requests arrive together for the same key, the service performs 50 loads instead of 1. The single call `cache.computeIfAbsent(key, k -> load(k))` removes the race, because the map runs the check and the insert as one step.

## A sentence to practice

For any solution, be ready to complete this sentence aloud:

> "The shared state is ___. It is protected by ___. Without that protection, thread A could ___ while thread B ___, and the result would be ___."

A candidate who can say this for each problem in the [problems](../problems/README.md) section has covered what the round measures.

## Three checks before saying "done"

1. **Safety:** can two threads ever be inside the critical section together?
2. **Liveness:** can any thread wait forever, through a deadlock, a lost wake-up, or starvation?
3. **Termination:** when the work ends, does every thread return?

Run the three checks aloud, in that order. Finding and fixing a bug in your own code during this review counts in your favor, because it shows the reasoning that the round is designed to measure.

## Go deeper

- [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=cheat-sheets-common-mistakes) works through each of these mistakes inside full interview problems.
- Related pages: [condition variables and monitors](../concepts/condition-variables-and-monitors.md), [deadlock, livelock, and starvation](../concepts/deadlock-livelock-and-starvation.md), and the [question bank](question-bank.md).
