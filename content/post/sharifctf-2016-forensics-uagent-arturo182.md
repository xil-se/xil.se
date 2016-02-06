+++
authors = "arturo182"
categories = [ "writeups" ]
date = "2016-02-06T20:42:49+01:00"
title = "SharifCTF 2016 Uagent (Forensics 100) Writeup"
+++

# Problem

> We think we are really cool, are we?

Attachement: a pcap file

# Solution

The pcap file contains a session of someone downloading a file over HTTP. The title suggests checking the user-agent field, which I did, as it turns out, all the HTTP Request packages have a user agent in this format: `sctf-app/iVBORw0=/`, which is obviously base64.  
So I wrote a python script to extract the base64, decode it and append to a file:

~~~python
from scapy.all import *
from scapy.layers import http
import base64

pcap = rdpcap('ragent.pcap')
req = [p for p in pcap if p.haslayer(http.HTTPRequest)]

png = open('flag.png', 'wb')
saved = []
for p in req:
    if p['TCP'].seq in saved:
        continue
    saved.append(p['TCP'].seq)

    r = p.getlayer(http.HTTPRequest)
    png.write(base64.b64decode(r.fields['User-Agent'][9:-1]))

png.close()
~~~

Obviously I didn't know it was a PNG at the time of writing the script, but after it was done, `file` told me so.
When we open the file, it says `the passw0rd of archive file is: yAThV9PyXgbeKNpzy7RI`. So there's a archive now? And here I was thinking I was done. But enough complaining! 

So there was another file hidden in the pcap, the logical conclusion was that it was the file actually being downloaded in the HTTP session. So we write another python script:

~~~python
from scapy.all import *
from scapy.layers import http

pcap = rdpcap('ragent.pcap')
req = [p for p in pcap if p.haslayer(http.HTTPResponse)]
arch = open('flag.zip', 'wb')

for p in req:
    r = p.getlayer(http.HTTPResponse)

    idx = r.fields['Content-Range'].index('-')
    start = int(r.fields['Content-Range'][6:idx])

    arch.seek(start)
    arch.write(r.payload.load)

arch.close()
~~~

As you notice, we use the Content-Range header information to decide where the bytes will be written, this is because the packages were out of order and overlapping, this way it all worked just fine.

We run the script, extract the archive using the password and we get a png file containing the flag in a very fancy font!

Flag: SharifCTF{94f7df30fbd061cc4f7294369a8bce1c}
