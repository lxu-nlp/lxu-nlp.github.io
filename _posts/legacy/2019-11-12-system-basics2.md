---
layout: post
title: "System (2): Basic Concept Review (Program, Process, libc)"
date: 2019-11-12
categories: [System]
tags: [system]
math: true
---

This article is part of my review notes of "Unix System Programming" course. In this post, we go over the basic concepts of program, process, and libc.

## Program

A program is an executable binary file in [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) format. Executable and Linkable Format (**ELF**) is a standard format for executable that works across multiple OS, and it encapsulates all the data resources of the program.

ELF format has:

* file header, program header: specifies metadata such as the instruction architecture, big or little endianness, etc.
* section header: specifies metadata about each section below, such as size
* sections: there are four data sections
  * Text: the object code of the actual executable instructions
  * Data: global or static variables with initial values
  * BSS: global or static variables that uninitialized or initialized to zero
  * Symbols

Unix command `file` can be used to read ELF header.
Unix command `size` can be used to read section sizes.

## Process

When we run a program, a process is created by OS, which is the environment in which the program executes.

When a program starts, OS will:
* Allocate memory space that is large enough to load the address space from ELF file
* The address space initially contains the following parts (from low to high address):
  * Text, Data, BSS: copied from the program
  * Stack, Heap, Shared Memory: runtime environment for local variables, dynamic memory allocation, shared memory addresses
  * Ublock: meta-information about this process that OS needs to manage over
* Set initial instruction pointer to the address of main (the main function is the entry of any C program).

The Ublock includes (the following is not a complete list):
* UID, GID: user, group ID
* EUID, EGID: effective user, group ID
* SUID, SGID: saved user, group ID
* Current working directory
* Timestamps
* PID: unique identifier of the process
* PPID: parent's PID)
* Open file table: a table that maintains file descriptors
* Signal table

The whole list of every process is visible to the OS.

### Permission

In the Ublock, there are at least two IDs for the user: UID and EUID.
* UID: the real user ID, or the login user ID; it is the user ID that executes the program
* EUID: effective user ID, which can be UID or others

A user can increase its access level by escalating EUID to a more privileged user ID. A common scenario is the `sudo` command.

The system call to set EUID is `setuid()`. When the SetUID bit is set, the EUID is the user ID of the program's owner.

### Open File Table

Open file table is maintained in each process. If there are any child processes, then the open file table is shared between parent process and all the child processes. Any changes made by child are reflected in parent, and vice versa.

For each opened file in the process, there will be a corresponding file descriptor. A file descriptor is a data structure that describes a file in use in a process, including the inode number, flags, and current offset.

Each entry of the open file table is conceptually a pair of file descriptor and its reference count. Usually the count would be $1$. The count gets incremented if a child process is created. Note that there can also be more than one file descriptors for the same file, if the same file is opened by system call more than once.

### Process Parameters

How do we pass any arguments or parameters to the program? There are two ways: explicit or implicit.
* Explicit: pass arguments like `program argv1 argv2 ...` when we start the program, then use them by `char* argv[]` in the program.
* Implicit: the program can access and use any environment variables. Environment variables are a string of key-value pair, and will be on the runtime stack. We can define the environment variables before we start the program.

## System Interface

System call is the main API to interact with the system. In Unix, we have a C library called "libc" or "glibc" that implements POSIX. Here are some facts about "libc":
* Most implementation in "libc" is written in assembly language, and a small portion is written in C language. Therefore, "libc" itself is system-dependent. However, the usage is "portable" since the interface is unified by POSIX.
* Most system calls are implemented using a trap instruction. A trap instruction is used to proactively switch from user mode to kernel mode. Therefore, most system calls involve context switch.
* When the context switch happens, OS saves the current instruction pointer and stack pointer, and switch to kernel mode to proceed with system call. Therefore, system call has access to all memory and I/O instructions because of kernel mode. By contrast, user mode doesn't have access to any I/O; without system call, all a process can do is to do computation within its address space.

The header file of "libc" is in `/usr/include`.

### Error Handling

* When a system call encounters error, it always return $-1$.
* There is a global variable called "errno". When a system call returns error, `errno` will be set to a certain value to indicate the type of error.
* `perror()` can be used to print the generic phrase for the `errno`.
