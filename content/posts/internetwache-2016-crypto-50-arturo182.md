+++
author = "arturo182"
categories = [ "writeups" ]
date = "2016-02-21T14:20:00+01:00"
title = "Internetwache CTF 2016 Crypto-Pirat (Crypto 50) Writeup"
+++

# Problem

>  Did the East German Secret Police see a Pirat on the sky? Help me find out!

Attachment: https://ctf.internetwache.org/files/crypto50.zip

# Solution

Sidenote: This one took us some time, we started ok but then our progress grinded to a halt because we just didn't know what to do with the output we got. The organizers of the CTF agreed that the challenge was not clear enough and after 12 hours added two hints:

> Hint: We had 9 planets from 1930–2006...
> Hint2: Each planet has a number. (There's a table on a well-known website). After that you might be interested in ciphers used by the secret police.

Before the hints we figured out the first part (planets to numbers) but we had no idea what to do with the numbers. The secret police hint was very helpful.

On to the challenge!

We unzip and get a text file with pretty Unicode characters:
~~~none
♆♀♇♀♆ ♇♇♀♆⊕ ♇♀♇♀♆ ♇♆♇♆⊕ ♆♇♆♇♇ ♀♆♇♆⊕ ♆♇♆♇♆ ♇♆♇♆⊕ ♆♇♇♀♇ ♀♆⊕♇♀ ♆⊕♇♀♆ ⊕♆♇♆♇ ♇♀♆♇♆ ⊕♇♀♇♀ ♆⊕♆♇♆ ♇♆♇♇♀ ♆⊕♆♇♆ ♇♆♇♆⊕ ♆♇♆♇♆ ♇♆⊕♇♀ ♆♇♇♀♆ ♇♆⊕♇♀ ♆♇♆♇♇ ♀♆⊕♆♇ ♆♇♇♀♇ ♀♇♀♆⊕ ♆♇♆♇♇ ♀♆⊕♇♀ ♇♀♆♇♆ ⊕♆♇♇♀ ♆⊕♇♀♆ ♇♇♀♇♀ ♆⊕♆♇♆ ♇♆♇♆♇ ♆⊕♇♀♆ ♇♇♀♆♇ ♆⊕♇♀♆ ♇♆♇♇♀ ♆⊕♆♇♆ ♇♇♀♇♀ ♇♀♆⊕♇ ♀♆♇♆♇ ♆⊕♇♀♇ ♀♇♀♆⊕ ♇♀♇♀♆ ♇♆♇♆⊕ ♆♇♆♇♆ ♇♆⊕♆♇ ♇♀♇♀♆ ⊕♆♇♆♇ ♆♇♆♇♇ ♀♆⊕♇♀ ♇♀♆♇♆ ♇♆⊕♆♇ ♆♇♆♇♇ ♀♇♀♆⊕ ♆♇♆♇♆ ♇♆⊕♆♇ ♇♀♆♇♆ ♇♆⊕♆♇ ♆♇♆♇♆ ♇♆♇♆⊕ ♆♇♇♀♇ ♀♆⊕♇♀ ♇♀♆♇♆ ⊕♆♇♆⊕ ♆♇♆♇♇ ♀♇♀♇♀ ♆⊕♇♀♆ ♇♇♀♆♇ ♆⊕♇♀♇ ♀♆♇♆♇ ♆♇♆⊕♇ ♀♆♇♆⊕ ♇♀♇♀♆ ♇♆♇♆⊕ ♆♇♆♇♆ ♇♆⊕♇♀ ♆♇♆♇♇ ♀♆⊕♆♇ ♆⊕♇♀♇ ♀♇♀♆⊕ ♆♇♇♀♆ ♇♆⊕♆♇ ♇♀♇♀♇ ♀♆⊕♆♇ ♇♀♇♀♆ ♇♆⊕♆♇ ♆♇♇♀♆ ⊕♇♀♆♇ ♆♇♆♇♇ ♀♆⊕♇♀ ♆♇♆♇♆ ♇♇♀♆⊕ ♇♀♆♇♆ ♇♆♇♇♀ ♆⊕♇♀♆ ♇♆♇♆♇ ♇♀♆⊕♇ ♀♆♇♆♇ ♆♇♇♀♆ ⊕♇♀♆♇ ♆♇♆♇♇ ♀
~~~

There are 4 unique characters in the file, they are all symbols of planets in our solar system:

| Symbol | Planet | Number of planet |
|------|-----------------------------|
| ♀    | Venus    | 2                |
| ⊕   | Earth    | 3                |
| ♆   | Neptune  | 8                 |
| ♇   | Pluto    | 9                |

We replace the characters with their numbers and get:
~~~none
82928 99283 92928 98983 89899 28983 89898 98983 89929 28392 83928 38989 92898 39292 83898 98992 83898 98983 89898 98392 89928 98392 89899 28389 89929 29283 89899 28392 92898 38992 83928 99292 83898 98989 83928 99289 83928 98992 83898 99292 92839 28989 83929 29283 92928 98983 89898 98389 92928 38989 89899 28392 92898 98389 89899 29283 89898 98389 92898 98389 89898 98983 89929 28392 92898 38983 89899 29292 83928 99289 83929 28989 89839 28983 92928 98983 89898 98392 89899 28389 83929 29283 89928 98389 92929 28389 92928 98389 89928 39289 89899 28392 89898 99283 92898 98992 83928 98989 92839 28989 89928 39289 89899 2
~~~

This is where we got stuck, there were just too many things that could be done with this data and too many ways it could have been interpreted.

After the hint, we managed to find a [code](https://rgpsecurity.wordpress.com/2014/10/17/stasi-vernam-cipher-gernator-tapir/) used by the [Stasi](https://en.wikipedia.org/wiki/Stasi) (East German Secret Police). It was named TAPIR (anagram of PIRAT, from the challenge's name) and it used numbers grouped in quintuples, it was perfect.

The code uses a special table to convert 1- and 2-digit sequences into characters.
![TAPIR](/imgs/internetwache-2016-crypto-50-arturo182_tapir.jpg)

When we merged all the 5-digit groups into one string and divided them into 2-digit groups, we found out that there are only 4 unique groups:

| Number | Meaning                                           |
|--------|---------------------------------------------------|
| 82     | Zi - Switch to numbers (green in the above table) |
| 83     | ZwR - space                                       |
| 89     | . (period)                                        |
| 92     | - (dash)                                          |

With that knowledge, we could use python to decipher the numbers:

~~~python
# coding=utf8

f = open('README.txt')
line = f.readline().replace('\n', '')
s = line.replace(' ', '')

# Unicode, in my code? How unorthodox!
s = s.replace('♀', '2')
s = s.replace('⊕', '3')
s = s.replace('♆', '8')
s = s.replace('♇', '9')

words = []
this = '' # C++/C#/Java/JavaScript programmers shed a single tear
for n in [s[i:i+2] for i in range(0, len(s), 2)]:
    if n == '82':
        continue
    elif n == '92':
        this = this + '-'
    elif n == '89':
        this = this + '.'
    elif n == '83':
        words.append(this)
        this = ''

print ' '.join(words)
~~~

The output:
~~~none
-.- --.. ..-. .... .-- - - ..-. -- ...- ... ... -.-. -..- ..--- ..- --. .- -.-- .... -.-. -..- ..--- -.. --- --.. ... .-- ....- --.. ...-- ... .-.. ..... .-- --. . ..--- -.-. --... -. --.. ... -..- . --- .-. .--- .--. ..- -...- -...- -...- -...- -...-
~~~

Which is morse code for:
~~~none
KZFHWTTFMVSSCX2UGAYHCX2DOZSW4Z3SL5WGE2C7NZSXEORJPU====
~~~

Which is base32 for:
~~~none
VJ{Neee!_T00q_Cvengr_lbh_ner:)}
~~~

Which is rot-13 for:
~~~none
IW{Arrr!_G00d_Pirate_you_are:)}
~~~

Flag: IW{Arrr!_G00d_Pirate_you_are:)}
