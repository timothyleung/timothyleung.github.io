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

And now we are ready to generate byte code using the program we just wrote. The byte code is how the machine going to understand our program. For details, check out intel-sys in the intel website. We will generate byte code using `nasm` and `ld`. We first generate object file using `nasm` and then we create an executable file using `ld`, finally we run `objdump -d exec_file` to get the byte code.  

First run, 
{% highlight ruby %}
nasm -f elf -o exit.o exit.asm 
{% endhighlight %}

`-f` specifies the output format, for all output format, check the [documentation][output_format], in this case we are use `elf` which is executable and linkable format object. 

Then we will run, 
{% highlight ruby %}
ld -m elf_i386 -s -o exit exit.o # -m elf_i386 is for 64bits machine to create 32 bits executable 
{% endhighlight %}

And now we have a executable program called `exit`. We can now run 
{% highlight ruby %}
objdump -d exit
{% endhighlight %}
to obtain the byte code. 

{% highlight ruby %}
timothy@timothy-23-b020a:~/Desktop/shellcode$ objdump -d exit

exit:     file format elf32-i386


Disassembly of section .text:

08048060 <.text>:
 8048060:	b8 01 00 00 00       	mov    $0x1,%eax
 8048065:	bb 00 00 00 00       	mov    $0x0,%ebx
 804806a:	cd 80                	int    $0x80
{% endhighlight %}

The middle column is the byte code and we obtain our first shellcode!`\xb8\x01\x00\x00\x00\xbb\x00\x00\x00\x00\xcd\x80`. However, most of the exploit treat shellcode as string. And we can see a lot of null byte `\x00` inside our shellcode. The program will not be executed as we wish if we use this as shellcode. So the next thing we need to do is to get rid of all the null byte. But how? 

Here's some common tricks. 
<ul>
	<li><code>xor eax, eax</code> which is equivalent to <code>mov eax, 0</code></li>
	<li>Try to use the smaller register if possible.For example, <code>mov al, 10</code></li>
</ul>
There are more tricks for you to explore, but this two are pretty common and perfect for beginner.

So here's our revised version of `exit.asm`
{% highlight ruby %}
	;; exit2.asm
	section .text
	global _start		;define the starting point of program
_start:	
	xor eax, eax		; set eax as 0
	inc eax			;increment eax by 1
	xor ebx, ebx		;set ebx as 0
        int 0x80		;go to kernel mode
{% endhighlight %}

We can obtain the new shellcode by following the same steps. The new shellcode will be `\x31\xc0\x40\x31\xdb\xcd\x80`. It is now shorter and without null bytes! Hooray!

We will continue writing shellcode involve string and exec in next post. Then we can start our buffer overflow attack. 











[alephone]: http://www-inst.eecs.berkeley.edu/~cs161/fa08/papers/stack_smashing.pdf
[nasm_doc]: http://www.nasm.us/doc/
[output_format]: http://www.nasm.us/doc/nasmdoc7.html
