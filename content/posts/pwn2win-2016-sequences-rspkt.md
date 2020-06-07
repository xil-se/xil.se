+++
author = "rspkt"
categories = [ "writeups" ]
date = "2016-03-28T00:44:00+01:00"
title = "Pwn2Win CTF 2016 Sequences (PPC 40) Writeup"

+++

# Problem

> Description: Warm up for the next PPC challenges. The Club guys love
sequences, and it is always good to know your enemy.

We will show you some sequences, and after each sequence you need to predict
the values which correctly fill the asked positions. We ALWAYS adopt the
convention that position 1 corresponds to the first number of the sequence.

Output sent by the server: Position - Sequence

Result format expected by the server: result1,result2,result3,result4...

# Solution

This challenge was pretty straight forward. Detect what sequences that were
used, apply them to find the nth number of the sequence, get flag.

The server sent the first three numbers from each sequence. First thing we
wanted to figure out if it was an arithmetic sequence (add a value) or
geometric sequence (multiply with value).

If it's an arithmetic sequence, the third number should be obtained by
calculating the difference between the first and second number, and add that to
the second number. If it's a geometric sequence, the third number should be
obtained by calculating the ratio between the first and second number, and
multiply that to the second number.

This worked fine for most sequences, but from time to time we got floating
point numbers as ratios. The two numbers that were always recurring was phi
(the golden ratio), and the Tribonacci constant. These ratios occur between
numbers in the Fibonacci sequence and the Tribonacci sequence. We delt with
this by simply implementing the generalized versions of the Fibonacci and
Tribonacci sequences, and applying those whenever the constants popped up in
the ratio calculations.


Code
~~~python
from __future__ import division
import re

from pwn import *

# Phi and the Tribonacci constant
PHI = 1.618033988749
TRIB = 1.839286755214


def gen_fib_n(a, b, n):
    if n < 2:
        return n
    for num in xrange(2, n):
        a, b = b, b + a
    return b


def gen_trib_n(a, b, c, n):
    if n < 3:
        return n
    for num in xrange(2, n):
        a, b, c = b, c, c + b + a
    return b


def calc_nth(a, b, c, n):
    # Calculate the sequence params
    params = {
        'add': b - a,
        'mul': b / a,
    }

    # IF fibonacci/tribonacci, treat specially
    if abs(params['mul'] - PHI) < 0.1:
        return gen_fib_n(a, b, n)
    if abs(params['mul'] - TRIB) < 0.1:
        return gen_trib_n(a, b, c, n)

    # Sequence with params, calculate the diff
    diffs = {
        'add': c - (b + params['add']),
        'mul': c - (b * params['mul']),
    }

    # Find the type with the smallest diff
    t = min(diffs.iteritems(), key=lambda (x, y): abs(y))[0]

    # Now we know the type. Let's calculate the nth term.
    if t == 'add':
        return int(a + (n - 1) * params['add'])
    elif t == 'mul':
        return int((a * (params['mul']**(n - 1))).round())


def doit():
    p = remote('pool.pwn2win.party', 1337)

    res = []
    for i in range(15):
        line = p.readline().rstrip()
        (n, a, b, c) = re.match('(\d+) - (\d+) -> (\d+) -> (\d+)', line).groups()
        nth = calc_nth(int(a), int(b), int(c), int(n))
        res.append(nth)

    answer = ','.join(map(str, res))
    p.sendline(answer)

    print p.readall()


if __name__ == '__main__':
    doit()

~~~
