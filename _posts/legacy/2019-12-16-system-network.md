---
layout: post
title: "System (8): Network and Socket"
date: 2019-12-16
categories: [System]
tags: [system, socket]
math: true
---

This article is part of my review notes of “Unix System Programming” course. In this post, we will go over the usage of **socket** in POSIX.

In previous posts, we discussed various Inter Process Communication (IPC) mechanism, including pipe, FIFO, message queue, shared memory. However, all these approaches only apply on a single machine. To achieve communication across machines, we need a standard protocol that enables sending data over network.

## TCP/IP

[TCP/IP](https://en.wikipedia.org/wiki/Internet_protocol_suite) is a suite of communication protocols that are used in Internet. It established a standard on how data should be packetized, addressed, transmitted, routed, and received over the network, providing a **stateful**, end-to-end communication mechanism.

Here we only focus on the socket part, without going through the entire TCP/IP.

## Socket

Socket is designed to work like a file descriptor, clean and simple, even though the underlying process can get very complex. In file-based IPC like pipe and FIFO, both read and write end has similar operation and use similar system call. However, in socket, we have the concept of **client** and **server**, which use a different set of system calls.

The system [call](http://man7.org/linux/man-pages/man2/socket.2.html) to get a socket is:

```int socket(int domain, int type, int protocol);```

* First argument: to specify which communication protocol to use. It supports about 20 protocols, and the most common protocols are `AF_INET` (IPv4) and `AF_INET6` (IPv6).
* Second argument: to specify the communication semantics. The common one is `SOCK_STREAM` (typical TCP), or `SOCK_DGRAM` (UDP).
* Third argument: to specify the sub communication protocol, and can be zero.

The client side to connect to the server:

1. `s = socket()` to create an endpoint and get the file descriptor
2. `connect(s, host_sockaddr, size_sockaddr)` to connect to the server, using server's internet address and port number
3. Use "s" just like a normal file descriptor to communicate with the server
4. `close(s)` to close the connection with server

The server side to connect to the client:

1. `s = socket()` to create an endpoint and get the file descriptor
2. `bind(s, self_sockaddr, size)` to bind the endpoint "s" to a local name and port
3. listen(s, max_connection) to start listening on client's connection
4. `fd = accept(s, ptr_client_sockaddr, ptr_addr_size)` to accept a client's connection and obtain a new file descriptor to interact with the client
5. `close(fd)` to close connection with the client
6. `close(s)` to stop listening

During the above process, there are some important things:
* We usually refer to a server by its hostname. To get the address needed for `connect()`, we can issue a system call: `struct hostent *gethostbyname(const char *name);`, which internally issues a query to DNS and get the IP address of the server.
* Some machines have little-endianness, and others have big-endianness. If we just send the binary (e.g. an integer) to each other and they have different endianness, the reading wouldn't make sense. Therefore, we need a standard byte order when sending data over internet. This can be achieved by four system calls. E.g., `uint32_t htonl(uint32_t hostlong);` will convert local byte order to standard byte order, and `uint32_t ntohl(uint32_t netlong);` will convert standard byte order to local byte order.
* For complex struct, it is best to define the serialization and deserialization strategy, to convert between struct and byte streams on any machines.

### Server Diagram

Usually a server can have connection to more than one client, which requires either multi-processing or multi-threading: upon accepting a client connection, the server should spin off a child process or thread to deal with the client connection.

However, since forking a child process can be quite expensive and not scalable, most higher level web servers such as Apache, Tomcat, make use of threads for clients' connection. "Thread pool size" is a common attribute to config a web server.

It is also possible to deal with connection without multiple processes or threads, when the task is not computational-heavy, and most of the time is spent on I/O. We can make use of this system [call](http://man7.org/linux/man-pages/man2/poll.2.html):

```int poll(struct pollfd *fds, nfds_t nfds, int timeout);```

* First argument: an array of file descriptors that have events of interests specified, such as `POLLIN` (fires off when the file descriptor has something to read)
* Second argument: the size of array
* Third argument: `poll()` will block if no events occur, and returns either an event occurs on any file descriptor, or time out

In this way, the server process will be blocked, and only spends CPU time when it needs to. Maybe it is better to study through actual code:

```c
/* Program to demonstrate server side of TCP communication */
/* There is one argument, the TCP port number to bind to */
/* This deemo illustrates using the POLL system call to monitor
   several fd's at one time in a single process */

#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <ctype.h>
#include <netdb.h>
//#include <stropts.h>
#include <poll.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main (argc,argv) 
	int argc;
	char *argv[];
{
	
	struct sockaddr_in sin; /* structure for socket address */
	int s;
	int fd;
	int len, i,num,count;
	struct pollfd pollarray[6];     /* up to 1+5 simultaneous connections*/
	int sum[6];						/* sum [1..5] for the 5 johns */
	int johns[6][40];				/*names of johns*/
	struct hostent *hostentp;

		/* set up IP addr and port number for bind */
	sin.sin_addr.s_addr= INADDR_ANY;
	sin.sin_port = htons(atoi(argv[1]));
        sin.sin_family= AF_INET;

		/* Get an internet socket for stream connections */
	if ((s = socket(AF_INET,SOCK_STREAM,0)) < 0) {
		perror("Socket");
		exit(1);
		}

		/* Do the actual bind */
	if (bind (s, (struct sockaddr *) &sin, sizeof (sin)) <0) {
		perror("bind");
		exit(2);
		}

		/* Allow a connection queue for up to 5 JOHNS */
	listen(s,5);

		/* Initialize the pollarray */
	pollarray[0].fd=s;     /* Accept Socket*/
	pollarray[0].events=POLLIN;
						  /* 5 possible john's */
	for (i=1;i<=5;i++) {pollarray[i].fd=-1;pollarray[i].events=POLLIN;}

	while(1) {
		poll(pollarray,6,-1);   /* no timeout, blocks until some activity*/

				/* Check first for new connection */
		if (pollarray[0].revents & POLLIN) {
			
			len=sizeof(sin);
			if ((fd= accept (s, (struct sockaddr *) &sin, &len)) <0) {
				perror ("accept");
					exit(3);
				}
					/* Find first available slot for new john */
			for (i=1;i<=5;i++) if (pollarray[i].fd == -1) break;
				pollarray[i].fd=fd;
				sum[i]=0;
				hostentp=gethostbyaddr((char *)&sin.sin_addr.s_addr,
					sizeof(sin.sin_addr.s_addr),AF_INET);
				strcpy((char *)&johns[i][0], hostentp->h_name);
			}

				/* If there are no new connections, process waiting john's */
		else for (i=1;i<=5;i++) {
			if ((pollarray[i].fd !=-1) && pollarray[i].revents) {
				count=read(pollarray[i].fd,&num,4);
				if (count==4) sum[i] +=ntohl(num);
				else {
					printf("John %d (%s) has sum %d\n",i,(char *)&johns[i][0], sum[i]);
					sum[i]=0;
					close(pollarray[i].fd);
					pollarray[i].fd = -1;
					}
				}
			}
		}
}
```
