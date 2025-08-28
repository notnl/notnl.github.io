---
layout: post
title:  "ROP x86 Playbook"
date:   2024-08-29 00:27:40 +0800
categories: jekyll update
---
This is a post that I have been wanting to make for a while, in the past 2 years ( 2022 - 2024 ) of my professional experience being involved in an IR team. 

I had the privilege of talking to industry professionals and they have encouraged me to explore certain ares in my downtime.

I had always been fascinated by Return-Oriented Programming (ROP), a technique used to craft the right context via malicious shellcode to exploit a system. Thus,this  is a post I will detail certain tips and tricks that I found very useful for myself that I would love to share. This also doubles as a revision for EXP-301.

---


# **Exchange Gadgets**

Exchange

{% highlight ruby %}
xchg eax, ecx;



{% endhighlight %}

---
