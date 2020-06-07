+++
author = "simonvik"
categories = [ "writeups" ]
date = "2016-02-21T20:00:12+01:00"
description = ""
title = "InternetWache 2016 404 Flag not found (misc 80) "

+++

# Problem

> Description: I tried to download the flag, but somehow received only 404 errors
:( Hint: The last step is to look for flag pattern.

(misc80, solved by 292)

Attachment: misc80.zip

# Solution

We are provided with a pcapng file, after opening the file in wireshark we can
see that it contains some HTTP Requests (without answers) and some DNS lookups.

After looking at it for a while we noticed that the hostnames looked
interesting, the first part of the hostname looked like hex-encoded ascii.


I wrote a small python-script that decodes the pcap and the DNS queries:

(I could not get python to read the pcapng so i converted it to pcap using [pcapng](http://pcapng.com/))

~~~PYTHON
#!/usr/bin/env python

import base64
import re

from scapy.all import *
from scapy.layers.dns import DNSRR, DNS, DNSQR

pcap = './flag.s0i0.pcap'
pkts = rdpcap(pcap)

lst = []

for p in pkts:

    if p.haslayer(DNS):

	if p.qdcount > 0 and isinstance(p.qd, DNSQR):
            q = p.qd.qname
	    m = re.search('([^.]+)', q);
	    h = m.group(1)
	    s =  ''.join([chr(int(''.join(c), 16)) for c in zip(h[0::2],h[1::2])])
	    if s not in lst:
                lst.append(s)


print lst
~~~

The script generated the  following output:

~~~
["In the end, it's all about fla", 'gs.\nWhether you win or lose do', "esn't matter.\n{Ofc, winning is", ' cooler\nDid you find other fla', 'gs?\nNoboby finds other flags!\n', 'Superman is my hero.\n_HERO!!!_', "\nHelp me my friend, I'm lost i", 'n my own mind.\nAlways, always,', ' for ever alone.\nCrying until ', "I'm dying.\nKings never die.\nSo", ' do I.\n}!\n']

Pretty printed:

In the end, it's all about flags.
Whether you win or lose doesn't matter.
{Ofc, winning is cooler
Did you find other flags?
Noboby finds other flags!
Superman is my hero.
_HERO!!!_
Help me my friend, I'm lost in my own mind.
Always, always, for ever alone.
Crying until I'm dying.
Kings never die.
So do I.
}!

~~~

We know the flag format and could see that the first letter after \n matches the format

We now have the flag : **IW{DNS_HACKS}**
