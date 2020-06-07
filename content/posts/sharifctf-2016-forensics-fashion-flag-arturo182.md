+++
authors = "arturo182"
categories = [ "writeups" ]
date = "2016-02-06T20:44:49+01:00"
title = "SharifCTF 2016 We lost the Fashion Flag! (Forensics 100) Writeup"
+++

# Problem

> In Sharif CTF we have lots of task ready to use, so we stored their data about author or creation date and other related information in some files. But one of our staff used a method to store data efficiently and left the group some days ago. So if you want the flag for this task, you have to find it yourself!

Attachement: A tar.gz file

# Solution

After unpacking the tarball we get two files: `sharif_tasks.tgz` and `fashion.model`, unpacking the tgz file gives us a directory of over 12000 data files. They are not very interesting, not just yet. The fashion.model file also seems to be a nondescript data, at least according to `file`. But if we look inside, we can see:

~~~none
0000000: 0800 0000 4665 6d74 6f5a 6970 0000 0000  ....FemtoZip....
~~~

A quick Google search return [the source](https://github.com/gtoubassi/femtozip) for a fzip utility, which we build and run like so:

~~~none
fzip --model fashion.model --decompress out --verbose
~~~

That unpacked all the 12000+ files! If we check any of them, they all seem to be json definitions of CTF challenges with flags included. That means we have 120k+ flags to go through! Luckily, each challenge has information such as category, points, year, so we can just grep for our current challenge:

~~~none
sed -e '$s/$/\n/' -s * | grep 2016 | grep foren | grep "'points': 100"
~~~

This returns 5 flags, which is way better thank 12000, we can try each of them. It just so happens that the first one is the correct one!

Flag: SharifCTF{2b9cb0a67a536ff9f455de0bd729cf57}