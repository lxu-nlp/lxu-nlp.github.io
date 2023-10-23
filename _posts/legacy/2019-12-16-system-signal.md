---
layout: post
title: "System (9): Signal Handling in Traditional Unix and Modern POSIX"
date: 2019-12-16
categories: [System]
tags: [system]
math: true
---

This article is part of my review notes of “Unix System Programming” course. In this post we will be going over two signal handling approaches in Unix: a classic but deprecated way, and a modern POSIX way.

## Signal

A signal is an integer message delivered to a running process asynchronously. It can be originated from OS or any other processes. When a signal is delivered, the running process will be interrupted by the kernel, and starts to execute the corresponding handler.

Unix has a reserved set of signal number that corresponds to certain error status. E.g. SIGILL represents "illegal instruction", SIGQUIT means "quit from keyboard".

We can define custom handlers to process different signals. The only exception is SIGKILL and SIGSTOP, which themselves cannot be caught, blocked, or ignored.

Let's look at the modern POSIX signal handling first.

## POSIX Signal Handling

In modern POSIX, the Ublock in each process has a signal table (recall the previous post), which conceptually is a set of [signal/action] pair. For each signal in the reserved set, there is a corresponding default action that is usually "ignore" or "terminate" (see details on man page).

POSIX allows us to use the following system call to define a custom handler/action for a signal:

```int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);```

* First argument: the signal to handle
* Second argument: to provide the hander and signal mask
* Third argument: pointer to store previous action if any

Notice that in the second argument, we also provide a signal mask, and this is one big advantage over classic signal handling: when handling a certain signal, we can also define a binary signal mask, which will temporarily block all masked signals when this handler is running. Why this is necessary? This is because when a handler itself is running, another signal can still come in, and the kernel will interrupt the process and start to execute a new signal handler. This is usually not desired. POSIX solves this issue by using signal mask and blocking certain signals temporarily. Later, we will see how classic signal handling deals with this issue (a worse solution).

### Restore Process to A Valid Running State

We can do all kinds of things in a signal handler. One particular thing we can do is to restore the program to a previous valid point. For example, if a program will have zero divisor exception when the user input is `0`, then upon the process receiving SIGFPE (floating-point exception), we can make the process restore to a pre-defined running point when everything is good and ask for user input again.

These are the two system calls to achieve this behavior:

```int sigsetjmp(sigjmp_buf env, int savemask);```

```void siglongjmp(sigjmp_buf env, int val);```

When the process first encounters `sigsetjmp()`, it will take a snapshot of the current running state and stores everything, including the entire runtime stack and data segment, along with the current values in the registers, so that when the process later encounters `siglongjmp()`, it will be able to restore the values of all global and local variables, as well as the previous instruction offset.

### Example

Let's look at some code to have a concrete idea of POSIX signal handling.

```c
/* Program to illustrate the use of POSIX signals on UNIX 
		The program runs a computational loop to 
		compute perfect numbers starting at a fixed
		point.

	A time alarm signal is used to periodically print status
	An interrupt signal is used for status on demand
	A quit signal is used to reset the test interval (or terminate) */

#include <signal.h>
#include <errno.h>
#include <setjmp.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void perfect(int);

sigjmp_buf jmpenv; /* environment saved by setjmp*/

int n; /* global variable indicating current test point */

int main() {
	
	int begin; /* starting point for next search*/
		/* interrupt routines*/
	void status();
	void query();

	sigset_t mask;
	struct sigaction action;


	if (sigsetjmp(jmpenv,0)) {
		printf("Enter search starting point (0 to terminate): ");
		scanf("%d",&begin);
		if (begin==0) exit(0);
		sigprocmask(SIG_UNBLOCK, &mask, NULL);
		}
	else begin=2;
	
	/* Status Routine will handle timer and INTR */

	sigemptyset(&mask);
	sigaddset(&mask, SIGINT);
	sigaddset(&mask, SIGALRM);
	sigaddset(&mask, SIGQUIT);
	action.sa_flags=0;
	action.sa_mask=mask;
	
	action.sa_handler=status;
	sigaction(SIGINT,&action,NULL);
	sigaction(SIGALRM,&action,NULL);

	
	action.sa_handler=query;
	sigaction(SIGQUIT,&action,NULL);

	/* start alarm clock */
	alarm(20);
		perfect(begin);
		}

void perfect(start) 
	int start;
{
	int i,sum;

	n=start;

while (1) {
	sum=1;
	for (i=2;i<n;i++)
		if (!(n%i)) sum+=i;
    
	if (sum==n) printf("%d is perfect\n",n);
	n++;
	}
}

void status(signum)
	int signum;

{

	alarm(0); /* shutoff alarm */

	if (signum == SIGINT) printf("Interrupt ");
	if (signum == SIGALRM) printf("Timer ");

	printf("processing %d\n",n);

	alarm(20);	/*restart alarm*/
}	

void query() {siglongjmp(jmpenv,1);}
```

## Classic Signal Handling

We can also define a custom handler in classic Unix. However, the classic signal handling approach lacks signal mask. To overcome the issue that during the running of a signal handler, a new signal can still come in, resulting in a new signal handler gets executed unexpectedly, we need to manually take care of it when we write a handler.

The classic system [call](http://man7.org/linux/man-pages/man2/signal.2.html) to associate a handler with a signal is:

```sighandler_t signal(int signum, sighandler_t handler);```

In the handler, the first thing we want to do is to reset the signal action to SIG_IGN to ignore any further incoming signals. And the last thing is to reset the signal action back to this handler.

Not only we need to manual do this in every handler, there is also a race condition: a signal can still come in before the execution of the first reset (although with low probability). Therefore, this classic signal handling approach is flawed and deprecated.

### Example
```c
void status(int signum)
{
	if ( signal(signum, SIG_IGN) == SIG_ERR) {
		perror("Status SIG_IGN");
		exit(1);
	}

        // Do some handling

	if (signal(signum, status) == SIG_ERR) {
		perror("Status reset");
		exit(2);
	}
}
```
