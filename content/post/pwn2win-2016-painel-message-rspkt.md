+++
authors = "rspkt"
categories = [ "writeups" ]
date = "2016-03-28T01:14:00+01:00"
title = "Pwn2Win CTF 2016 Painel Message (Forensics 50) Writeup"

+++

# Problem

> Description: The last month we gained access to a video portraying a Club's
electronics project. It looks like they want to insert this display into the
the digital panels found in urban buses that circulate in large cities. We
need to discover what this is all about.

Those who automate the ENTIRE resolution of this challenge deserve 20 bonus
points. Show your code to a judge (@). ;)

# Solution

We were given a video of a two-row LED display with the leds being turned on
and off in 29 different configurations. We started off by transcribing these
into a binary form, ending up with the following:

~~~
011111110100011 1100100011111110
010000010101110 0010001010000010
010111010100101 0100101010111010
010111010010011 1111011010111010
010111010011001 0011111010111010
010000010110111 1001000010000010
011111110101010 1010101011111110
000000000100111 1101111000000000
000111010100100 0000011111001110
001000001010001 0100111111100000
001011010100100 0010001101100000
000101001010111 1100100110110010
001011010000001 1011100001101100
000011101101110 0110011011110110
010010010011000 0011001010000100
010100101111000 0111101110110110
011111010011110 0001100000000100
010110100000111 1001010111100100
010111110100100 0011101111111000
010001001101000 0111100011010010
010010011000110 1001101111101000
000000000111101 0111101000110010
011111110010010 0011011010101000
010000010000011 1111111000100010
010111010110011 0100101111101010
010111010101001 1010110000111000
010111010110011 1110011101111100
010000010001101 0000101001110100
011111110011000 0100110101001000
~~~

A lot of time was spend trying to perform various XOR approaches in order to
find some ASCII values in the mess of binary digits. The fact that the upper
row was only 15 bits made the whole thing seem a bit off. It wasn't until
Konrad (who had spent the last couple of hours solving the QRGrams challenge)
noticed the two QR-like squares in the upper corners that things got clear.
Translating the binary digits to an image revealed the QR-code, which gave us
the flag, **CTF-BR{Mentor comanda!}**.

[![Challenge video](/imgs/pwn2win-2016-painel-message-2.png)](/imgs/pwn2win-2016-painel-message-2.png)
[![QR code](/imgs/pwn2win-2016-painel-message-1.png)](/imgs/pwn2win-2016-painel-message-1.png)
