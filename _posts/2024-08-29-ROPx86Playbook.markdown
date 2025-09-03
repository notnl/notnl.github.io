---
layout: post
title:  "ROP x86 Playbook"
date:   2024-08-29 00:27:40 +0800
categories: jekyll update
---
This is a post that I have been wanting to make for a while, in the past 2 years ( 2022 - 2024 ) of my professional experience being involved in an IR team. 

I had the privilege of talking to industry professionals and they have encouraged me to explore certain areas in my downtime.

I had always been fascinated by Return-Oriented Programming (ROP), a technique used to craft the right context via malicious shellcode to exploit a system (x86). Thus,this is a post I will detail certain tips and tricks that I found very useful for myself that I would love to share. This also doubles as a revision for EXP-301.

Note that these are not advanced especially for experienced professionals, some might even seem fairly obvious, some are what I have learned from EXP-301 course content(not copied over obviously), just things that I had to do to overcome certain limitations(or rather, forced to use a certain gadget).

---


# **Realigning ROP Chain after a Call Gadget**

Under limited circumstance where I had to utilize a gadget with call instruction ,
Because a call instruction pushes the return address onto the stack, now my ROP Chain is broken.

Thus, for realignment purposes, I could simply point it towards a POP instruction,especially when it is calling indirectly via a register.


The problem:

#### Your forced to use this gadget to assign to the address at esi

{% highlight ruby %}


mov [esi], eax; call edi;  



{% endhighlight %}

Now we have to find a gadget that allows us to restore the ROP chain after call edi;

Since ESP -= 0x4 

![Alt text](https://i.postimg.cc/v8ngvNxm/Screenshot-2025-09-03-235010.png "a title")


Just find a pop gadget

![Alt text](https://i.postimg.cc/VkMjrhfn/Screenshot-2025-09-03-231538.png "a")


Assign EDI to that address

![Alt text](https://i.postimg.cc/tTRPcChN/Screenshot-2025-09-03-231609.png "a title")

Now we have realigned our stack 





---

#  **Adding Addresses**

Lets say eax =  Stack Address, ecx = offset


if we want to 

{% highlight ruby %}
add eax, ecx;

{% endhighlight %}

Because of null bytes present at an arbitrary value such as 0x105 = 0x00000105 (for x86, 4 bytes), we can't just simply set this value to our stack

In python, we cannot simply just 
ropchain += pack("<L",0x105) 


Going through the course content for EXP-301, they gave many ideas to overcome this.
Easiest method is finding the signed 2's complement


{% highlight ruby %}
0x0 - 0x105 = 0xFFFFFEFB

ropchain += pack("<L",0xFFFFFEFB) 


{% endhighlight %}

then simply find gadgets that does these instructions

{% highlight ruby %}

 pop ecx; ret;
 neg ecx; ret;  // This converts 0xFFFFFEFB to 0x105
 add eax, ecx;

{% endhighlight %}


#### What if we can't find a way to negative the register? And we only had add eax, ecx; and pop ecx;?

We can do a calculating trick 

For example : 

{% highlight ruby %}

0x10102315 +  0x100   =  0x10102415

{% endhighlight %}

But we can't send 0x100 to the buffer due to null bytes

0x100 can be obtained via adding and causing an overflow


{% highlight ruby %}
0x100 = (0x77777777 + 0x88888888) + 0x101 ( Our target value, which is 0x100 + 0x1 . Why + 0x1? Check below)


{% endhighlight %}

Since 0x101 can't be passed to the buffer, due to null bytes at 0x00000101 

We Add 0x101 to either 0x77777777 or 0x88888888  


{% highlight ruby %}
0x88888888 + 0x101 = 0x88888989


{% endhighlight %}
Then 


{% highlight ruby %}
(0x77777777 + 0x88888989) = 0x00000100 (The overflow is ignored)

{% endhighlight %}

How does this calculation work?

{% highlight ruby %}

	0111 0111 0111 0111 0111 0111 0111 0111 (0x77777777) + 
	1000 1000 1000 1000 1000 1000 1000 1000 (0x88888888)
=       1111 1111 1111 1111 1111 1111 1111 1111  (0xFFFFFFFF)  

{% endhighlight %}


Now that 0xFFFFFFFF = -1 in 2's complement	
To get a desired number lets say 0x100, take the value 0x100 then add by 0x1, since we start from -1, = 0x100+0x1 = 0x101

We don't need to do this if we are able to find the proper gadgets, this is only IF you are limited to using an add gadget and you need to +0x100 from it.





#  **Replace mov Gadgets**

#### Lets say we want to mov eax, ecx; However, there are no such instructions that can be used easily. What gadgets can we use to replace this mov instruction?

Exchange gadgets

{% highlight ruby %}
xchg eax, ecx;

{% endhighlight %}

Add gadgets, but make sure eax is zero-ed

{% highlight ruby %}
xor eax, eax; 
add eax, ecx;

{% endhighlight %}

Sub gadgets, but make sure eax is zero-ed


{% highlight ruby %}
neg ecx; 
sub eax, ecx; 


{% endhighlight %}

# **Truncate Found ROP gadgets**

If the gadget is very lengthy, you could remove all instructions before by simply pointing to it.
But we have to make sure that the new location does not contain 'bad bytes'


Look at the length of these gadget:

![Alt text](https://i.postimg.cc/bvNT4hf8/Screenshot-2025-09-04-000851.png "a title")

What if we only want the later part of it?

[![Screenshot-2025-09-04-001054.png](https://i.postimg.cc/DwYJykb7/Screenshot-2025-09-04-001054.png)]

In this case we want the last 2 instructions, we can simply just use the address

**0x100d2ba2** 



---
