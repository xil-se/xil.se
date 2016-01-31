+++
date        = "2016-01-31"
title       = "HackIM 2016 Crypto 3 Writeup"
description = "Crypto 3 - 400pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim" ]
+++

# Problem
> After entring the luxurious condomium,you get the feel that you are in home of a yester Star. the extravagant flooring and furnishings shows the richness of this star. But where is she? There she is, lying peacefuly on her couch. See what Envy has done to her...with a perfectly well maintained attractive body she still looks sex diva, except for her face beyond recogniton. Her identity is crucial to know who killed her and why? In absence of any personal data around there is only a file. with a cryptic text in it. Preity sure she has used her own name to XOR encrypt the file. And challenge is to know her name..

Attachement:
http://ctf.nullcon.net/crypto/AncientSecretsOfTheKamaSutra.txt

# Solution

Even though the challenge was to guess the key based on the hints, we hackers like to crack things, so I used a very neat tool called [xor-analyze](https://github.com/ThomasHabets/xor-analyze), which analysed the file and gave me its best guess at the key, which was correct.

```
./xor-analyze -v ~/ctf/AncientSecretsOfTheKamaSutra.txt freq/linux-2.2.14-int-m0.freq
```

Flag: Jeanna Fine

