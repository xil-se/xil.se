+++
authors = "arturo182"
categories = [ "writeups" ]
date = "2016-02-21T12:19:00+01:00"
title = "Internetwache CTF 2016 Eso Tape (Rev 80) Writeup"
+++

# Problem

>  I once took a nap on my keyboard. I dreamed of a brand new language, but I could not decipher it nor get its meaning. Can you help me? Hint: Replace the spaces with either '{' or '}' in the solution.

Attachment: https://ctf.internetwache.org/files/rev80.zip

Solved by 95 teams

# Solution

We unpack and get a priner.tb file looking like this:

~~~none
## %% %++ %++ %++ %# *&* @** %# **&* ***-* ***-* %++ %++ @*** *-* @*** @** *+** @*** ***+* @*** **+** ***+* %++ @*** #% %% %++ %++ %++ %++ @* %# %++ %++ %++ %% *&** @* @*** *-** @* %# %++ @** *-** *-** **-*** **-*** **-*** @** @*** #% %% %++ %++ %++ %++ %# *+** %++ @** @* %# *+** @*** ## %% @***
~~~

Looks like an esoteric language, the name of the challenge supports the assumption.

After some googling, we find [TapeBagel](https://esolangs.org/wiki/TapeBagel) which fits the description perfectly. The problem is that there is only one interpreter (Oh!), which is not complete (Oh!) and is written in Perl (Oh!!).

And I'm saying this for like the 5th time today: python to the rescue!

At the time of the challenge we hacked together a quick and dirty interpreter that worked only with the exact instructions we got in the challenge, but since I found it really fun to do, I cleaned up the code and added all the remaining functionality (except pause program, because there doesn't seem to be a resume command).

~~~python
from sys import stdout

tape = [0, 0, 0]
index = 0

f = open('priner.tb')
lines = f.read().replace('\n', '')
code = lines.split(' ')

def do_math(instr, op, func):
    ints = instr.split(op)

    index1 = len(ints[0]) - 1
    index2 = len(ints[1]) - 1

    tape[index] = func(tape[index1], tape[index2])

for instr in code:
    if len(instr) == 0:
        continue

    if instr[0] == '#':
        if instr[1] == '%':
            tape = [1, 1, 1]
        elif instr[1] == '#':
            tape = [0, 0, 0]
    elif instr[0] == '%':
        if instr[1:3] == '++':
            tape[index] = tape[index] + 1
        elif instr[1:3] == '--':
            tape[index] = tape[index] - 1
        elif instr[1] == '#':
            index = index + 1
        elif instr[1] == '%':
            index = 0
        elif instr[1] == '&':
            inp = raw_input()
            tape[index] = int(inp)
    elif instr[0] == '@':
        if instr[1] == '@':
            if len(instr) == 3:
                stdout.write(str(tape[0]))
            elif len(instr) == 4:
                stdout.write(str(tape[1]))
            elif len(instr) == 5:
                stdout.write(str(tape[2]))
        else:
            if len(instr) == 2:
                stdout.write(chr(ord('@') + tape[0]))
            elif len(instr) == 3:
                stdout.write(chr(ord('@') + tape[1]))
            elif len(instr) == 4:
                stdout.write(chr(ord('@') + tape[2]))
    elif instr[0] == '&':
        if instr[1] == '@':
            stdout.write('\r')
    elif '&' in instr:
        do_math(instr, '&', lambda x, y: x * y)
    elif '+' in instr:
        do_math(instr, '+', lambda x, y: x + y)
    elif '-' in instr:
        do_math(instr, '-', lambda x, y: x - y)
    elif '$' in instr:
        do_math(instr, '$', lambda x, y: x / y)
    else:
        print instr

stdout.write('\n')
~~~

One more thing worth noticing is that it seems the code supplied by InternetWache interpreted the language definition slightly different than the example perl implementation. It's because the specification was not very clear.

For math operations, it was not specified where the result should be stored. The perl code stored the result back in the first operand, which mean that instruction like this `***+**` was interpreted as `tape[3] = tape[3] + tape[2]`, but InternetWache's code assumed that the result should be stored at the current index: `tape[index] = tape[3] + tape[2]`, which I think makes more sense.

My implementation was written to work with the code we were supplied with, so it makes the same assumptions.

That being said, the Hello World code from the wiki still works with my implementation, because it does not use any math instructions.

When we run the interpreter we get `IW@ILOVETAPEBAGEL@`.

Flag: IW{ILOVETAPEBAGEL}

Sidenote: Check this awesome program in TapeBagel I wrote, it ask for 2 numbers and prints the sum of them! :D

~~~none
## %& %# %& %% *+** @@*
~~~