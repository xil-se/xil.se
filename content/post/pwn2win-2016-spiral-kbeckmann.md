+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-03-28T00:30:27+02:00"
description = ""
title = "pwn2win 2016 Square Infinite Spiral Writeup"

+++

# Problem

Didn't save the original text but here are my words:

Imagine an infinite 2d grid. Start at (0, 0). Move one unit right, then turn left whenever possible so that you don't intersect with your trail. I.e. make a counter clock wise spiral. The challenge input is the number of iterations, the solution is the coordinate where the point ends up.

For input=4, the solution is (-1, 1)
```
----------
  ------ |
  |    | |
  | .--- |
  --------

```


# Solution

It took a while for me to get what the challenge really was about. Instead of wasting time on trying to solve the problem, I did what a proper script kiddie does well - I used Google. I found a very good [solution on stackoverflow](http://stackoverflow.com/a/19287714/1747702) that takes the number of iterations and returns the coordinates in O(1) time.

The challenge input is a huge number, sometimes over 1024 bits in length, so we need to support that.

I re-wrote the code in python, rotated the coordinates (since the spiral went clockwise in the code above) and adapted it to use `decimal` to get arbitrary precision. Then just hooked it up to the game server and got the flag.

```python
import math
import decimal
from pwn import *
context.log_level = 0

_ctx = decimal.getcontext()
_ctx.prec=2**12

def floor(x):
    with decimal.localcontext() as ctx:
        #ctx.prec=100000000000000000
        ctx.prec=1
        ctx.rounding=decimal.ROUND_FLOOR
        return x.to_integral_exact()

# http://stackoverflow.com/a/19287714/1747702
def spiral(n):
    #given n an index in the squared spiral
    #p the sum of point in inner square
    #a the position on the current square
    #n = p + a

    _n = decimal.Decimal(n)

    #r = math.floor((math.sqrt(n + 1) - 1) / 2) + 1
    r = long(floor((decimal.Decimal(_n + 1).sqrt() - 1) / 2) + 1)

    #compute radius : inverse arithmetic sum of 8+16+24+...=
    p = (8 * r * (r - 1)) / 2
    #compute total point on radius -1 : arithmetic sum of 8+16+24+...

    en = r * 2
    #points by face

    a = (1 + n - p) % (r * 8)
    #compute de position and shift it so the first is (-r,-r) but (-r+1,-r)
    #so square can connect

    pos = [0, 0, r];
    tmp = math.floor(a / (r * 2))
    #find the face : 0 top, 1 right, 2, bottom, 3 left
    if tmp == 0:
        pos[0] = a - r
        pos[1] = -r
    elif tmp == 1:
        pos[0] = r
        pos[1] = (a % en) - r
    elif tmp == 2:
        pos[0] = r - (a % en)
        pos[1] = r
    elif tmp == 3:
        pos[0] = -r
        pos[1] = r - (a % en)

    #print("n : ", n, " r : ", r, " p : ", p, " a : ", a, "  -->  ", pos)
    newpos = [-pos[1], pos[0], r]
    return newpos

#for i in xrange(25):
#    print i+1, spiral(i)

#print spiral(87617132336372079846506616471286455749443259655954533789357600981378524569118335678751536947646080176816437606601550323221987024289898039719680451033326652052129641143066615453270361535742146132971398209593358690841000867875335581644900086496061447470988285895603240925546341429765197561195418284611512072176183221118915681135559699902117689293065428013532910524117825539447351947261308698064828704511533056969797355257686729816425986743810441213016227815358114008621623753229561235048044)

p = process("openssl s_client -connect programming.pwn2win.party:9004", shell=True)

p.recvuntil("Verify return code: 0 (ok)\n---\n")

while True:
    N=p.recvuntil("\n")
    print "N=", N
    ret = spiral(long(N) - 1)
    p.sendline(str(ret[0]) + " " + str(ret[1]))

print p.recv()

#CTF-BR{iT-Was-JusT-BIgInteGeR-MFQnQRqKLcSHi}

```
