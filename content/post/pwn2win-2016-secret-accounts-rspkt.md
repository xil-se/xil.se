+++
authors = "rspkt"
categories = [ "writeups" ]
date = "2016-03-28T11:22:00+01:00"
title = "Pwn2Win CTF 2016 Secret Accounts (Py 80) Writeup"

+++

# Problem

> Description: Through many months of sniffing, we discovered a server running
a software which the Club uses to manage information about secret bank accounts
abroad. We even obtained its source code. We need to obtain access to the
system in order to discover the real name of the owner of the account
possessing the greater amount of money, in which bank it is, and the real
amount. As you might expect, it seems that the Club has hunkered down to assert
only authorized people, which really know what they are doing, are able to
operate this system and to interpret information provided by it. Rumors exist
that the Mentor himself manages it, as amazing as it may seem (he is old but
not deciduous!).

You will need good “python-fu” to win this challenge!

Submit the flag in the following format: CTF-BR{info1_info2_info3}.

Hint: Who is Fideleeto (Cuba!) in real life? Take this into account. :)

# Solution

The provided Python file was somewhat obfuscated with lots of unconventional
things like octal numbers, unnecessary arithmetics and so on. The first part of
the challenge was getting past the intial password prompt. The password check
algorithm was very convoluted, so we started off by translating octal to
to decimal, and simplified it:

~~~python
name=raw_input("Login name: ")
master=file("master.txt").readlines()
pwdmaster=file("passwd.txt").readlines()
master="".join(master).strip("\n")+"z"*72+"|"+"Z"*15
birthyear=master[9:13]
status=bool()

if ((name[:9] == master[:9]) and
	(name[9:13] == master[9:13]) and
	(len(name[13:]) == len(master[13:]))
	and (name[13:] > master[13:])) == False:
	status = False
else:
	status = True
~~~

So the password is 9 characters, appended by a birthyear and lots and lots of
lowercase and uppercase `Z`. There was a hint in the description referencing
Cuba. We used that to figure out the birthyear as the birthyear of Cuban leader
Fidel Castro. Since the assignment is following the theme of the CTF, the first
nine characters are likely Fideleeto. The remaining part of the password was
easy to construct. Just add the `Z`, and make sure the string input string is
"larger" than that in the `master` variable (we did this by changing one of the
uppercase `Z` to lowercase).

Once we got past the password, we were shown a prompt with different
account-related actions:

~~~
Available Options:

1 - View Owners of Accounts
2 - View Banks of Accounts
3 - Modify Sum
4 - Exit
~~~

Simply entering the number here is not sufficient, since the input numbers are
altered in the function `toption(opt)`. To figure out what input that gave what
option value, we just bruteforced it. We found the following mappings:

~~~
toption(110) = 1
toption(132) = 2
toption(144) = 3
toption(162) = 4
~~~

So now that we can provide the menu with real options, we need to enter another
password. Looking at the code, we can see that things get interesting here.
They are using the `input()` function. The Python documentation states this
this is `equivalent to eval(raw_input(prompt))`. That sounds very dangerous.
Let's see if it can give us shell ;)

We enter the following in the input prompt: `os.system('/bin/sh')`, and we're
provided a shell as expected. From there we can read the `infos` file,
containing bank account information:

~~~
{"Eduardo Cunha" :
	["Credit Suisse","5002005"],

"Dilma Roussef" :
	["Coop Bank","22582599"],

"Lul4 M0lusc0" :
	["Compagnie Bancaire Helvetique","25987369"],

"Delubio Soares" :
	["Compagnie Bancaire Helvetique","2100125"],

"Delcidio Amaral" :
	["Graubundner Kantonalbank","1780695"],

"Jose Dirceu" :
	["WIR Bank","8528314"],

"Renan Calheiros" :
	["Julius Baer", "11254255"]}
~~~

Here we can see that Lul4 M0lusc0 has the highest balance, so we enter his
information into the flag format, and get our points.
