+++
author = "arturo182"
categories = [ "writeups" ]
date = "2016-02-21T13:15:00+01:00"
title = "Internetwache CTF 2016 Quick Run (Misc 60) Writeup"
+++

# Problem

>  Someone sent me a file with white and black rectangles. I don't know how to read it. Can you help me?

Attachment: https://ctf.internetwache.org/files/misc60.zip

Solved by 269 teams

# Solution

We get a text file with a lot of base64, we dump it to a file:
~~~none
cat README.txt | base64 -d > out
~~~

and see what's inside:

[![QR Codes!](/imgs/internetwache-2016-misc-60-arturo182_qrcode.png)](/imgs/internetwache-2016-misc-60-arturo182_qrcode.png)

There were 24 ASCII QR Codes in the file, no python magic here, just a quick scan with a mobile app and we get the message letter-by-letter: `Flagis:IW{QR_C0DES_RUL3}`.

Flag: IW{QR_C0DES_RUL3}