---
layout: post
title: "System (7): Condition Variables, Consumer-Producer Problem"
date: 2019-12-16
categories: [System]
tags: [system]
math: true
---

This article is part of my review notes of “Unix System Programming” course. In this post, we continue the topic from the last post: **thread** and **mutex**, and discuss the classic consumer-producer problem using mutex and **condition variables**.

## Consumer-Producer Problem

Sometimes, just using mutex is not enough. Let's look at a simple consumer-producer problem (example from Advanced Unix Programming).

Thread A is a producer that adds items to a queue with certain capacity, and thread B is a consumer that removes items from the queue. We use a single mutex M to make sure exclusive access to the queue:

![consumer](/assets/img/legacy/consumer1.png)

It works. However, thread B would waste a lot of time if the queue is empty: it would just keep locking and unlocking M and checks if any item is available. These checking is unnecessary and simply just burns CPU time without bringing any good. It would be nice if thread B keeps being blocked once the queue is empty, until being notified by a signal that there is something available in the queue.

We can try to use a second mutex, but it turns out that there could be deadlock and other problems when using two mutexes in this scenario (details on the book). To solve this problem, POSIX provides a mechanism called **condition variables**.

### Condition Variables

Here is our solution:

![consumer](/assets/img/legacy/consumer2.png)

Here is the illustration from the book:

> "In Thread B, cond_wait waits until condition C is signaled, which it is in Thread A’s step 4. But cond_wait has a special interaction with mutex M (its second argument): The mutex must be locked when cond_wait is called. It is unlocked during the waiting, and then relocked automatically when cond_wait returns. That way there’s no possibility of a missed signal or of deadlock."
> "Why is cond_wait in a loop that tests whether the queue is empty, when condition C gets signaled (by Thread A) only when the queue is nonempty? It’s because cond_wait, like any UNIX blocking system call, is subject to being interrupted (e.g., by the arrival of a traditional UNIX signal, such as SIGINT), in which case there might be a return with the queue still empty. A loop that tests the predicate (empty queue, in this case) ensures that we’ll go right back into cond_wait on such a spurious return. Because spurious returns are infrequent, the wasted CPU time is inconsequential."
> "Thus, to signal and wait for a condition, you need three things: a condition vari- able, a mutex, and a predicate. The first two are special data types, but the predicate is just some ordinary code that makes sense for your program—it’s the concrete details of what the condition variable represents abstractly."

Above is pretty self-explanatory from the book. To initialize a condition variable, the easiest way is:

```pthread_cond_t cond = PTHREAD_COND_INITIALIZER```

### cond_broadcast vs. cond_signal

There are two system [calls](https://linux.die.net/man/3/pthread_cond_signal) that wake up waiting threads:

```int pthread_cond_broadcast(pthread_cond_t *cond);```

```int pthread_cond_signal(pthread_cond_t *cond);```

The difference is `broadcast` would unblock all waiting threads: unblock 1st waiting thread A -> A checks condition -> A fails condition, keep waiting -> unblock 2nd waiting thread B -> B checks condition -> B passes condition, acquires lock -> broadcasting is over

In other words, `broadcast` would unblock all waiting threads one-by-one, and if any thread is able to pass the condition and acquire the lock, this broadcasting round is over.

`signal` only unlocks at least one waiting thread.

We shall see that if only one condition is used in both signal/broadcast and wait, then signal and broadcast have no difference, since the first unblocked waiting thread would always pass the condition.

When there are more than one condition, e.g. broadcast would wake up all waiting threads when a condition variable `val > 0`, and the waiting threads are checking different conditions: `val > 10` or `val <= 10`, then we do need broadcast in this case, to make sure all conditions are checked.

This consumer-producer problem can be a general framework to power up event-driven program, where each thread can be an event producer and throws out events to a queue, and one special thread as an event consumer to process all events in the queue. Following this framework, the program is more modular and less prone to some complex deadlock problem.

## Other Variants

A simpler version of consumer-producer problem is discussed in the previous post, where the queue capacity is only one. In this case, we can achieve synchronization using only two semaphores/mutexes, without using condition variables. The two semaphores/mutexes represent "ready to produce" and "read to consume".

