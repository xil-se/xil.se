+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-03-28T00:52:12+02:00"
description = ""
title = "Pwn2Win 2016 QRGrams Writeup"

+++

# Problem

N/A. Something about QR and anagrams.

# Solution

Connect to a server and get a bunch of ascii and ansi QR codes, then are supposed to reply with a text and some numbers.

I solved it using `qrtools`, `png` and an [anagram solver](http://code.runnable.com/UqAsOF0vlLULAAAK/anagram-solver-for-python) I found using Google. The anagram solver needs a dictionary to work, so I had to find a [sorted word list by frequency](https://github.com/first20hours/google-10000-english/blob/master/google-10000-english-usa.txt) (N-gram, the hint in the challenge name).

I generate a png image with the qrcode and then feed it to qrtools to decode a string. This string is an anagram that I then solve with the anagram solver previously mentioned.

Sorry about the mess but here's the code.

Change to the anaram solver:
```python
def process(words, word):
	jumbledWord = word
	filteredWords = [filterWordsBasedOnLength(jumbledWord,word) for word in words]
	filteredWords = filter(None,filteredWords)
        for dictionaryWord in filteredWords:
            if computeValuForEachWord(dictionaryWord,jumbledWord):
                print dictionaryWord
                return dictionaryWord
```

Solution

```python
from pwn import *
import qrtools
import png
import ana
#context.log_level = 0

r = remote('pool.pwn2win.party', 31337)

global g_i
g_i = 0

def decodeascii(data, fname):
    width = len(data[0]) * 5
    height = len(data) * 5
    f = open(fname, 'wb')
    w = png.Writer(width, height, greyscale=True)
    rows = []
    for i in range(len(data)):
        row = []
        for x in data[i]:
            col = 255 if x == '0' else 0
            row += [col]*5
        rows += [row] * 5
    w.write(f, rows)
    f.close()

def decode(data):
    global g_i
    g_i += 1
    fname = 'qr-%d.png' % g_i
    data = data.split('\n')
    if data[0][0] == '0' or data[0][0] == '1':
        print "ASCII"
        decodeascii(data, fname)

    else:
        print "ANSI"
        newdata = []
        # Mmmm, ctf code is beautiful.
        for line in data:
            line = line.replace(unhex('1b5b376d20201b5b306d'), 'O')
            line = line.replace(unhex('1b5b34396d20201b5b306d'), 'I')
            line = line.replace('O', '0')
            line = line.replace('I', '1')
            newdata.append(line)
            #print "[%s]" % line
        decodeascii(newdata, fname)

    x = qrtools.QR(filename=fname)
    x.decode()
    print x.data
    return x.data


words = [x.strip() for x in open("google-10000-english-usa.txt", "r").read().split()]
words += [x.strip() for x in open("wordsEn.txt", "r").read().split()]


postfix = []
answers = []
part = ""
r.recvuntil('\n')
hax = True
while hax:
    ret = r.recv()
    if ret.find('Phrase') > -1:
        print "Phrase!"
        hax = False
    part += ret
    #print part
    nn = part.find('\n\n')
    if nn >= 0:
        x = part[:nn].strip()
        part = part[nn+2:]
    elif not hax:
        nn = part.find('Phrase')
        x = part[:nn].strip()
    else:
        continue

    append = ""
    print "qr: [%s]" % x
    if x == '': continue
    y = decode(x)

    try:
        int(y)
        postfix.append(y)
    except:
        pass

    rep="0123456789"
    for ch in y:
        if ch in rep:
            y=y.replace(ch,'')

    if y == '': continue

    cap = False
    rep="ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    for ch in y:
        if ch in rep:
            cap = True

    y = y.lower()

    if y.find('.') >= 0:
        y = y.replace(".", "")
        append = "."
    if y.find(',') >= 0:
        y = y.replace(",", "")
        append = ","
    if y.find('!') >= 0:
        y = y.replace("!", "")
        append = "!"
    if y.find('?') >= 0:
        y = y.replace("?", "")
        append = "?"
    if y.find(';') >= 0:
        y = y.replace(";", "")
        append = ";"

    decoded = ana.process(words, y) + append
    if cap:
        if len(decoded) > 1:
            decoded = decoded[0].upper() + decoded[1:]
        else:
            decoded = decoded[0].upper()

    print y, decoded
    answers += [decoded]
    print "progress: [%s]" % ' '.join(answers)

xx = ' '.join(answers)
print xx
xx = xx + " " + postfix[0] + " " + postfix[1]
print xx
r.sendline(xx)

print r.recv()
print r.recv()

```
