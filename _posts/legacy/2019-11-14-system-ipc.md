---
layout: post
title: "System (4): Inter Process Communication (IPC) Part 1"
date: 2019-11-14
categories: [System]
tags: [system, ipc]
math: true
---

This article is part of my review notes of "Unix System Programming" course. In this post, we start to look into the Inter Process Communication (IPC): **pipe, FIFO, message queue**.

All these three techniques require a system call each time to send or receive data. Since any communication involves at least two parties - a read end and a write end, the system call imposes blocking behavior by default. Let's look at the blocking behavior first.

## Blocking

The blocking is a general concept for the consumer-producer problem. Blocking can take place on either the read end or the write end. Here let's discuss the blocking in the context of pipe, FIFO and message queue.
* Read from an empty collection (either pipe, FIFO, message queue): the process gets blocked until there are something available in the collection
* Write to a collection with insufficient capacity: the process gets blocked until there are enough space available
* Write to a collection without a read end open (for pipe, FIFO): the process gets blocked until another process opens the collection with the read end
* Read from a collection with more than available data: returns min(requested, available)

## Pipe

`pipe()` is a system call that creates unidirectional data channel/buffer in memory. The signature is ([man page](http://man7.org/linux/man-pages/man2/pipe.2.html)):

```int pipe(int pipefd[2]);```

Two file descriptors are assigned: `pipefd[0]` is for the read, and `pipefd[1]` is for the write. Everything written to `pipefd[1]` can be read from `pipefd[0]`.

Here are a few things to know about the pipe buffer:
* The buffer has a size limit, typically 4KB or 8KB.
* `pipe` operates on memory directly, without disk activity. Therefore speed is fast.
* `pipe` imposes blocking behavior by default.
* `close(pipefd[1])` will signal `EOF` to the read end.

It is common to use `pipe()` together with `fork()` to send data back and forth between parent process and child processes. From the previous post, we know that the open file table is shared between parent and children. There are several points to pay attention to:
* Each process should first close the read/write end that it is not using.
* Each process should close the read/write end after use. Especially, the process to write should `close(pipefd[1])`, so signal the reading process `EOF`; otherwise, the reading process will be kept blocked.

Another useful system call is [dup2](http://man7.org/linux/man-pages/man2/dup.2.html):

```int dup2(int oldfd, int newfd);```

`dup2()` copies the file descriptor at `oldfd` to `newfd`. A common trick is to replace the `stdin` (fd is $0$), `stdout` (fd is $1$) to achieve read/write redirection.

Combining `fork()`, `pipe()`, `dup2()`, we can achieve some control flow that we see in shell. For example, when we execute "`ls | wc`" in the shell, what happened is the following:
* Parent process (shell process): `pipe(fd), fork(), fork(), close(fd[0]), close(fd[1]), wait(), wait()`
* Child process 1 (from `fork()`): `close(fd[0]), dup2(fd[1], 1), close(fd[1]), execl("ls", "ls", NULL)`
* Child process 2 (from `fork()`): `close(fd[1]), dup2(fd[0], 0), close(fd[0]), execl("wc", "wc", NULL)`

## FIFO

In the first post of this series, we discussed different types of files. Here we have a new file type: FIFO. The [man page](http://man7.org/linux/man-pages/man3/mkfifo.3.html) has a good introduction of FIFO:

> "A FIFO special file is similar to a pipe, except that it is created in a different way. Instead of being an anonymous communications channel, a FIFO special file is entered into the filesystem by calling mkfifo(). Once you have created a FIFO special file in this way, any process can open it for reading or writing, in the same way as an ordinary file. However, it has to be open at both ends simultaneously before you can proceed to do any input or output operations on it. Opening a FIFO for reading normally blocks until some other process opens the same FIFO for writing, and vice versa."

In summary, FIFO is a named version of pipe, and similar to pipe in many ways:
* Both work at memory speed, since data only transfers in buffer, and never goes to the actual file system.
* Similar to pipe, "if any data is still in a FIFO when all readers and writers close their file descriptors, the data is discarded with no error indication. This is like a water pipe too: The water leaks out if you dis- connect the pipe at both ends." (from Advanced UNIX Programming)
* If a FIFO file is opened without **O_NONBLOCK** flag, then the read/write system call will have blocking behavior (same as pipe) by default.
* Both guarantees atomic write. so there can be multiple writers. However, both cannot have multiple readers because of no atomicity guarantees for reading.

And also some differences:
* The FIFO file is persistent on the file system once created (although the actual transferred data never goes to file system). Any process can access to FIFO file and starts IPC. However, pipe can only be operated within the same address space.
* Data transfer for FIFO comes from source address space, to kernel buffer, to destination address space (same for message queue). Therefore. FIFO is more expensive than pipe.

To create a FIFO file, we can use the system call:

```int mkfifo(const char *pathname, mode_t mode);```

If we don't want the blocking behavior, we can use the **O_NONBLOCK** flag in the `open()`.

## System V Message Queue

`pipe` and FIFO are all byte stream oriented. Message queue is object/message oriented. It doesn't require any file descriptor; instead, a key is required to identify a certain message queue. The default blocking behavior also applies here.

There are three types of System V IPC objects: message queue, semaphore and shared memory segment (will be talked about in the next post). They are the following properties:
* They are not designed to be like files, and file-related system calls such as `read()` or `stat()` doesn't work on them. They have their own lifetime rules, permission system.
* Lifetime is the same as the kernel (destroyed on reboot).
* They are accessed by an identifier that is assigned by kernel. Any processes with the identifier are able to access the IPC object.

When it comes to the message queue, here is its unique properties, compared with pipe or FIFO:
* We can have multiple readers for message queue, since it is object oriented, and we don't need worry about reading atomicity; while there can be only be one reader for pipe or FIFO.

To create or get a message queue, we have the system call ([man page](http://man7.org/linux/man-pages/man2/msgget.2.html)):

```int msgget(key_t key, int msgflg);```

The return value is the message queue ID.

To send/receive a message, we have the system call:

```int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);```

```ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);```

**IPC_NOWAIT** flag can be specified here to remove blocking behavior.

Each message also has a type attribute (see the [man page](http://man7.org/linux/man-pages/man2/msgrcv.2.html)), and it can be used as message priority level.

`ipcs` is a Unix utility command to list the current message queues and shared memory (next post) in the system.

At last, it is also worthing noting that pipe, FIFO, and message queue all operate on single instance, and cannot be transferred across network.

