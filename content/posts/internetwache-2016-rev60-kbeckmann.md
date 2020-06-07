+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-21T01:10:57+01:00"
title = "InternetWache 2016 File Checker (Reverse 60) Writeup"

+++

# Problem

> Description: My friend sent me this file. He told that if I manage to reverse it, I'll have access to all his devices. My misfortune that I don't know anything about reversing :/

(rev60, solved by 220)

Attachment: rev60.zip

# Solution

Looking at the provided binary we quickly see that it is reading a file `.password` and compares its contents to a set of values.

~~~c++
v4 = 4846;
v5 = 4832;
...
v2 = (*(&v4 + a1) + *a2) % 4919;
result = (unsigned int)v2;
~~~

With some quick python-fu i got the flag:

~~~python
print ''.join([chr(4919 - c) for c in [4846,4832,4796,4849,4846,4843,4850,4824,4852,4847,4818,4852,4844,4822,4794]])
~~~

Flag is: **IW{FILE_CHeCKa}**.
