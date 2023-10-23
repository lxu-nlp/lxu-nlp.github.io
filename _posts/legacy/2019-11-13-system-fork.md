---
layout: post
title: "System (3): File Handling; Fork and Child Process"
date: 2019-11-13
categories: [System]
tags: [system]
math: true
---

This article is part of my review notes of "Unix System Programming" course. In this post, we go over the file handling and read/write operations, and the forking mechanism to create child processes.

## File Handling

In the previous post, we talked about the concept of the **open file table** and **file descriptor**. A file descriptor keeps the file inode number and its current offset, and it is passed to the read/write system call to identify where to read/write.

The system call to [open](http://man7.org/linux/man-pages/man2/open.2.html) a file and get its file descriptor is ([man page](http://man7.org/linux/man-pages/man2/open.2.html)):

```int open(const char *pathname, int flags, mode_t mode);```

The return value is the file descriptor value. It is just an integer, representing the index of the "real" file descriptor in the open file table.

We can pass different flags to define the behavior of the open operation, such as whether to create a new file when the path doesn't exist. We will discuss some important flags later.

The system call to [read](http://man7.org/linux/man-pages/man2/read.2.html) and [write](http://man7.org/linux/man-pages/man2/write.2.html) a file is:

```ssize_t read(int fd, void *buf, size_t count);```

```ssize_t write(int fd, const void *buf, size_t count);```

Both takes a file descriptor as an argument to identify where to read or write.

There are three special file descriptors: $0$ for standard input, $1$ for standard output, $2$ for standard error. These three special file descriptors are available directly, and never need to be opened in the program.

### Synchronized I/O

Since read or write on disk is pretty expensive, by default, the system call uses underlying buffer or cache to reduce the overhead of disk activities. Therefore, the read or write itself can be pretty efficient.

An important aspect is the write operation. By default, the write system call will return immediately after the data has been copied to the underlying buffer cache, but not necessarily already written on disk. This is not always desired; some programs may require high data integrity. We can control the synchronization behavior by using some flags in the open system call.

* **O_SYNC**: this flag provides synchronized I/O file integrity completion; write system call won't return until all data and associated metadata are flushed to the underlying hardware.
* **O_DSYNC**: this flags provides synchronized I/O data integrity completion; write system call guarantees that all data is written to the hardware after it returns, but not necessarily metadata. This is a weakened version of O_SYNC.

In addition to use flags, we can also explicitly call `fsync()` to flush all data to the underlying hardware.

On the contrary, `aiocb` is a data structure in libc that defines asynchronous read/write operation.

`fcntl()` is the system call to reset the flags after the file is opened.

### Buffered I/O

read/write is efficient because it utilizes some underlying buffer cache. However, from the previous post, we know that each system call is still expensive, because it involves context switch, from user mode to kernel mode. To alleviate this problem, C standard library provides a set of buffered I/O functions to reduce the number of system calls, hence minimizing context switching.

The related functions are: `fopen()`, `fgetc()`, `fgets()`, `fputc()`, `fputs()`, etc.

These functions encapsulate pre-read and post-write operations that are hidden from the users. When `fopen()` is called, a buffer is created in the process's BSS segment. The `fputs()` will write data to the buffer; it will issue a write system call when the buffer is full.

In summary, the buffered I/O wraps up the read/write system call and minimizes the context switching. Therefore, for a usual program, it is recommended and also more convenient to use buffered I/O, instead of issuing system call directly.

## Fork and Child Process

[fork](http://man7.org/linux/man-pages/man2/fork.2.html) is the system call to create a child process which is a clone of the parent process. The exact system call is:

```pid_t fork(void);```

The child process has a copy of the following in the parent process:
* All register values
* Address space (text, data, BSS, runtime stack, ...)
* Most of the Ublock values, including signal table and signal mask; some values are different, such as:
  * PID is the new child PID
  * PPID is the PID of the parent process
  * The open file table is shared between child processes and the parent process; `fork()` will increment the reference count of each file descriptor by $1$

Since all register values and the runtime stack of the child process are exactly the same as those of the parent process, the child process has the same next instruction, and can use all variables, as if it were the parent process.

The return value for parent process is the child PID; for the new child process, the return value is $0$. In the program, we can make use of this return value to determine if it is a child process or the parent process. The general framework is like this:

```
if(!fork()) {
    // child code
} else {
    // parent code
}
```

Usually in the child code, we want to terminate and return the child process after the job is finished; and in the parent code, we call `wait()`; when `wait()` returns, it means the child process has finished and returned ([man page](http://man7.org/linux/man-pages/man2/waitid.2.html)).

```pid_t wait(int *wstatus);```

Combining `fork()` with the ability to execute other program, we can create some powerful control flow. We will also see more about `fork()` usage with Inter Process Communication (IPC) in the next post.

### Execution of Another Program

In the program, we can let the current process to execute another program, by using [execl](http://man7.org/linux/man-pages/man3/exec.3.html) system call.

```int execl(const char *pathname, const char *arg, ..., (char *) NULL);```

Notice that the last argument has to be a `NULL` byte, to indicate the termination of the argument list.

This system call replaces the current process with a new process that starts upon the given pathname. See details on the [man page](http://man7.org/linux/man-pages/man2/execve.2.html). Simply said,
* It creates a new address space for the existing process (new text, data, BSS, stack...)
* It keeps the almost same Ublock and same PID, except some attributes such as:
  * Signal table is not preserved
  * Shared memory is detached

Basically the process that calls `execl` is thrown away, and replaced by a new process with almost the same Ublock and same PID.

The usual pattern is that, in the parent process, we call `fork()` and wait for child to finish, and in the child process, we call `execl()` to execute another program.

```
if(!fork()) {
    // child code; set up
    execl(path, argv1, ...);
} else {
    // parent code; set up
    wait();  // wait for child process to finish
    // post process
}
```
