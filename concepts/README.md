# Concepts

Eight short pages that cover the ideas behind nearly every concurrency interview question. Each page ends with the questions interviewers ask about it. Read them in order if the topic is new, or go directly to the one that an interviewer asked about.

| Page | The question it answers |
|---|---|
| [Threads vs processes](threads-vs-processes.md) | What do threads share, and how does concurrency differ from parallelism? |
| [Race conditions](race-conditions.md) | Why is `count++` unsafe, and what is a check-then-act bug? |
| [Locks, mutexes, and semaphores](locks-mutexes-and-semaphores.md) | Which lock fits which problem, and how does a mutex differ from a semaphore? |
| [Deadlock, livelock, and starvation](deadlock-livelock-and-starvation.md) | What are the four conditions for deadlock, and how is each one removed? |
| [Condition variables and monitors](condition-variables-and-monitors.md) | How does a thread wait for a state, and why is the check a loop? |
| [Thread pools](thread-pools.md) | How many threads should a pool have, and what happens when its queue is full? |
| [Atomics and memory visibility](atomics-and-memory-visibility.md) | What does `volatile` guarantee, and what is compare-and-swap? |
| [Async vs threads](async-vs-threads.md) | When is an event loop better than threads, and what does Python's global interpreter lock change? |

After the concepts, practice with the [problems](../problems/README.md), or test recall with the [question bank](../cheat-sheets/question-bank.md).

For full lessons on every topic here, see [Grokking Multithreading and Concurrency for Coding Interviews](https://www.designgurus.io/course/grokking-multithreading-and-concurrency-for-coding-interviews?utm_source=github&utm_medium=repo&utm_campaign=concurrency-interview&utm_content=concepts-readme).
