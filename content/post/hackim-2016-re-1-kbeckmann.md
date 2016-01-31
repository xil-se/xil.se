+++
date        = "2016-01-31"
title       = "HackIM 2016 RE 1 'ZorroPub' Writeup"
description = "ZorroPub . 100pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim", "reversing" ]
+++

# Problem

We are provided with the following binary:

```
$ file zorro_bin
zorro_bin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=5bd9436f341615c804471bb5aec37e426508a7af, stripped

$ ldd zorro_bin
    ./zorro_bin: /usr/lib/libcrypto.so.1.0.0: no version information available (required by ./zorro_bin)
    linux-vdso.so.1 (0x00007ffc84da4000)
    libcrypto.so.1.0.0 => /usr/lib/libcrypto.so.1.0.0 (0x00007f3b1f2a9000)
    libc.so.6 => /usr/lib/libc.so.6 (0x00007f3b1ef05000)
    libdl.so.2 => /usr/lib/libdl.so.2 (0x00007f3b1ed01000)
    libz.so.1 => /usr/lib/libz.so.1 (0x00007f3b1eaeb000)
/lib64/ld-linux-x86-64.so.2 (0x00007f3b1f721000)
```

Running it:
```
$ ./zorro_bin
./zorro_bin: /usr/lib/libcrypto.so.1.0.0: no version information available (required by ./zorro_bin)
Welcome to Pub Zorro!!
Straight to the point. How many drinks you want?3
OK. I need details of all the drinks. Give me 3 drink ids:1 2 3
Invalid Drink Id.
Get Out!!
```

Relevant decompiled code:
```
seed = 0;
puts("Welcome to Pub Zorro!!");
printf("Straight to the point. How many drinks you want?", a2);
__isoc99_scanf("%d", &v6);
if ( (signed int)v6 <= 0 )
{
    printf("You are too drunk!! Get Out!!");
    exit(-1);
}
printf("OK. I need details of all the drinks. Give me %d drink ids:", v6);
for ( i = 0; (signed int)i < (signed int)v6; ++i )
{
    __isoc99_scanf("%d", &v7);
    if ( v7 <= 16 || v7 > 0xFFFF )
    {
        puts("Invalid Drink Id.");
        printf("Get Out!!");
        exit(-1);
    }
    seed ^= v7;
}
i = seed;
v10 = 0;
while ( i )
{
    ++v10;
    i &= i - 1;
}
if ( v10 != 10 )
{
    puts("Looks like its a dangerous combination of drinks right there.");
    puts("Get Out, you will get yourself killed");
    exit(-1);
}
srand(seed);
MD5_Init(&v11);
for ( i = 0; (signed int)i <= 29; ++i )
{
    v10 = rand() % 1000;
    sprintf(&s, "%d", (unsigned int)v10);
    v3 = strlen(&s);
    MD5_Update(&v11, &s, v3);
    v13[i] = v10 ^ dword_6020C0[4 * i];
}
v13[i] = 0;
MD5_Final(v12, &v11);
for ( i = 0; (signed int)i <= 15; ++i )
sprintf(&s1[2 * i], "%02x", (unsigned __int8)v12[i]);
if ( strcmp(s1, "5eba99aff105c9ff6a1a913e343fec67") )
{
    puts("Try different mix, This mix is too sloppy");
    exit(-1);
}
result = printf("\nYou choose right mix and here is your reward: The flag is nullcon{%s}\n", v13);
v5 = *MK_FP(__FS__, 40LL) ^ v16;
return result;
```

# Solution
Based on the decompiled code, we should enter "10" drinks and then enter 10 values between 0x10 and 0xffff.

The binary generates a seed based on the input that is used to seed rand() which is later used. Instead of solving the mystery input, I bruteforced the seed value to solve this.


```
int crack(int seed)
{
    MD5_CTX v11;
    unsigned char v12[16];
    unsigned char v13[32];
    unsigned char s1[40];

    srand(seed);
    MD5_Init(&v11);

    int i, v10, v3;
    char s;
    for (i = 0; i <= 29; ++i )
    {
        v10 = rand() % 1000;
        sprintf(&s, "%d", (unsigned int)v10);
        v3 = strlen(&s);
        MD5_Update(&v11, &s, v3);
        v13[i] = v10 ^ dword_6020C0[4 * i];
    }
    v13[i] = 0;
    MD5_Final(v12, &v11);
    for ( i = 0; (signed int)i <= 15; ++i )
        sprintf(&s1[2 * i], "%02x", (unsigned char)v12[i]);
    if (strncmp(s1, correct_hash, 32) == 0) {
        printf("SUCCESS! seed=[%d] [%s] [%s]\n", seed, v13, s1);
        return 0;
    }
    return 1;
}

int main()
{
    int i = 0;
    while (crack(i++));
    return 0;
}
```

Running the cracker, we see the following output:

```
$ gcc crack.c -lcrypto -o crack && ./crack
SUCCESS! seed=[59306] [nu11c0n_s4yz_x0r1n6_1s_4m4z1ng] [5eba99aff105c9ff6a1a913e343fec67]
```


**The flag: nullcon{nu11c0n_s4yz_x0r1n6_1s_4m4z1ng}**
