---
layout: post
title: "Security (2): Assembly Exercise"
date: 2019-12-17
categories: [System]
tags: [system, assembly]
math: true
---

This article is part of my review notes of “Computer Security” course by Ymir Vigfusson. We can find course recording from previous years on his YouTube [channel](https://www.youtube.com/watch?v=KwJyKmCbOws&t=21).

In the previous post, we reviewed the very basic of x86 assembly language. Let's do some exercise. The main objective here is to do decompiling. Because in real-world security tasks, we don't get to see the source code to find the vulnerabilities. More often, we have access to the binary executable file, and we can use GDB debugger to look at the machine instructions in assembly. Therefore, decompiling is an important skill.

The following problem is from [adversary.io](https://platform.adversary.io/missions/bomblab). I will provide my solution in the end as reference.

## Problem

Task: the evil program will ask for user input and decide whether to execute a bomb program. Part of the source code is given. We want to know the decision logic and provide the correct input to defuse the bomb.

Source code (decision logic is hidden):

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

void defuse(char *phrase) {
	printf ("DEFUSING BOMB...\n")
	// Try to cut these wires in the REAL bomb ...
	if (fork() == 0)
		execl("/home/drevil/realbomb", phrase, NULL);
	exit(-1);
}

void explode_bomb()
{
	// Cut the wrong wire...
	defuse ("BOOM!") 
}


void wiring(char *input) {
 	// CLASSIFIED TOP SECRET//NOFORN
	// ...
}


// Read one string from standard input
char *read_string()
{
	static char buffer[256];
	fgets (buffer, stdin, NULL);
	buffer[strlen(buffer)-1] = '\0'; // Remove newline
	return (buffer);
}


int main(int argc, char **argv) {
	char *phrase = NULL;

	printf ("\033[1;36m[[[\033[0m MOST EVIL BOMB v13.38 -- \033[1;37mThis one will work.\033[1;36m]]]\033[0m\n");
	printf ("Enter magic phrase:\n");

	phrase = read_string();	

	wiring (phrase);

	return 0;
}
```

As we can see, the `wiring()` is hidden to us. We want to use GDB debugger to look at its assembly code:

```
gdb$ disass wiring
Dump of assembler code for function wiring:
   0x08048944 <+0>:     push   %ebp
   0x08048945 <+1>:     mov    %esp,%ebp
   0x08048947 <+3>:     push   %edi
   0x08048948 <+4>:     push   %esi
   0x08048949 <+5>:     push   %ebx
   0x0804894a <+6>:     sub    $0x1c,%esp
   0x0804894d <+9>:     call   0x8048570 <__x86.get_pc_thunk.bx>
   0x08048952 <+14>:    add    $0x16ae,%ebx
   0x08048958 <+20>:    mov    0x8(%ebp),%esi
   0x0804895b <+23>:    lea    -0x20(%ebp),%eax
   0x0804895e <+26>:    push   %eax
   0x0804895f <+27>:    lea    -0x1c(%ebp),%eax
   0x08048962 <+30>:    push   %eax
   0x08048963 <+31>:    lea    -0x1561(%ebx),%eax
   0x08048969 <+37>:    push   %eax
   0x0804896a <+38>:    push   %esi
   0x0804896b <+39>:    call   0x8048520 <sscanf@plt>
   0x08048970 <+44>:    add    $0x10,%esp
   0x08048973 <+47>:    cmp    $0x2,%eax
   0x08048976 <+50>:    je     0x804897d <wiring+57>
   0x08048978 <+52>:    call   0x8048719 <explode_bomb>
   0x0804897d <+57>:    cmpl   $0x7,-0x1c(%ebp)
   0x08048981 <+61>:    jle    0x80489d3 <wiring+143>
   0x08048983 <+63>:    cmpl   $0x0,-0x20(%ebp)
   0x08048987 <+67>:    js     0x80489d3 <wiring+143>
   0x08048989 <+69>:    mov    -0x1c(%ebp),%edi
   0x0804898c <+72>:    sub    $0x8,%esp
   0x0804898f <+75>:    lea    -0x1(%edi),%eax
   0x08048992 <+78>:    push   %eax
   0x08048993 <+79>:    push   %edi
   0x08048994 <+80>:    call   0x804887f <fun>
   0x08048999 <+85>:    add    $0x10,%esp
   0x0804899c <+88>:    test   %eax,%eax
   0x0804899e <+90>:    je     0x80489ba <wiring+118>
   0x080489a0 <+92>:    push   $0x20
   0x080489a2 <+94>:    push   $0x0
   0x080489a4 <+96>:    pushl  -0x20(%ebp)
   0x080489a7 <+99>:    lea    0x60(%ebx),%eax
   0x080489ad <+105>:   push   %eax
   0x080489ae <+106>:   call   0x80488be <bliss>
   0x080489b3 <+111>:   add    $0x10,%esp
   0x080489b6 <+114>:   cmp    %eax,%edi
   0x080489b8 <+116>:   je     0x80489bf <wiring+123>
   0x080489ba <+118>:   call   0x8048719 <explode_bomb>
   0x080489bf <+123>:   sub    $0xc,%esp
   0x080489c2 <+126>:   push   %esi
   0x080489c3 <+127>:   call   0x804863b <defuse>
   0x080489c8 <+132>:   add    $0x10,%esp
   0x080489cb <+135>:   lea    -0xc(%ebp),%esp
   0x080489ce <+138>:   pop    %ebx
   0x080489cf <+139>:   pop    %esi
   0x080489d0 <+140>:   pop    %edi
   0x080489d1 <+141>:   pop    %ebp
   0x080489d2 <+142>:   ret
   0x080489d3 <+143>:   call   0x8048719 <explode_bomb>
   0x080489d8 <+148>:   jmp    0x8048989 <wiring+69>
End of assembler dump.
```

The wiring itself is not easy already. What's worse is that it also calls other two unknown functions: `fun` and `bliss`. Let's also GDB them.

```
gdb$ disass fun
Dump of assembler code for function fun:
   0x0804887f <+0>:     push   %ebp
   0x08048880 <+1>:     mov    %esp,%ebp
   0x08048882 <+3>:     sub    $0x8,%esp
   0x08048885 <+6>:     mov    0xc(%ebp),%ecx
   0x08048888 <+9>:     cmp    $0x1,%ecx
   0x0804888b <+12>:    je     0x80488b3 <fun+52>
   0x0804888d <+14>:    test   %ecx,%ecx
   0x0804888f <+16>:    jle    0x80488b7 <fun+56>
   0x08048891 <+18>:    mov    0x8(%ebp),%eax
   0x08048894 <+21>:    cltd
   0x08048895 <+22>:    idiv   %ecx
   0x08048897 <+24>:    mov    %edx,%eax
   0x08048899 <+26>:    test   %edx,%edx
   0x0804889b <+28>:    jne    0x804889f <fun+32>
   0x0804889d <+30>:    leave
   0x0804889e <+31>:    ret
   0x0804889f <+32>:    sub    $0x8,%esp
   0x080488a2 <+35>:    sub    $0x1,%ecx
   0x080488a5 <+38>:    push   %ecx
   0x080488a6 <+39>:    pushl  0x8(%ebp)
   0x080488a9 <+42>:    call   0x804887f <fun>
   0x080488ae <+47>:    add    $0x10,%esp
   0x080488b1 <+50>:    jmp    0x804889d <fun+30>
   0x080488b3 <+52>:    mov    %ecx,%eax
   0x080488b5 <+54>:    jmp    0x804889d <fun+30>
   0x080488b7 <+56>:    mov    $0x0,%eax
   0x080488bc <+61>:    jmp    0x804889d <fun+30>
End of assembler dump.
```

```
gdb$ disass bliss
Dump of assembler code for function bliss:
   0x080488be <+0>:     push   %ebp
   0x080488bf <+1>:     mov    %esp,%ebp
   0x080488c1 <+3>:     push   %edi
   0x080488c2 <+4>:     push   %esi
   0x080488c3 <+5>:     push   %ebx
   0x080488c4 <+6>:     sub    $0xc,%esp
   0x080488c7 <+9>:     mov    0x8(%ebp),%ebx
   0x080488ca <+12>:    mov    0xc(%ebp),%esi
   0x080488cd <+15>:    mov    0x10(%ebp),%edx
   0x080488d0 <+18>:    mov    0x14(%ebp),%ecx
   0x080488d3 <+21>:    mov    %ecx,%edi
   0x080488d5 <+23>:    sub    %edx,%edi
   0x080488d7 <+25>:    mov    %edi,%eax
   0x080488d9 <+27>:    shr    $0x1f,%eax
   0x080488dc <+30>:    add    %edi,%eax
   0x080488de <+32>:    sar    %eax
   0x080488e0 <+34>:    add    %edx,%eax
   0x080488e2 <+36>:    cmp    %edx,%ecx
   0x080488e4 <+38>:    jl     0x804892f <bliss+113>
   0x080488e6 <+40>:    cmp    %edx,%ecx
   0x080488e8 <+42>:    je     0x8048909 <bliss+75>
   0x080488ea <+44>:    cmp    (%ebx,%eax,4),%esi
   0x080488ed <+47>:    jg     0x8048916 <bliss+88>
   0x080488ef <+49>:    push   %eax
   0x080488f0 <+50>:    push   %edx
   0x080488f1 <+51>:    push   %esi
   0x080488f2 <+52>:    push   %ebx
   0x080488f3 <+53>:    call   0x80488be <bliss>
   0x080488f8 <+58>:    add    $0x10,%esp
   0x080488fb <+61>:    test   %eax,%eax
   0x080488fd <+63>:    js     0x8048936 <bliss+120>
   0x080488ff <+65>:    add    %eax,%eax
   0x08048901 <+67>:    lea    -0xc(%ebp),%esp
   0x08048904 <+70>:    pop    %ebx
   0x08048905 <+71>:    pop    %esi
   0x08048906 <+72>:    pop    %edi
   0x08048907 <+73>:    pop    %ebp
   0x08048908 <+74>:    ret
   0x08048909 <+75>:    cmp    %esi,(%ebx,%eax,4)
   0x0804890c <+78>:    setne  %al
   0x0804890f <+81>:    movzbl %al,%eax
   0x08048912 <+84>:    neg    %eax
   0x08048914 <+86>:    jmp    0x8048901 <bliss+67>
   0x08048916 <+88>:    push   %ecx
   0x08048917 <+89>:    add    $0x1,%eax
   0x0804891a <+92>:    push   %eax
   0x0804891b <+93>:    push   %esi
   0x0804891c <+94>:    push   %ebx
   0x0804891d <+95>:    call   0x80488be <bliss>
   0x08048922 <+100>:   add    $0x10,%esp
   0x08048925 <+103>:   test   %eax,%eax
   0x08048927 <+105>:   js     0x804893d <bliss+127>
   0x08048929 <+107>:   lea    0x1(%eax,%eax,1),%eax
   0x0804892d <+111>:   jmp    0x8048901 <bliss+67>
   0x0804892f <+113>:   mov    $0xffffffff,%eax
   0x08048934 <+118>:   jmp    0x8048901 <bliss+67>
   0x08048936 <+120>:   mov    $0xffffffff,%eax
   0x0804893b <+125>:   jmp    0x8048901 <bliss+67>
   0x0804893d <+127>:   mov    $0xffffffff,%eax
   0x08048942 <+132>:   jmp    0x8048901 <bliss+67>
End of assembler dump.
```

In order to provide the correct input, we need to decompile the algorithm used in `wiring()`, `fun()` and `bliss()`.

If you have looked at `fun()` and `bliss()` carefully, you will notice they are also recursive functions! This will make it very hard to just guess the correct input without knowing the exact algorithm.

I don't think I can write a clear explanation here in this post (maybe better with a video). So I will just skip the analysis and decompiling process. My solution is provided below.

## Solution

```c
int fun(int a, int b) {
	if(a == 1) eax = 1 and return;
	if(a == 0) eax = 0 and return;

	int c = b;
	int d = c % a;
	if(d == 0)  return;
	else {
		fun(a - 1, b);
	}
}
```


```
----
args: addr, 0, 0x20
ebx: addr of array
esi: 2nd input
edx: 0
ecx: 0x20

edi: 1st input

eax = ((leftmost bit of (ecx - edx) + 1st input) / 2) + edx


-------------------------------

int first = FIRSTINPUT that > 7;

bliss(int* a, int b, int c, int d) {
	// Initially, b = SECONDINPUT, c = 0, d = 0x20

	// int e = ((leftmost bit of (d-c) + c) / 2) + c;
	b = d - c;
	int e = c / 2 + c;
	if(d < c) 
		return -1;
	if (d == c) {
		if(b != a[e])
			return 0;
		else
			return 1;
	}

	if(b > a[e]) {
		inr r = bliss(a, b, e+1, d);
		if(r < 0)
			return -1;
		else
			return 2 * r + 1;
	} else {
		int r = bliss(a, b, c, e);
		if(r < 0)
			return -1;
		else
			return 2 * r;
	}
}
```
