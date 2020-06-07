+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-20T23:44:08+01:00"
title = "InternetWache 2016 Bank (Crypto 90) Writeup"

+++


# Problem
> Description: Everyone knows that banks are insecure. This one super secure and only allows only 20 transactions per session. I always wanted a million on my account.

(crypto90, solved by 104)

Attachment: crypto90.zip

Service: 188.166.133.53:10061

# Solution

The challenge file contains a python script that is also running on the challenge server.

Running it shows us an interface to do transactions.

We can put money into our account by creating a transaction and then signing it using a signature that we get from the server. The flag will be shown to us if our balance reaches 1000000. The problem is that we can only put 5000 units into our account, and are limited to 20 transactions.

The solution is to create a crafted signature.

Looking at the source we find that the signature contains simply a constant string and the value we want to put into our account. It is encrypted with a simple xor crypto but with an unknown key.

Since we know the plaintext prefix `TRANSACTION:` we can do a fast known plaintext attack. Using the key, we can craft a new ciphertext but with a diferent message. I changed the transaction value to 50000 units, but in hindsight i should have used 99999.

I added this in the Cipher class:
~~~python
def crack(self, ct):
  print('cracking', ct.encode('hex'))
  i = 0
  while i < 2**32:
    self.__r.set_x(i)
    pt = ""
    for c in ct:
      pt += chr(ord(c) ^ (self.__r.get_next() % 2**7))
    if pt.startswith("TRANSACTION"):
      print 'plaintext: ', pt, i
      break
    i += 1
  print "FOUND IT ", str(i)
  self.__r.set_x(i)
  pt = "TRANSACTION:50000"
  ct = ""
  for c in pt:
    ct += chr( ord(c) ^ (self.__r.get_next() % 2**7) )
  print ('ENCRYPTED', ct.encode('hex'))
  return i

...
# added this to the command parser
elif("crack" in cmd):
  a = cmd.replace("crack ","")
  ct = a.decode('hex')
  print "crack!", a
  key = C.crack(ct)
  t, i = A.create_t(50000)
  t.set_k(i)
  ct = C.encrypt(t)

~~~

I then manually ran my cracker *sigh* and crafted signatures 20 times. It worked and I got the flag.

I lost the flag for the writeup and will not run the script again...
