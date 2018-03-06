---
layout: post
title: Fuzzing Fun
tags: AFL fuzzing blaze bourbon
---

The [American Fuzzy Lop](http://lcamtuf.coredump.cx/afl/) is an interesting beast. It feeds on power and free processor cores and poops crashes. 

Babby's first fuzzing is actually pretty easy to get into. Find a program that can be compiled with some sort of `make` or `cmake` style build script and switch out the compiler with AFL's version of either `gcc` or `clang` (if you are feeling adventurous). After that, it's only a couple commandline options until you have a working fuzzing harness of simple programs.

I know there are more complex examples, but that's less fun (for now).

And for the love of all things good in this world, don't try to compile VLC from source (with AFL). I'm serious. If you value your sanity, don't. 

A few months ago, I threw AFL at `gdb`. Specifically, `gdb` loading a binary and then doing nothing with it (quitting immediately). As (manual) post processing, I made sure that the binary still executed as expected. The test "Hello, World" style binary had a function call, a couple libc calls, and a call to a vDSO syscall -- enough to be interesting but minimal enough to be debuggable. It was also stripped.

Recently, I did a similar thing for `objdump -d -Mintel`. `readelf` is next.

I have repeatable crashes for both -- after about a day of fuzzing. I have Linux x64 binaries that run correctly but crash these utilities.

I threw three cores at this (i7-6700K), but the AFL "deterministic" instance has been the most fruitful. I actually probably have multiple repeatable crashes for both programs. I need to get better postprocessing scripts. That's where the real work is with fuzzing.

I'll probably release details eventually. As for now, my goal is to wreak havoc on competitors of [Blaze CTF](https://ctftime.org/ctf/129). Assuming that the organizers will let me make challenges. I know where some of them live, though, so I should be able to manage. Stretch goals: find bugs in Binary Ninja, IDA, QIRA, etc....

Also, is something like this impactful enough for a CVE? Todo: crash triage and figure out guidelines for CVE submission. 

Today's post is brought to you by bourbon. 
