+++
author = "arturo182"
categories = [ "writeups" ]
date = "2016-02-21T13:20:00+01:00"
title = "Internetwache CTF 2016 Procrastination (Crypto 80) Writeup"
+++

# Problem

>  Watching videos is fun!

Attachment: https://ctf.internetwache.org/files/crypto80.zip

Solved by 74 teams

# Solution

We unpack and get a webm file. When opened, it plays one of the [big hits from the 80s](https://www.youtube.com/watch?v=dQw4w9WgXcQ) for 36 seconds.
Let's run `mediainfo` on it, we see that it contains one video track and two audio tracks, let's extract the second audio track.
~~~~none
avconv -i song.webm -map 0:2 audio.wav
~~~~

When playing it, we can hear phone dial noises, also known as [DTMF](https://en.wikipedia.org/wiki/Dual-tone_multi-frequency_signaling). So we run DTMF recognition software.
~~~none
multimon -t wav -a DTMF audio.wav
~~~

And we get the DTMF tones as output:
~~~none
DTMF: 0
DTMF: 1
DTMF: 1
DTMF: 1
DTMF: 0
DTMF: 1
DTMF: 2
DTMF: 7
DTMF: 0
...snip...
DTMF: 0
DTMF: 1
DTMF: 2
DTMF: 3
~~~

We can observe that it seems to be 2-3 digits separated by `0`. Let's group them and remove the separating zeros:
~~~none
111 127 173 104 122 60 116 63 123 137 127 61 124 110 137 120 110 60 116 63 123
~~~

Oh no, it's those pesky octal numbers [again](/post/internetwache-2016-misc-50-arturo182)!
Let's do what we did last time (code reuse FTW!).

~~~python
import string

f = open('dtmf.txt')
line = f.readline().replace('\n', '')
print ''.join([chr(string.atoi(x, base=8)) for x in line.split(' ')])
~~~

(For some reason multimon didn't get the last 3 digits, `125 = base8(})`, but it wasn't hard to figure out)

Flag: IW{DR0N3S_W1TH_PH0N3S}