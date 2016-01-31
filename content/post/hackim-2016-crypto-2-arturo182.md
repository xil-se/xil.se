+++
date        = "2016-01-31"
title       = "HackIM 2016 Crypto 2 Writeup"
description = "Crypto 2 - 400pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim" ]
+++

# Problem
> Some one was here, some one had breached the security and had infiltrated here. All the evidences are touched, Logs are altered, records are modified with key as a text from book.The Operation was as smooth as CAESAR had Conquested Gaul. After analysing the evidence we have some extracts of texts in a file. We need the title of the book back, but unfortunately we only have a portion of it...

Attachement:
http://ctf.nullcon.net/crypto/The_extract.txt

# Solution

Again, a very subtle hint at CAESAR's cipher, we can use one of the online tools available, for example [this one](http://www.xarg.org/tools/caesar-cipher/), which actually has a "guess" option, and will try to find the proper key.  
The tool quickly guesses key 16, which gives us a paragraph. As a *connoisseur* of Contemporary Romance, I didn't even have to Google to know that it was from one of my favourites: "In the Shadow of Greed (Crimson Romance)" by Nancy C. Weeks.

Flag: Shadow of Greed
