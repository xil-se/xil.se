+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-20T23:11:07+01:00"
title = "InternetWache 2016 Oh Bob! (Crypto 60) Writeup"

+++

# Problem

> Description: Alice wants to send Bob a confidential message. They both remember the crypto lecture about RSA. So Bob uses openssl to create key pairs. Finally, Alice encrypts the message with Bob's public keys and sends it to Bob. Clever Eve was able to intercept it. Can you help Eve to decrypt the message?

(crypto60, solved by 167)

Attachment: crypto60.zip

# Solution

The challenge zip file contains 3 public keys and a file with three encrypted strings.

The public keys can be investigated using openssl:

~~~
openssl rsa -in  task/bob3.pub -pubin -text -modulus
Public-Key: (228 bit)
Modulus:
    0c:5b:69:e1:97:9e:54:1f:85:da:cd:e2:aa:14:d2:
    72:2a:84:6f:41:b3:db:83:e6:67:e3:b3:d1:1d
Exponent: 65537 (0x10001)
Modulus=C5B69E1979E541F85DACDE2AA14D2722A846F41B3DB83E667E3B3D11D
~~~

Note that the key size is 228 bits, which is short. We can crack them.

Using the tool [yafu](https://sourceforge.net/projects/yafu/) we can factor the three keys very quickly on a pretty standard gaming machine. Took me only a couple of seconds to compute.

~~~
$ yafu
factor(0xC5B69E1979E541F85DACDE2AA14D2722A846F41B3DB83E667E3B3D11D)
p = 19193025210159847056853811703017693
q = 17357677172158834256725194757225793
~~~

Now we need to use the factors p and q to decrypt the message. Fortunately I was lazy and used google to find a very nice [writeup](https://github.com/p4-team/ctf/tree/master/2015-09-26-trendmicro/rsa) that helped me a lot.

The following code decrypts the messages:

~~~python
from Crypto.Util.number import *
import base64

def egcd(a, b):
    u, u1 = 1, 0
    v, v1 = 0, 1
    while b:
        q = a // b
        u, u1 = u1, u - q * u1
        v, v1 = v1, v - q * v1
        a, b = b, a - q * b
    return a, u, v

def decrypt(p, q, e, n, ct):
    phi = (p - 1) * (q - 1)
    gcd, a, b = egcd(e, phi)
    d = a
    if d < 0:
        d += phi
    pt = pow(ct, d, n)
    print(long_to_bytes(pt))

m1 = base64.b64decode('DK9dt2MTybMqRz/N2RUMq2qauvqFIOnQ89mLjXY=')
m2 = base64.b64decode('AK/WPYsK5ECFsupuW98bCFKYUApgrQ6LTcm3KxY=')
m3 = base64.b64decode('CiLSeTUCCKkyNf8NVnifGKKS2FJ7VnWKnEdygXY=')

p = 20016431322579245244930631426505729
q = 17963604736595708916714953362445519
e = 65537
n = 0xD564B978F9D233504958EED8B744373281ED1418B29F1ECFA8093D8CF
decrypt(p, q, e, n, bytes_to_long(m1))

p = 16549930833331357120312254608496323
q = 16514150337068782027309734859141427
e = 65537
n = 0xA23370E7D0FB00232164AC6D642840FC54E9202433F927A60EB5ADBD9
decrypt(p, q, e, n, bytes_to_long(m3))

p = 19193025210159847056853811703017693
q = 17357677172158834256725194757225793
e = 65537
n = 0xC5B69E1979E541F85DACDE2AA14D2722A846F41B3DB83E667E3B3D11D
decrypt(p, q, e, n, bytes_to_long(m2))
~~~

~~~
$ python crack.py
<junk>IW{WEAK_R
<junk>SA_K3YS_4R
<junk>3_SO_BAD!}
~~~

Concatenating the printable strings, we get the flag: **IW{WEAK_RSA_K3YS_4R3_SO_BAD!}**.

I would not have been able to solve this without the help from [yafu](https://sourceforge.net/projects/yafu/) and this fantastic [writeup](https://github.com/p4-team/ctf/tree/master/2015-09-26-trendmicro/rsa).
