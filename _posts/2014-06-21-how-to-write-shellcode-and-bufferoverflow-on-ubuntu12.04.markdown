---
layout: post
title:  "How to write shellcode and bufferoverflow on ubuntu12.04"
date:   2014-06-21 20:25:11
categories: 
---

This is a post relate to basic buffer overflow and how to write shellcode. The prerequisite for this is know how to write assembly languages and have basic idea about stack in computer system. 

First, I will talk about how to write shellcode. 

There are many methods to do this, one of them is described by Aleph One, you can have a look [here][alephone]. The second and one of the most common way is to write an `.asm` file and generate object code using `nasm`. I had a problem with this before, because I used to write `AT&T` syntax assembly code, stored as `.S` file and compile using `gcc`. 

Here's the [documentation][nasm_doc] for nasm.  

And now we can write our first shellcode that call `exit`, a system call. 
Here's the program
{% highlight ruby %}
	;; exit1.asm
	section .text
	global _start		;define the starting point of program
_start:	
        mov eax, 1       	; syscall number is 1 for eax
        mov ebx, 0		;setting ebx as zero
        int 0x80		;go to kernel mode
{% endhighlight %}

There're few points I want to specify.
<ul>
	<li>When we make a system call, <code>eax</code> store the system call number, first parameter will be stored in <code>ebx</code>, second in <code>ecx</code>, third in <code>edx</code>, fifth in <code>edi</code> and sixth in <code>ebp</code></li>
	<li><code>.text</code> section is used for keeping the actual code. This section must begin with the declaration <code>global _start</code>, which tells the kernel where the program execution begins</li>
	<li><code>.bss</code> section is used for declaring variables</li>
	<li><code>.data</code> section is used for declaring initialized data or constants. This data does not change at runtime.</li>
</ul>


[alephone]: http://www-inst.eecs.berkeley.edu/~cs161/fa08/papers/stack_smashing.pdf
[nasm_doc]: http://www.nasm.us/doc/
