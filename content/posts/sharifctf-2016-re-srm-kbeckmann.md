+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-06T21:41:14+01:00"
title = "SharifCTF 2016 SRM (RE 50) Writeup"
+++

# Problem

> The flag is : The valid serial number

Points: 50

Solved by 176 team(s)

# Solution

We are provided with a PE32 windows executable:
~~~
RM.exe: PE32 executable (GUI) Intel 80386, for MS Windows
~~~

The binary has a lot of code, so I started to look for how it interacts with the user.

sub_401280() calls message box APIs so it looks interesting. It turns out that this is where all the magic happens.

It reads two inputs, a valid e-mail address and a serial number. I was over thinking this and thought I had to write an actual keygen that had to generate a key that only works with a specific e-mail address, but this was not the case.

The key was hard-coded in the check.

~~~c++
if ( strlen(v12) != 16
      || v12[0] != 67
      || v24 != 88
      || v12[1] != 90
      || v12[1] + v23 != 155
      || v12[2] != 57
      || v12[2] + v22 != 155
      || v12[3] != 100
      || v21 != 55
      || v13 != 109
      || v20 != 71
      || v14 != 113
      || v14 + v19 != 170
      || v15 != 52
      || v18 != 103
      || v16 != 99
      || v17 != 56 )
~~~

Sorting out the order of the input (v12[0]..[3], v13, ... v24) made it easy to find the flag.

Values are `67 90 57 100 109 113 52 99 56 103 57 71 55 98 65 88`, or `CZ9dmq4c8g9G7bAX`.

Running the binary through Wine verified that the serial was correct.

*Flag is CZ9dmq4c8g9G7bAX*
