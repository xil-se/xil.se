+++
date        = "2016-01-31"
title       = "HackIM 2016 Trivia 3 Writeup"
description = "Trivia 3 - 300pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim" ]
+++

# Problem
> Bill Gates loves Cipher.

Attachement: http://ctf.nullcon.net/trivia/trivia3.png

# Solution

This one took some time to figure out, mostly because the description is so vague. But using reverse image search for the cipher characters and "Bill" as keyword (Gates was a red herring), we managed to find [Bill's Cipher](http://gravityfalls.wikia.com/wiki/List_of_cryptograms/Books#Bill.27s_symbol_substitution_cipher).

With the substitution table, it was very easy to get the proper string.

Flag: aglisurakshakivastu
