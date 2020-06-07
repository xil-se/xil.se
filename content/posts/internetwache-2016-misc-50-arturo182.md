+++
authors = "arturo182"
categories = [ "writeups" ]
date = "2016-02-21T12:56:49+01:00"
title = "Internetwache CTF 2016 The Hidden Message (Misc 50) Writeup"
+++

# Problem

>  My friend really can't remember passwords. So he uses some kind of obfuscation. Can you restore the plaintext?

Attachment: https://ctf.internetwache.org/files/misc50.zip

Solved by 347 teams

# Solution

We get a text file with the following contents:

~~~
0000000 126 062 126 163 142 103 102 153 142 062 065 154 111 121 157 113
0000020 122 155 170 150 132 172 157 147 123 126 144 067 124 152 102 146
0000040 115 107 065 154 130 062 116 150 142 154 071 172 144 104 102 167
0000060 130 063 153 167 144 130 060 113 012
0000071
~~~

Looks like a hex dump but with decimals instead of hex numbers, right? Not really, numbers like 172 are too big for ascii codes. How about octal? Let's try.

~~~python
import string

res = ''
f = open('README.txt')
for line in f:
    res = res + ''.join([chr(string.atoi(x, base=8)) for x in line.split(' ')[1::]])

print res,
~~~

The output is `V2VsbCBkb25lIQoKRmxhZzogSVd7TjBfMG5lX2Nhbl9zdDBwX3kwdX0K`, we're good.

We unbase64 it and get

> Well done!
>
> Flag: IW{N0_0ne_can_st0p_y0u}

Flag: IW{N0_0ne_can_st0p_y0u}