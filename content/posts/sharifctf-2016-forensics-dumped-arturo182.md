+++
author = "arturo182"
categories = [ "writeups" ]
date = "2016-02-06T20:38:49+01:00"
title = "SharifCTF 2016 Dumped (Forensics 100) Writeup"
+++

# Problem

> In Windows Task Manager, I right clicked a process and selected "Create dump file". I'll give you the dump, but in return, give me the flag!

Attachment: A compressed Windows process dump

# Solution

I'm sure the creators had something very interesting in mind, however it seems they didn't verify all the possibilities, because after extracting the xz file, I was able to run:

    strings RunMe.DMP | grep -E "^SharifCTF{[0-9a-f]{32}}$"

Which returned the flag: `SharifCTF{4d7328869acb371ede596d73ce0a9af8}`

End that was the end of it
