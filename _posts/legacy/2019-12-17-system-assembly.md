---
layout: post
title: "Security (1): Assembly Review"
date: 2019-12-17
categories: [System]
tags: [system, assembly]
math: true
---

This article is part of my review notes of "Computer Security" course by Ymir Vigfusson. We can find course recording from previous years on his YouTube [channel](https://www.youtube.com/watch?v=KwJyKmCbOws&t=21).

This series assumes some background knowledge in C, Assembly, and Unix operating system.

## x86 Assembly

In this series, we will be using AT&T syntax. One big difference between AT&T and Intel syntax is that, in an instruction, the source operand is on the left, and the destination operand is on the right. E.g. `(movl $2, %eax)` means move `2` to the register `eax`.

### Registers and Stack

Registers directly interact with the CPUs. They are used to store variables specified by the machine code instruction, and also store the address of the current frame on the runtime stack.

There are some general purpose registers and special registers.

![assembly](/assets/img/legacy/am1.png)

Registers (from Course Slide)

Specifically,
* `esp` points to the "top" of the current stack frame
* `ebp` points to the "bottom" or "base" of the current stack frame
* `eip` points to the next instruction in the process

In Unix system, the stack of a process grows **downward**. Therefore, the stack "top" actually has a smaller value of the address, and the stack "base" has a larger value of the address.

### Memory Addressing Mode

In assembly, since we can only directly access registers, we need a reference to access variables in memory. Therefore, we have the following format for memory addressing:

```D(Rb, Ri, S) ==> Mem[Reg[Rb] + S * Reg[Ri] + D]```

E.g. 0x80(%edx, %ecx, 4) represents the value on memory address `(%edx + 4 * %ecx + 0x80`).

Two common commands are used to move variables back and forth between registers and memory:
* `movl src, dest`: move value from source to destination. If "src" or "dest" is in memory addressing mode, then the dereferenced value will be used; otherwise, the direct value from the register will be used.
* `leal src, dest`: similar to `movl`. However, in `leal`, the "src" value will not be dereferenced.

### Arithmetic Instructions

Some most common two operand instructions:
* `addl src, dest: dest += src`
* `subl src, dest: dest -= src`
* `xorl src, dest: dest = dest XOR src`
* `sall src, dest: dest = dest << src`

Some most common one operand instructions:
* `incl dest: dest += 1`
* `decl dest: dest -= 1`
* `negl dest: dest = -dest`

### Condition Control

In assembly, the comparison is done by the following steps:
* `(coml b, a)` will execute `(a - b)` using arithmetic instruction
* The arithmetic instruction (e.g. `addl` or `subl`) always implicitly sets certain flags about the result (see picture below)
* `setX` instruction is used to set single byte based on the combinations of flags
* Use this single byte accordingly: continue current execution, or jump to a different instruction offset

![assembly](/assets/img/legacy/am2.png)
![assembly](/assets/img/legacy/am3.png)

### Loops

Loops are implemented by "goto instruction_addr" command in assembly (similar to "goto" statement in C).

### Procedure Control

How does the registers keep the information of different stack frames? Here is an overview of the process:

* Procedure call: triggered by (call function_addr)
  * Push the address of next instruction on stack
  * jump to the address of the function
* Procedure execution:
  * Set up current stack frame: (push %ebp), (movl %esp, %ebp); the stack looks like: old ebp (addr at %ebp), old eip (addr at 4(%ebp)), arg1 (addr at 8(%ebp)), arg2...
  * Execute function
  * Restore stack frame right before leaving: pop %ebp
* Prodecure returning: ret
  * Pop back the address of next instruction from stack to eip
  * Jump to eip

Note: no special care for esp. esp will auto change by push or pop.

## Example

Assembly language itself is a course, and it is out of the scope to go over the details of assembly in this post. Let's just look at a simple example to refresh our memory.

![assembly](/assets/img/legacy/am4.png)

In the next post, we will challenge some more demanding assembly problems. 

