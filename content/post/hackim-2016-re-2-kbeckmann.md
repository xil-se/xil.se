+++
date        = "2016-01-31"
title       = "HackIM 2016 RE 2 'Pseudorandom' Writeup"
description = "Random as F*!@ . 300pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim", "reversing" ]
+++

# Problem

We are provided with the following binary:
```
$ file pseudorandom_bin
pseudorandom_bin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=5a0f467ef94ee8fa770ecda91c4326f00b2c6c30, stripped

$ ldd pseudorandom_bin
./pseudorandom_bin: /usr/lib/libcrypto.so.1.0.0: no version information available (required by ./pseudorandom_bin)
	linux-vdso.so.1 (0x00007ffe25b0f000)
	libcrypto.so.1.0.0 => /usr/lib/libcrypto.so.1.0.0 (0x00007fe7ef580000)
	libc.so.6 => /usr/lib/libc.so.6 (0x00007fe7ef1dc000)
	libdl.so.2 => /usr/lib/libdl.so.2 (0x00007fe7eefd8000)
	libz.so.1 => /usr/lib/libz.so.1 (0x00007fe7eedc2000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fe7ef9f8000)

```

Running it:
```
$ ./pseudorandom_bin
./pseudorandom_bin: /usr/lib/libcrypto.so.1.0.0: no version information available (required by ./pseudorandom_bin)
I will generate some random numbers.
If you can give me those numbers, you will be $$rewarded$$
hmm..thinking...
```
Stalled. Looking at the decompiled binary, we see a couple of
```
sleep(rand());
```
Binary patch them to NOPs so we can run the binary.

Entering some random value:
```
2
Nope. That is not what I am expecting!!
```

So we have to feed it with something. Let's analyze the binary further.

The interesting stuff decompiled:
```
v24 = 0;
v23 = a1;
v22 = a2;
printf("I will generate some random numbers.\n", a2);
printf("If you can give me those numbers, you will be $$rewarded$$\n");
printf("hmm..thinking...");
fflush(stdout);
v3 = rand();
sleep(v3);
v21 = rand();
v20 = 0;
printf("OK. I am Ready. Enter Numbers.\n");
v17 = sub_400C30((unsigned int)v21);
v18 = (1 << sub_400B40((unsigned int)v21)) - 1;
MD5_Init(&v15);
SHA1_Init(&v14);
while ( v20 != v21 )
{
  __isoc99_scanf("%d", &v19);
  v4 = v18;
  if ( !sub_400EA0(v18, v19) )
    sub_400AE0(v4);
  v18 = v19 + v18 - 409176519 + 409176519;
  sprintf(&s, "%d", v19);
  v5 = strlen(&s);
  MD5_Update(&v15, &s, v5);
  v6 = strlen(&s);
  SHA1_Update(&v14, &s, v6);
  while ( ~(~v17 | ~v18) )
  {
    v20 = v17 ^ v20 | v17 & v20;
    v18 &= v17 ^ v18;
    v17 = sub_400C30(v21 & (v20 ^ (unsigned int)v21));
  }
}
MD5_Final(v12, &v15);
for ( i = 0; i < 16; i = i - 648881180 + 648881181 )
  sprintf(&s1[2 * i], "%02x", (unsigned __int8)v12[i]);
if ( strcmp(s1, (const char *)(unsigned int)"15b74b4db57d0afdfe98eb5dbc3b542b") )
  sub_400AE0(s1);
printf("Good Job!!\n");
printf("Wait till I fetch your reward...");
fflush(stdout);
v7 = rand();
sleep(v7);
SHA1_Final(v11, &v14);
for ( i = 0; i < 20; ++i )
  sprintf(&v9[2 * i], "%02x", (unsigned __int8)v11[i]);
printf("OK. Here it is\n");
for ( i = 0; i < 40; i = i - 1320014540 + 1320014541 )
  v9[i] = (dword_6020D0[4 * i] & 0x4A | ~dword_6020D0[4 * i] & 0xB5) ^ (v9[i] & 0x4A | ~v9[i] & 0xB5);
printf("The flag is:nullcon{%s}\n", v9);
return v24;
```

# Solution
Being lazy and wanting to get results fast, I simply patched the code to brute force itself. Sorry for the different variable names here, they changed when decompiling the patched binary.


```
int guess_start = 0;
int oldv19 = 0;
while ( v19 != v20 )
{
  if (oldv19 != v19) {
    // When v19 changes we need to reset the guess
    oldv19 = v19;
    guess_start = 0;
  }

  printf("v19=%d, v20=%d\n", v19, v20);
  //scanf("%d", &v18);
  int x = guess_start;
  printf ("cracking [%d]=>", v17);
  int ret = -1;
  while (ret == -1 || ret == 0) {
    ret = sub_400EA0(v17, x++);
  }
  guess_start = x*2 - 2;

  printf ("[%d]\n", x - 1);
  fprintf(stderr, "%d\n", x-1);

  v18 = x - 1;
  if ( !(unsigned int)sub_400EA0((unsigned int)v17, v18) )
  ...
```

This prints the correct numbers to stderr that we then can feed to the actual binary. Had some problem with the hashing in my patched program, but doesn't matter, got the flag!


```
$ ./crack 2> numbers
$ ./pseudorandom_bin < numbers
./pseudorandom_bin: /usr/lib/libcrypto.so.1.0.0: no version information available (required by ./pseudorandom_bin)
I will generate some random numbers.
If you can give me those numbers, you will be $$rewarded$$
hmm..thinking...OK. I am Ready. Enter Numbers.
Good Job!!
Wait till I fetch your reward...OK. Here it is
The flag is:nullcon{50_5tup1d_ch4113ng3_f0r_e1i73er_71k3-y0u}
```


**The flag: nullcon{50_5tup1d_ch4113ng3_f0r_e1i73er_71k3-y0u}** 


### Full input to the binary

```
$ cat numbers
32768
65536
131072
262144
524288
1048576
2097152
4194304
8388608
16777216
33554432
67108864
134217728
268435456
16384
32768
65536
131072
262144
524288
1048576
2097152
4194304
8388608
16777216
33554432
67108864
134217728
8192
16384
32768
65536
131072
262144
524288
1048576
2097152
4194304
8388608
16777216
4096
8192
16384
32768
65536
131072
262144
524288
1048576
2097152
2048
4096
8192
16384
32768
65536
131072
262144
524288
1048576
1024
2048
4096
8192
16384
32768
65536
131072
262144
524288
512
1024
2048
4096
8192
16384
32768
65536
131072
262144
256
512
1024
2048
4096
8192
16384
32768
65536
128
256
512
1024
2048
4096
8192
16384
32768
64
128
256
512
1024
2048
4096
32
64
128
256
16
32
64
128
8
16
32
64
4
8
16
32
2
1
```
