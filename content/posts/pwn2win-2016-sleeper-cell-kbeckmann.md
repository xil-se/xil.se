+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-03-28T01:07:36+02:00"
description = ""
title = "Pwn2Win 2016 Sleeper Cell (Reverse) Writeup"

+++

# Problem

N/A

# Solution

We get an ELF 64-bit executable that we should reverse.

Running it gives us a `cat`-like interface; it echoes back what comes in on stdin.

Looking at the disassembly, I don't really feel like understanding it - let's do dynamic RE instead.

Entering a bunch of 'A's and stepping, I find interesting things happening:

```
[-------------------------------------code-------------------------------------]
   0x400e49:	lea    rsi,[rsp+0x10]
   0x400e4e:	mov    edi,0x6020e0
   0x400e53:	call   0x400d00 <_ZStrsIcSt11char_traitsIcESaIcEERSt13basic_istreamIT_T0_ES7_RSbIS4_S5_T1_E@plt>
=> 0x400e58:	mov    rdx,QWORD PTR [rax]
   0x400e5b:	mov    rdx,QWORD PTR [rdx-0x18]
   0x400e5f:	test   BYTE PTR [rax+rdx*1+0x20],0x5
   0x400e64:	jne    0x400f20
   0x400e6a:	lea    rsi,[rsp+0x10]
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe790 --> 0x601de0 --> 0x4011e0 (cmp    QWORD PTR [rip+0x200c18],0x0        # 0x601e00)
0008| 0x7fffffffe798 --> 0x4010ff (add    rsp,0x18)
0016| 0x7fffffffe7a0 --> 0x615598 ('A' <repeats 44 times>)
0024| 0x7fffffffe7a8 --> 0xff
0032| 0x7fffffffe7b0 --> 0x615548 ("DFHLNRTXDFLPRVBHJPTVBFLTXZDFJXBHJTVBHLRXZJLP")
0040| 0x7fffffffe7b8 --> 0x40171d (add    rbx,0x1)
0048| 0x7fffffffe7c0 --> 0x615608 ("DFHLNRTXDFLPRVBHJPTVBFLTXZDFJXBHJTVBHLRXZJLP")
0056| 0x7fffffffe7c8 --> 0x0
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x0000000000400e58 in ?? ()
gdb-peda$
```

I play around with this and it looks to be a simple rot() based on position. Too lazy, I just use the A-input as key.

Poking around, I find another string that looks like an encrypted flag: `FYM-OI}olte_zi_wdqedd_djrzuj_shgmEDFqo{`.

```
_ct  = "FYM-OI}olte_zi_wdqedd_djrzuj_shgmEDFqo{"
_key = "DFHLNRTXDFLPRVBHJPTVBFLTXZDFJXBHJTVBHLR"
key = [ord(x) - ord('A') for x in _key]

ct = _ct.upper()
pt = ""
for i in xrange(len(ct)):
    if '_{}'.find(ct[i]) >= 0:
        pt += ct[i]
        continue
    x = ord(ct[i]) - ord('A')
    x = (x - key[i] + 26) % (26)
    x += ord('A')
    pt += chr(x)
print pt
# CTFVBR}RIOT_IN_PUBLIC_SQUARE_VGZDLIEJD{
pt="CTF-BR{riot_in_public_square_vgzdLIEjd}"
```

Adjusting the case and braces returns the flag `CTF-BR{riot_in_public_square_vgzdLIEjd}`.
