+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-06T22:02:14+01:00"
title = "SharifCTF 2016 Serial (RE 150) Writeup"
+++

# Problem

> Run and capture the flag!

Points: 150

Solved by 110 team(s)

# Solution

We are provided with a x86-64 linux executable:
~~~
rgg: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.26, BuildID[sha1]=77e92e8b1bd4f26641bab4dbf563037a7b9538d2, not stripped
~~~

The binary isn't very big but looks funny in various decompilers and disassemblers.

The code does a lot of unaligned jumps and fills the section in between with junk, so disassemblers get really confused.

~~~asm
0x0000000000400a83 <+231>:	add    esi,edi
0x0000000000400a85 <+233>:	(bad)  
0x0000000000400a86 <+234>:	(bad)  
0x0000000000400a87 <+235>:	cmp    al,0x5a
0x0000000000400a89 <+237>:	je     0x400a99 <main+253>  <------- jump to 0x400a99
0x0000000000400a8b <+239>:	mov    ax,0x5eb
0x0000000000400a8f <+243>:	xor    eax,eax
0x0000000000400a91 <+245>:	je     0x400a8d <main+241>
0x0000000000400a93 <+247>:	call   0x41ed81
0x0000000000400a98 <+252>:	add    BYTE PTR [rdi],cl

   0x400a99 should be here but it doesn't even register in gdb

0x0000000000400a9a <+254>:	mov    dh,0x85
0x0000000000400a9c <+256>:	add    esi,edi
~~~

I gave up the statical analysis and ran the binary instead which was a fun experience.

Using paper and pen I tracked what was going on.

- Input must be 16 bytes
- Read str[ 0], must be equal to 'E'
- Read str[15], add with str[0], sum must be 0x??
- Read str[ 1], must be equal to 'Z'
- Read str[14], add with str[1], sum must be 0x??
- Repeat until whole string has been read.

The flag is the correct input, which is `EZ9dmq4c8g9G7bAV`.

I really enjoyed this challenge because I had to stop and actually think, take notes, and not just quickly find the flag somewhere.

*Flag is EZ9dmq4c8g9G7bAV*
