+++
date        = "2016-01-31"
title       = "HackIM 2016 Trivia 4 Writeup"
description = "Trivia 4 - 400pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim" ]
authors     = "arturo182"
+++

# Problem
> Use the file. Get the flag. But, you know what, I hate pipes.

Attachement: http://ctf.nullcon.net/trivia/trivia4.txt

# Solution

The file contains the esoteric Ook language, we can convert it to text using [this tool](http://www.splitbrain.org/services/ook).  
We get:

> starting from 0.0.0.0, print the following IPs.  
> 7277067th IP Address  
> 7234562th IP Address  
> 7302657th IP Address  
> 91238th IP Address  
> 746508th IP Address  
> 7211531th IP Address  
> 7300098th IP Address  
> 7211788th IP Address  
> 723558th IP Address  
> 91248th IP Address  
> 7237378th IP Address  
> 723557th IP Address  
> 7234562th IP Address  
> 723567th IP Address  
> 749067th IP Address  
> Hint: Anything specific about all the IPs?

We can use python to convert those numbers to dot-decimal notation with some python-fu:
~~~python
import socket, struct, binascii

ips = [7277067, 7234562, 7302657, 91238, 746508, 7211531, 7300098, 7211788, 723558, 91248, 7237378, 723557, 7234562, 723567, 749067]

result = ''
for ip in ips:
    # -1 because it's the nth ip and we count base 0
    as_str = socket.inet_ntoa(struct.pack('!I', ip - 1))
    # print as_str  # uncomment to see the raw ips
    as_str = as_str.replace('.', '')
    as_int = int(as_str, 2)
    result = result + binascii.unhexlify('%x' % as_int)

# they hate pipes
result = result.replace('|', '')

print result
~~~

The result is the flag.

Flag: ziesjyougotivz
