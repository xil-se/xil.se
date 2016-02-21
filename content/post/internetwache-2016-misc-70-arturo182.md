+++
authors = "arturo182"
categories = [ "writeups" ]
date = "2016-02-21T13:33:00+01:00"
title = "Internetwache CTF 2016 Rock With The Wired Shark (Misc 70) Writeup"
+++

# Problem

>  Someone sent me a file with white and black rectangles. I don't know how to read it. Can you help me?

Attachment: https://ctf.internetwache.org/files/misc70.zip

Solved by 454 teams

# Solution

What we get is a packet capture file. I usually approach those with python, but this one was easier to do with just Wireshark.

But first, let's run `strings` on it for good measure, two things are interesting:

~~~none
Authorization: Basic ZmxhZzphenVsY3JlbWE=
...snip...
Content-Disposition: attachment; filename="flag.zip"
~~~

The authorization base64 translates to `flag:azulcrema`.

Now we use Wireshark to extract all HTTP objects, we get two html files and `flag.zip`. When trying to extract the zip, it asks for a password. What could it be? Hmmm. Let's try `azulcrema`... voila! We get a `flag.txt`.

Flag: IW{HTTP_BASIC_AUTH_IS_EASY}

