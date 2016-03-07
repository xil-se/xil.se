+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-03-07T17:00:00+01:00"
description = ""
title = "Boston Key Party 2016 JIT In My Pants (Reverse) Writeup"

+++

# Problem

Jit in my pants - 3 - 38 solves : reversing: Because reversing an obfuscated jit'ed virtual machine for 3 points is fun!

# Solution

I downloaded it immediately when it was released and solved it really fast, but it turned out they uploaded the wrong binary. After a while they uploaded the proper binary and the fun could begin.

The binary takes the flag as argv[1]: `./c3803116bd70e802483d3bc4c4b564d2 BKPCTF{flag...}`.

We're provided with a binary with a strange loop and a call to a function pointer. Looks too weird to understand, so I gave up static analysis quickly.

The way I solved the easy binary was to just put a breakpoint in `write` because it prints `Nope.` when you enter the wrong flag. When the breakpoint hits, I scanned the memory for strings and could quickly find the BKPCTF flag because of the format.

But the new binary wasn't that easy. They encrypted the flag.

I put a breakpoint in `write` again, but this time I made a core dump at this point by invoking `generate-core-file` from GDB.

Then I loaded the core dump in a proper reverse engineering tool and could start analyzing it.

From GDB I already knew the address of the code that invoked write, but it turns out that this was coming from the libc flush. The backtrace wasn't interesting. I want to know who generated the text. After setting a couple of hardware write breakpoints using `watch` I tracked down the code that actually generates the output. I then continued the core dump analysis in this location.

After scanning though the assembly, I could see where the encrypted flag is put into the stack.

~~~
Encrypted flag: [FMTEPB}U3`_YEl0jj_hYcpp0emuYcv_Yu4Y355<w]
~~~

I then tracked down where argv[1] is accessed and could find the comparing algorithm. It is basically doing the following:

~~~
for i in len(userflag):
  if (chr(userflag[i]) ^ 0x05) - 1 != flag[i]:
    return 2
return 1
~~~

So I wrote a python one-liner that decrypts the flag:

~~~
s = "FMTEPB}U3`_YEl0jj_hYcpp0emuYcv_Yu4Y355<w"
''.join([chr((ord(c) + 1) ^ 0x5) for c in s])
~~~

This in turn reveals the flag: **BKPCTF{S1de_Ch4nnel_att4cks_are_s0_1338}**.

The flag of the easy binary was `BKPCTF{S1de_Ch4nnel_att4cks_are_3asier_than_st4tic_analySis}` for the record.
