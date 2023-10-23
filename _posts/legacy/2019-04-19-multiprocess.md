---
layout: post
title: "Summary of Multithreading and Multiprocessing in Java and Python"
date: 2019-04-19
categories: [Programming]
tags: [java, python, multiprocess]
math: true
---

**Multithreading** and **multiprocessing** are provided in various modern programming languages for parallel execution. However, their implementation is language-specific, and the usage can be quite different among different languages. This article serves as a summary of their concepts and usage in Java and Python.

## Multithreading vs. Multiprocessing

Here we briefly introduce the abstract concepts of multithreading and multiprocessing regardless of implementation.

Both of them are designed for multitasking. The program can initiate multiple "workers" and let them execute "simultaneously". Two things to notice:
* The "worker" can be either thread or process which differs quite a lot.
* Each "worker" appears to execute "simultaneously"; they may be actual parallel execution on multiple cores, or "fake" parallelism by context switch. If latter is the case, the overall performance might be degraded by the overhead of context switch. Therefore, the context switch type of parallelism is only suggested for IO-intensive jobs rather than computation-intensive jobs.

The main differences between multithreading and multiprocessing are as follows:
* Multithreading shares the memory heap within the process; any thread can access or modify the objects in the same memory heap. Therefore, thread is rather lightweight because it contains less resources, and doesn't carry the memory space. This reduces the overhead of spinning up new threads or switching thread context. However, this also brings the downside of shared object management. To keep the shared object consistent under multiple threads, extra language-specific measurements have to be taken, which will be discussed in the later of this article.
* Multiprocessing spawns new processes instead of thread. Each process generally has a complete, private set of basic run-time resources including its own memory heap; therefore, all objects in the memory have to be copied when spawning new sub-processes, which increases the overhead of multiprocessing. However, the benefit is that we don't need shared object management, since each process only accesses its own copy of objects.

In simple applications, it is more common to use multithreading than multiprocessing. The following article will focus on multithreading in Java and Python and discuss the similarities and differences.

## Multithreading in Java

JVM natively supports multithreading running on multiple cores. Therefore, multithreading in Java can be true parallel execution. The general rule of thumb to achieve maximum performance is to have $c \times n$ threads, where $n$ is the number of cores, and $c$ is a small number under $5$. If there are too many threads relative to the number of cores, context switch will have to kick in, which brings about some overhead, and this overhead will decrease the performance for CPU-intensive jobs. In other words, a large number of threads only makes sense when the job type is IO-related such as server.

So how does Java handle shared object management? The design of JVM chooses to support fine-grained lock for each resource or object. Therefore in Java, we can use keywords such as "synchronized", "final", "volatile" to control the shared object. For a more detailed guide of shared object management in Java, we can check out my previous post. Here we discuss three examples:
* `synchronized`: any thread that needs to access a "synchronized" object would have to obtain the lock of this object. This guarantees only one thread can access this object each time, which avoids any data inconsistency.
* `final`: any field marked with "final" is immutable, or read-only, and can only be initialized in the constructor. Since the value is guaranteed not to be modified after initialization, the compiler will allow each thread to freely access/read this field.
* `volatile`: a field marked with "volatile" guarantees any value change will be visible to other threads. In multithreading, the JVM may utilize the local cache of each core to improve the performance, which may cause data inconsistency across multiple cores. JVM will guarantee to push any value change from the local cache to the main memory, and distribute to all other caches, so that local caches will always be consistent on all "volatile" fields.

## Multithreading in Python

Python is a script language and has different implementation. **CPython** is the C-implementation of Python, and Jython is the Java-implementation of Python. Since CPython is mostly used, here we only discuss the CPython case.

Unfortunately, CPython is not thread-safe, and the interpreter doesn't support fine-grained locking mechanism like JVM. Instead, it has a global interpreter lock (**GIL**); any thread must hold the GIL to access the memory space, and the parallel execution in CPython is just context switch. Therefore in CPython, multithreading is only suggested for IO-related jobs.

Python also provides multiprocessing libraries for true parallel execution. If the job type is CPU-intensive, the performance of multiprocessing is likely to surpass that of multithreading in CPython.
