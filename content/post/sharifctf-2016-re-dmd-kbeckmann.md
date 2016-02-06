+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-06T21:25:14+01:00"
title = "SharifCTF 2016 dMd (RE 50) Writeup"
+++

# Problem

> Attachment: A data blob
>
> Flag is : The valid input

Points: 50

Solved by 243 team(s)

# Solution

We are provided with a x86-64 linux binary:
~~~
dMd: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=2643fecd383362fe9593ef8605a9ce882a85a38a, not stripped
~~~

The decompiled binary tells us what's going on straight away.

It's simply checking the md5 sum of the input for a hard coded value.

~~~c++
  std::operator<<<std::char_traits<char>>(&std::cout, "Enter the valid key!\n", envp);
  ...
  md5(&v41, &v40);
  ...
  if ( *(_BYTE *)v42 != 55
    || *(_BYTE *)(v42 + 1) != 56
    || *(_BYTE *)(v42 + 2) != 48
...snip....
    || *(_BYTE *)(v42 + 29) != 53
    || *(_BYTE *)(v42 + 30) != 99
    || *(_BYTE *)(v42 + 31) != 48 )
  {
    // yay got the flag
~~~

The md5sum it's checking for is `780438d5b6e29db0898bc4f0225935c0`. Using [an online cracker](https://crackstation.net/) we quickly find the input to be `b781cbb29054db12f88f08c6e161c199` which is `md5("grape")`.

*Flag is b781cbb29054db12f88f08c6e161c199*
