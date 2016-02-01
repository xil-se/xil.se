+++
date        = "2016-01-31"
title       = "HackIM 2016 Misc 2 Writeup"
description = "Hiding In Plain Sight - 200pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim" ]
authors     = "arturo182"
+++

# Problem
> Find out the secret key hidden in these packets!

Attachement: http://ctf.nullcon.net/misc/m200-HiPs.rar

Files:

* f101.pcap
* f102.pcap
* f103.pcap

# Solution

We get three **p**acket **cap**ture files, when inspecting with Wireshark, we can see they log ping requests and responses between two hosts. Upon closer inspection we see that they seem to have quite large data payloads attached, so we decided to extract and concatenate all the data from the first file.  
It was not immediately clear what the data was, but upon further inspection and some research, I decided to try using yEnc:

~~~python
from scapy.all import *
from yenc import decode

pcap = rdpcap('f101.pcap')
f = open('f101.yenc','w')

for p in pcap:
    if p[ICMP].type == 8:
        f.write(''.join(str(p[Raw])))

f.close()

decode('f101.yenc', 'f101.bin')
~~~

This gave us a binary file with an interesting content inside:
~~~
...snip...
00007a0: 0a3f 7b29 2b7b 7d5b 5d2b 3d3c 2b2f 285d 402b 5b5d 293f 265d 282e 3c7d 7b2a 255e  .?{)+{}[]+=<+/(]@+[])?&](.<}{*%^
00007c0: 3f7e 283f 7b26 2d3d 2e2e 405d 2d28 3d2d 5e2d 292d 2b7d 2a7d 3c7e 5d5e 3c28 5b5b  ?~(?{&-=..@]-(=-^-)-+}*}<~]^<([[
00007e0: 2d0a 293a 7b2d 407d 2a5d 7c2a 2f7b 5b5b 7b7e 5b2f 2c28 7d7d 2b2f 407c 7b3f 3e2f  -.):{-@}*]|*/{[[{~[/,(}}+/@|{?>/
0000800: 3e25 267b 5b2b 3c2a 2c3d 285b 2d5e 407b 2c2f 283d 5d3c 2f3a 265e 2c3a 2629 3d2d  >%&{[+<*,=([-^@{,/(=]</:&^,:&)=-
0000820: 2926 0a5a 574e 6f62 7941 695a 6949 4b5a 574e 6f62 7941 6962 4349 4b5a 574e 6f62  )&.ZWNobyAiZiIKZWNobyAibCIKZWNob
0000840: 7941 6959 5349 4b5a 574e 6f62 7941 695a 7949 4b5a 574e 6f62 7941 6965 7949 4b5a  yAiYSIKZWNobyAiZyIKZWNobyAieyIKZ
0000860: 574e 6f0a 6279 4169 6257 5131 6333 5674 4967 706c 5932 6876 4943 496f 4a32 5a73  WNo.byAibWQ1c3VtIgplY2hvICIoJ2Zs
0000880: 5957 636e 4b53 494b 5a57 4e6f 6279 4169 6653 494b 3a7b 2b5d 3d5e 7e7e 2e40 2d5e  YWcnKSIKZWNobyAifSIK:{+]=^~~.@-^
00008a0: 3c2b 2e3e 0a3c 2f28 253e 2b28 292f 2a2e 7d28 3e25 3c2e 7b2a 253a 3e40 2c2d 7b2b  <+.>.</(%>+()/*.}(>%<.{*%:>@,-{+
00008c0: 2c5e 3f2c 2f40 2f7d 3a2b 293d 3d2e 257b 2d7e 253d 3e29 3a7d 2b5d 2a29 5d3d 7c40  ,^?,/@/}:+)==.%{-~%=>):}+]*)]=|@
00008e0: 2a3a 263e 3c0a 3f7b 2c5d 2e2d 2e7d 5e29 7e26 3d7e 255d 7e3d 3c3d 3f2f 2f7b 4028  *:&><.?{,].-.}^)~&=~%]~=<=?//{@(
...snip...
~~~

If we remove the newline from the ASCII text we get: `ZWNobyAiZiIKZWNobyAibCIKZWNobyAiYSIKZWNobyAiZyIKZWNobyAieyIKZWNobyAibWQ1c3VtIgplY2hvICIoJ2ZsYWcnKSIKZWNobyAifSIK` which is base64 for: 
~~~
echo "f"
echo "l"
echo "a"
echo "g"
echo "{"
echo "md5sum"
echo "('flag')"
echo "}"
~~~

Flag: flag{327a6c4304ad5938eaf0efb6cc3e53dc}
