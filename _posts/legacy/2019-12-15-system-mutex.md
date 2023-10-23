---
layout: post
title: "System (6): Thread and Mutex"
date: 2019-12-15
categories: [System]
tags: [system, mutex]
math: true
---


This article is part of my review notes of “Unix System Programming” course. In this post, we will review the concept of thread, the creation and execution of **thread**, and how to use **mutex** to achieve synchronization between multiple threads.

## Thread in POSIX

In the previous post, we talked about forking and child processes, which is a standard way for a program to spin off processes and make use of multiple CPUs. To communicate with each other, parent and child processes can utilize various Inter Process Communication (IPC) approaches, including pipe, FIFO, message queue, and shared memory, as we have discussed in the last two posts. However, all these IPC approaches operate on different address space (cross-process), introducing some necessary overhead.

There is another way to make use of multiple CPUs, on the same address space: use multiple threads inside a single program, and the address space and resources are shared between them. Many higher level programming languages provide multithreading features, such as Java and Python (see my previous post). Here we focus on the thread interface specified by POSIX: [pthread](http://man7.org/linux/man-pages/man7/pthreads.7.html).

Here are some properties of threads:
* Each thread has its own stack, which means each thread has its own state and local variables.
* All threads share the other resources within the process, including: Text (code), Data and BSS (global variables), heap, other system resources such as Ublock, which includes process ID, uid and gid, open file table, signal table, etc.
* Some attributes are not shared, and are independent in each thread, such as: thread ID, signal mask, `errno` variable, scheduling policy, etc. By default, a newly created thread will inherit the signal mask of the calling thread.

### Pthread API

To create a thread, use the system [call](http://man7.org/linux/man-pages/man3/pthread_create.3.html):

```int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);```

* First argument: the pointer to the thread ID, which will be fulfilled after return.
* Second argument: to specify the thread attribute, such as the size of the stack, and scheduling policy.
* Third argument: the actual thread routine, the task assigned to this thread.
* Fourth argument: the sole argument to be passed to the routine, which means there can be at most one argument for the thread routine.

As said on the man page, a thread can be terminated in three ways:
* Active termination: the routine either returns, or calls `pthread_exit()`, in which cases the return value can be retrieved in `pthread_join()`.
* Passive termination: `pthread_cancel(pthread_t thread)` is called by others with the according thread ID.
* Exit all: if either thread calls `exit()`, the entire process will terminate.

Just like the parent process can wait for the termination of child processes, a thread A can wait for the termination of another thread B by calling the system [call](http://man7.org/linux/man-pages/man3/pthread_join.3.html) with thread B's ID.

```int pthread_join(pthread_t thread, void **retval);```

The thread routine takes at most one argument, and can return a value. However, as specified by the API, both of them need to be a pointer type. Since each thread has its own stack, and the stack will be destroyed after the routine returns, we need to make sure the return variable is **NOT** a local variable. A common pattern is to dynamically create a variable on heap, and returns the variable address. In this way, the variable value can be retained after the termination of the thread. Whoever receives the address should also be responsible to `free()` it.

## Synchronization: Mutex

In the previous post, we have talked about how to use **semaphore** to achieve synchronization in shared memory. For threads, because they shared many resources within the process, we also need a synchronization mechanism, to make sure exclusive access to certain critical resources. Semaphore still works; however, we should use another locking mechanism: **mutex**, which is a lightweight version of semaphore.

Here are some features of mutex, compared with semaphore:
* Unlike semaphore, mutex makes use of the [spin lock](https://en.wikipedia.org/wiki/Spinlock): it only issues a system call when it is locked for a long time; for short period of time, it just does the check without system call. Therefore, mutex is less expensive, because of less context switch brought by the system call.
* Unlike semaphore, mutex can only be binary: either lock or unlock.

Like semaphore, a mutex needs to be initialized. And there are two ways:
* `pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER` (the easiest way)
* Use `pthread_mutex_init()` system call.

The lock and unlock operations are specified by the following system call:
* `int pthread_mutex_lock(pthread_mutex_t *mutex);`
* `int pthread_mutex_unlock(pthread_mutex_t *mutex);`

And just like semaphore, the general pattern is: lock; use resources; unlock.

In the next post, we will be discussing how to use mutex in the classic consumer-producer problem.
