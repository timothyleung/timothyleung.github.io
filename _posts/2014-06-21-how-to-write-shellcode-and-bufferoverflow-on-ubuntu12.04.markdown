---
layout: post
title:  "How to write shellcode and bufferoverflow on ubuntu12.04"
date:   2014-06-21 20:25:11
categories: 
---

This is a post relate to basic buffer overflow and how to write shellcode. The prerequisite for this is know how to write assembly languages and have basic idea about stack in computer system. 

First, I will talk about how to write shellcode. 

There are many methods to do this, one of them is described by Aleph One, you can have a look [here][alepheone]. The second and one of the most common way is to write an `.asm` file and generate object code using `nasm`. I had a problem with this before, because I used to write `AT&T` syntax assembly code, stored as `.S` file and compile using `gcc`. 

Here's the [documentation][nasm_doc] for nasm.  

And now we can write our first shellcode that call `exit`, a system call. 
Here's the program
{% highlight ruby %}
;exit.asm
[SECTION .text]
global _start
_start:
        xor eax, eax       ;exit is syscall 1
        mov al, 1       ;exit is syscall 1
        xor ebx,ebx     ;zero out ebx
        int 0x80
{% endhighlight %}

[alephone]: http://www-inst.eecs.berkeley.edu/~cs161/fa08/papers/stack_smashing.pdf

[nasm_doc]: http://www.nasm.us/doc/