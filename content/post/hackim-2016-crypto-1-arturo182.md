+++
date        = "2016-01-31"
title       = "HackIM 2016 Crypto 1 Writeup"
description = "Crypto 1 - 500pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim" ]
authors     = "arturo182"
+++

# Problem
> You are in this GAME. A critical mission, and you are surrounded by the beauties, ready to shed their slik gowns on your beck. On onside your feelings are pulling you apart and another side you are called by the duty. The biggiest question is seX OR success? The signals of subconcious mind are not clear, cryptic. You also have the message of heart which is clear and cryptic. You just need to use three of them and find whats the clear message of your Mind... What you must do?

Attachement:
http://ctf.nullcon.net/crypto/crypto1.zip

Contents:

* Heart_clear.txt
* Heart_crypt.txt
* Mind_crypt.txt


# Solution

The description is super subtle that XOR should be used, so first to get the key we do `Heart_clear.txt XOR Heart_crypt.txt`, which gets us a key, then we do `key XOR Mind_crypt.txt`.


~~~python
def xor_strings(a, b):
    return ''.join(chr(ord(i) ^ ord(j)) for i, j in zip(a, b))

heart_clear = open('Heart_clear.txt').read()
heart_crypt = open('Heart_crypt.txt').read()
key = xor_strings(heart_clear, heart_crypt)

mind_crypt = open('Mind_crypt.txt').read()
mind_clear = xor_strings(mind_crypt, key)
print mind_clear
~~~

Which gives us a link:   https://play.google.com/store/apps/collection/promotion_3001629_watch_live_games?hl=en

At first I thought that was the flag (because HackIm has no flag format, tsk, tsk), but it was not accepted, if you follow the link, the header is:

> Never Miss a Game

Which is the flag.
