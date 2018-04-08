---
title: Assembly Insight - 1
date: 2018-04-07
tags:
- Computer Science
- C
---
Just came across [this gem](https://www.youtube.com/watch?v=yOyaJXpAYZQ&t=0s&list=WL&index=48) when browsing cat videos on Youtube. After watching it I decided to run through the case by myself on my mac to see how a C program is working under the hood.

I want to be a bit geeky this time - using Vim(recently I've been learning some basic usage of Vim/Vi editor) to write raw C program, using command line to compile the program and also run the executable. Everything in the terminal.

# Compile and Run C Program

First, here is the C program for testing - the same as the testing case in the video. Save it as `fib.c`.

{% highlight c linenos %}
#include <stdio.h>
int main(void)
{
    int x, y, z;
    x = 0;
    y = 1;
    do
    {
        printf("%d\n", x);
        z = x + y;
        x = y;
        y = z;
    } while (x < 255);
}
{% endhighlight %}

Secondly, use `gcc` to compile the .c program - type command `gcc fib.c`. GCC is *GNU Compiler Collection, a compiler system produced by the GNU Project supporting various programming languages*. Also we can use `clang` - type command `clang fib.c`. Clang is *a compiler front end for the programming languages C, C++, Objective-C, Objective-C++, OpenMP, OpenCL, and CUDA, designed to act as a drop-in replacement for the GNU Compiler Collection.*

This will compile the program and generate `a.out` file. To run the executable we have to use command `./a.out`.

`a.out` is a file format used in older versions of Unix-like computer OS for executables, object code, and, in the later systems, shared libraries. This is an abbreviated form of *"assembler output"*. Also we can use `clang -o fib fib.c` to generate file `fib` instead of the default `a.out` :)

# Assembly Insight

Then use command `otool -tv fib` to observe the corresponding assembly code of our compiled C program. `otool` is *object file displaying tool*, `-t` is to print the text section and `-v` is to disassemble it to make it readable.

And we got:
{% highlight shell linenos %}
fib:
(__TEXT,__text) section
_main:
0000000100000f30        pushq   %rbp
0000000100000f31        movq    %rsp, %rbp
0000000100000f34        subq    $0x20, %rsp
0000000100000f38        movl    $0x0, -0x4(%rbp)

0000000100000f3f        movl    $0x0, -0x8(%rbp) # x = 0;
# movl (move long) moves value $0x0 into address offset in the memory -0x8 AKA x
# %rbp (stack base pointer)
0000000100000f46        movl    $0x1, -0xc(%rbp) # y = 1;

// start of loop
0000000100000f4d        leaq    0x5a(%rip), %rdi
0000000100000f54        movl    -0x8(%rbp), %esi
0000000100000f57        movb    $0x0, %al
0000000100000f59        callq   0x100000f8c # Call printf()

0000000100000f5e        movl    -0x8(%rbp), %esi
# x is moved into %esi register
0000000100000f61        addl    -0xc(%rbp), %esi
# y is added into %esi register
0000000100000f64        movl    %esi, -0x10(%rbp) # z = x + y;
# %esi is moved into z
0000000100000f67        movl    -0xc(%rbp), %esi
0000000100000f6a        movl    %esi, -0x8(%rbp) # x = y;
# y is moved into x
0000000100000f6d        movl    -0x10(%rbp), %esi
0000000100000f70        movl    %esi, -0xc(%rbp) # y = z;
# z is moved into y
0000000100000f73        movl    %eax, -0x14(%rbp)
# %eax contains the return value of the printf call
0000000100000f76        cmpl    $0xff, -0x8(%rbp) # while(x < 255);
# cmpl (compare long) compare x with 255($0xff)
0000000100000f7d        jl      0x100000f4d
# jump to address 0x100000f4d
0000000100000f83        movl    -0x4(%rbp), %eax
0000000100000f86        addq    $0x20, %rsp
0000000100000f8a        popq    %rbp
0000000100000f8b        retq
{% endhighlight %}

# Assembly Syntax Explanation

## movl vs movq

([Source](https://www.quora.com/What-is-the-difference-between-movq-and-movl-assembly-instruction)) When using [GAS(GNU Assembler)](https://en.wikipedia.org/wiki/GNU_Assembler) assembly language mnemonics for an Intel CPU, then:

* movq moves a quadword (64-bits) from source to destination.
* movl moves a long (32-bits) from source to destination.

Note that, in the Intel x86 world, a word is 16 bits, a doubleword is 32 bits (two 16-bit words), and a quadword is 64 bits (four 16-bit words).

GAS syntax uses these suffixes (e.g., q, l, b, etc.) on the instruction mnemonics to indicate how much data is being moved.

The Intel syntax doesn’t use suffixes, but instead uses the register name to deduce the amount of data, since each register has a known size. So, in Intel syntax, both instructions would use the same mnemonic: mov.

Also note that, in GAS syntax, the source operand appears first, and the destination operand appears second. This is the reverse order of the [Intel syntax](https://en.wikipedia.org/wiki/X86_assembly_language#Syntax), where the destination operand appears first, and the source operand appears second.

TBC
