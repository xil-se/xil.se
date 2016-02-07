+++
authors = "rspkt"
categories = [ "writeups" ]
date = "2016-02-07T14:00:00+01:00"
title = "SharifCTF 2016 Hack By The Sound (Misc 200) Writeup"
+++

# Problem

> A well known blogger has came to a hotel that we had good relationships with its staffs. > >
> We tried to capture the sound of his room by placing a microphone inside the desk.
>
> We have recorded the sound about the time that he has typed a text in his blogg. You could >
> find the text he typed in "Blog Text.txt".
>
> We reduce noises somehow and found that many characters may have the same keysound. Also >
>
> we know that he use meaningful username and password.
>
> Could you extract the username and password of his blog?
>
> flag is concatenation of his username and password as usernamepassword.

Points: 200

Solved by 16 team(s)

# Solution

For this challenge we were provided a txt-file containing the blog post, as
well as a wav-file containing the barely audible "recording" of the blogger
typing his post.

As a first step, I normalized the wav file in [Audacity](http://www.audacityteam.org).
This makes the file a bit easier to work with.

I suspected that this challenge would require lots of manual labour, such as
annotation, so after some browsing I came across [Praat](http://www.fon.hum.uva.nl/praat/),
a phonetics toolkit capable of annotating and analyzing waveforms.

It was fairly easy to map the words in the text file we were given to the
waveform. The waveform begins with the blogger entering three strings of text,
and then starts writing the post. We can assume that these three inputs are
URL, username and password.

[![Praat in action](/imgs/sharifctf-2016-misc-sound-rspkt_praat.png)](/imgs/sharifctf-2016-misc-sound-rspkt_praat.png)

Looking closer at the waveform, it seems like there's differences in the
peak amplitudes for different characters. I went ahead and wrote down the
peak intensities for different input characters, and wound up with the table
below:


| Char | Recorded intensities |
|------|----------------------|
| a    | `60.98`, `60.77`, `60.85` |
| b    | `66.16`                   |
| c    | `63.94`, `63.88`, `63.94` |
| d    | `63.90`, `63.99`, `63.88` |
| e    | `64.03`, `63.96`, `63.97` |
| f    | `65.16`, `65.09`          |
| g    |                           |
| h    | `67.15`                   |
| i    | `68.30`, `68.23`, `68.17` |
| j    | `67.74`                   |
| k    |                           |
| l    | `68.23`, `68.22`, `68.30` |
| m    | `67.69`                   |
| n    | `67.05`, `67.02`, `67.10` |
| o    | `68.73`, `68.78`, `68.76` |
| p    | `68.67`, `68.81`          |
| q    |                           |
| r    | `65.21`, `65.12`          |
| s    | `62.49`, `62.56`, `62.57` |
| t    | `66.21`, `66.17`, `66.16` |
| u    | `67.70`                   |
| v    |                           |
| w    | `62.40`                   |
| x    |                           |
| y    | `67.12`, `67.17`, `67.17` |
| z    |                           |


What's worth noting here is that the intensity seems to correlate with the
position of the character on the keyboard. Characters on the far left (`a`,
`s`, `w`) have lower intensities than those on the far right (`o`, `p`, `l`).
My theory is that this behavior simulates that the right side of the bloggers
laptop is closer to the microphone.

Another thing I noted is that characters on the same keyboard "row" have
indistinguishable intensities. The characters `e`, `d` and `c` are an example of
this. This means that we have to consider each row of characters when trying to
infer the bloggers credentials.

Going back to the credentials input, I came up with the following candidates:

| Intensity | Candidates    |
|-----------|---------------|
| 60.79     | `q`, `a`, `z` |
| 63.87     | `e`, `d`, `c` |
| 67.68     | `u`, `j`, `m` |
| 68.31     | `i`, `k`      |
| 67.20     | `y`, `h`, `n` |


Password:

| Intensity | Candidates    |
|-----------|---------------|
| 62.52     | `w`, `s`, `x` |
| 68.81     | `o`, `l`      |
| 67.69     | `u`, `j`, `m` |
| 63.88     | `e`, `d`, `c` |
| 66.21     | `t`, `g`, `b` |
| 67.15     | `y`, `h`, `n` |
| 68.27     | `i`, `k`      |
| 67.12     | `y`, `h`, `n` |
| 66.23     | `t`, `g`, `b` |


After a few seconds of ocular analysis, it's clear that the username is *admin*
and the password is *something*.

This gives us our flag, **adminsomething**.
