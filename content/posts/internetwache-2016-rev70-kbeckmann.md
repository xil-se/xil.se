+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-21T01:18:38+01:00"
title = "InternetWache 2016 ServerfARM (Reverse 70) Writeup"

+++

# Problem

> Description: Someone handed me this and told me that to pass the exam, I have to extract a secret string. I know cheating is bad, but once does not count. So are you willing to help me?

(rev70, solved by 184)

Attachment: rev70.zip

# Solution

The zip contains an ARM binary that seems to be compiled in a pretty nice way, no -O3 here.

I reversed the binary and could rebuild the flag from the execution paths. There were enough hard coded values to find the flag quite quickly.

~~~c++
printf("%s", "IW{");
putchar(83);
printf("%c%c\n", 46, 69);
printf("%s", ".R.");
putchar(86);
printf("%c%c\n", 46, 69);
puts(".R>=F:");
printf("%c%s%c\n", 65, ":R:M", 125);
~~~

The correct input to the binary is `\x01\nAB\n1337\n`.

Flag is **IW{S.E.R.V.E.R>=F:A:R:M}**
