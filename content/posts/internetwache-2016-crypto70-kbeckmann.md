+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-20T23:29:14+01:00"
title = "InternetWache 2016 A numbers game II (Crypto 70) Writeup"

+++

# Problem

> Description: There was this student hash design contest. All submissions were crap, but had promised to use the winning algorithm for our important school safe. We hashed our password and got '00006800007d'. Brute force isn't effective anymore and the hash algorithm had to be collision-resistant, so we're good to go, aren't we?

(crypto70, solved by 86)

Attachment: crypto70.zip

Service: 188.166.133.53:10009

# Solution

Connecting to the service we are faced with a bruteforce blocker / challenge.

~~~
$ nc 188.166.133.53 10009
You need to provide your proof of work: A sha1 hash with the last two bytes set to 0. It has 36626829 as the prefix. It should match ^[0-9a-z]{15}$
Enter work:
~~~

We wrote a hacky php script to solve this.

~~~php
<?php
function randomPassword() {
    $alphabet = '0123456789abcdefghijklmnopqrstuvwxyz';
    $pass = array();
    $alphaLength = strlen($alphabet) - 1;
    for ($i = 0; $i < 7; $i++) {
        $n = rand(0, $alphaLength);
        $pass[] = $alphabet[$n];
    }
    return implode($pass);
}

while(true){
	$pass = randomPassword();
	$pass = $argv[1] . $pass;

	$hash = sha1($pass);
	if(preg_match('/0000$/', $hash)){
		echo $pass, "\n";
	}
}
~~~

Looking at the challege zip file we find a python script that computes a "hash" of some sort. Our task is to find a plaintext that generates the hash `00006800007d`.

I decided to bruteforce it. After successfully finding a plaintext that generated the correct hash, the server responded that it was too short. It had to be at least 18 characters long. So i changed the script.

I added the following lines to the provided script.

~~~python
import itertools

...

c = [chr(x) for x in range(ord('A'), ord('z') + 1)]
combos = [''.join(c) for c in itertools.combinations(c, 4)]
for a in combos:
	x = "A"*18 + a
	hash = myhash(x)
	if hash == '00006800007d':
		print x
		break
~~~

The output will be `AAAAAAAAAAAAAAAAAAMdnx` after a couple of minutes depending on your hardware.

Entering the generated output gives us the flag:
~~~
$ nc 188.166.133.53 10009
You need to provide your proof of work: A sha1 hash with the last two bytes set to 0. It has 30888409 as the prefix. It should match ^[0-9a-z]{15}$
Enter work:308884090euymhc
Thank you. Please continue with the login process...
Password: AAAAAAAAAAAAAAAAAAMdnx
Logged in!
IW{FUCK_YOU_HASH_MY_ASS}
~~~

Flag is **IW{FUCK_YOU_HASH_MY_ASS}**.
