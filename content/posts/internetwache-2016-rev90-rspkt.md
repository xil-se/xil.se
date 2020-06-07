+++
author = "rspkt"
categories = [ "writeups" ]
date = "2016-02-21T18:18:38+01:00"
title = "Internetwache CTF 2016 The Cube (Rev 90) Writeup"

+++

# Problem

> Description: I really like Rubik's Cubes, so I created a challenge for you. I put the flag on the white tiles and scrambled the cube. Once you solved the cube, you'll know my secret.

(rev90, solved by 232)

Attachment: rev90.zip

# Solution

In this task we were given a text-file containing the scrambling of a Rubik's
cube, as well as the different sides of the cube before it was scrambled.

    Scrambling:
    F' D' U' L' U' F2 B2 D2 F' U D2 B' U' B2 R2 D2 B' R' U B2 L U R' U' L'


    White side:
    -------
    |{|3| |
    | |D|R|
    | |W| |
    -------

    Orange side:
    -------
    | | | |
    | | | |
    | | | |
    -------

    Yellow side:
    -------
    |}| | |
    |3| | |
    | | | |
    -------


    Red side:
    -------
    |I| | |
    | | | |
    | | |C|
    -------

    Green side:
    -------
    | | | |
    | | | |
    | | | |
    -------

    Blue side:
    -------
    | | | |
    | | | |
    | | | |
    -------


First observation is that the flag is five characters long, since there are
nine characters spread out over the cube, and of these, four are `IW{}`.
This makes it really easy to brute-force this challenge, since there are only
120 possible permutations. However, brute-forcing is no fun.

Another way would be to solve this using a physical Rubik's cube. In the lack
of one of those, we decided to go with the programmatic solution.

The implementation is pretty straight forward. First we define a representation
of the cube, which naturally is a 6x3x3 array (or list). We define the six
different rotations in different functions. Some valuable observations where are:

* Each rotation of a side affects the four adjacent sides
* The meaning of each rotation depends on the starting state of the cube
* The scrambling needs to be done in reverse in order to de-scramble the cube
* Counter-clockwise rotations (marked with prim-sign) are equivalent to three
  clockwise rotations, and double rotations are reversed with double rotations

Internetwache were kind enough to supply us with the starting state
[in a tweet](https://twitter.com/internetwache/status/701062084969283585).
The starting state of the cube was:

Front: green
Right: red
Up: white
Back: blue
Left: orange
Down: yellow

Having implemented all rotations, and knowing the starting state, we can solve
the cube. Using the provided script, the following output was given:

~~~python
[' ', ' ', ' ']
[' ', ' ', ' ']
[' ', ' ', ' ']

[' ', ' ', ' ']
[' ', ' ', ' ']
[' ', ' ', ' ']

['I', 'W', '{']
['3', 'D', 'R']
['C', '3', '}']

[' ', ' ', ' ']
[' ', ' ', ' ']
[' ', ' ', ' ']

[' ', ' ', ' ']
[' ', ' ', ' ']
[' ', ' ', ' ']

[' ', ' ', ' ']
[' ', ' ', ' ']
[' ', ' ', ' ']
~~~

This gives us the flag, IW{3DRC3}.


Code
~~~python
# The cube
sides = {
    'W': [
        ['{', '3', ' '],
        [' ', 'D', 'R'],
        [' ', 'W', ' '],
    ],

    'O': [
        [' ', ' ', ' '],
        [' ', ' ', ' '],
        [' ', ' ', ' '],
    ],

    'Y': [
        ['}', ' ', ' '],
        ['3', ' ', ' '],
        [' ', ' ', ' '],
    ],

    'R': [
        ['I', ' ', ' '],
        [' ', ' ', ' '],
        [' ', ' ', 'C'],
    ],

    'G': [
        [' ', ' ', ' '],
        [' ', ' ', ' '],
        [' ', ' ', ' '],
    ],

    'B': [
        [' ', ' ', ' '],
        [' ', ' ', ' '],
        [' ', ' ', ' '],
    ],
}


# This rotates all squares on one side
def rot(s):
    return [[s[2-i][j] for i in range(3)] for j in range(3)]


# These functions rotate adjacent sides
def F(c):
    tmp = (c[2][2][0], c[2][2][1], c[2][2][2])

    c[2][2][0], c[2][2][1], c[2][2][2] = c[4][2][2], c[4][1][2], c[4][0][2]
    c[4][2][2], c[4][1][2], c[4][0][2] = c[5][0][2], c[5][0][1], c[5][0][0]
    c[5][0][2], c[5][0][1], c[5][0][0] = c[1][0][0], c[1][1][0], c[1][2][0]
    c[1][0][0], c[1][1][0], c[1][2][0] = tmp

    c[0] = rot(c[0])
    return c


def R(c):
    tmp = (c[2][2][2], c[2][1][2], c[2][0][2])

    c[2][2][2], c[2][1][2], c[2][0][2] = c[0][2][2], c[0][1][2], c[0][0][2]
    c[0][2][2], c[0][1][2], c[0][0][2] = c[5][2][2], c[5][1][2], c[5][0][2]
    c[5][2][2], c[5][1][2], c[5][0][2] = c[3][0][0], c[3][1][0], c[3][2][0]
    c[3][0][0], c[3][1][0], c[3][2][0] = tmp

    c[1] = rot(c[1])
    return c


def U(c):
    tmp = (c[3][0][2], c[3][0][1], c[3][0][0])

    c[3][0][2], c[3][0][1], c[3][0][0] = c[4][0][2], c[4][0][1], c[4][0][0]
    c[4][0][2], c[4][0][1], c[4][0][0] = c[0][0][2], c[0][0][1], c[0][0][0]
    c[0][0][2], c[0][0][1], c[0][0][0] = c[1][0][2], c[1][0][1], c[1][0][0]
    c[1][0][2], c[1][0][1], c[1][0][0] = tmp

    c[2] = rot(c[2])
    return c


def B(c):
    tmp = (c[2][0][2], c[2][0][1], c[2][0][0])

    c[2][0][2], c[2][0][1], c[2][0][0] = c[1][2][2], c[1][1][2], c[1][0][2]
    c[1][2][2], c[1][1][2], c[1][0][2] = c[5][2][0], c[5][2][1], c[5][2][2]
    c[5][2][0], c[5][2][1], c[5][2][2] = c[4][0][0], c[4][1][0], c[4][2][0]
    c[4][0][0], c[4][1][0], c[4][2][0] = tmp

    c[3] = rot(c[3])
    return c


def L(c):
    tmp = (c[2][0][0], c[2][1][0], c[2][2][0])

    c[2][0][0], c[2][1][0], c[2][2][0] = c[3][2][2], c[3][1][2], c[3][0][2]
    c[3][2][2], c[3][1][2], c[3][0][2] = c[5][0][0], c[5][1][0], c[5][2][0]
    c[5][0][0], c[5][1][0], c[5][2][0] = c[0][0][0], c[0][1][0], c[0][2][0]
    c[0][0][0], c[0][1][0], c[0][2][0] = tmp

    c[4] = rot(c[4])
    return c


def D(c):
    tmp = (c[0][2][0], c[0][2][1], c[0][2][2])

    c[0][2][0], c[0][2][1], c[0][2][2] = c[4][2][0], c[4][2][1], c[4][2][2]
    c[4][2][0], c[4][2][1], c[4][2][2] = c[3][2][0], c[3][2][1], c[3][2][2]
    c[3][2][0], c[3][2][1], c[3][2][2] = c[1][2][0], c[1][2][1], c[1][2][2]
    c[1][2][0], c[1][2][1], c[1][2][2] = tmp

    c[5] = rot(c[5])
    return c


def print_cube(c):
    for side in c:
        for row in side:
            print row

        print ""


def solve(c):
    # Original:
    # F' D' U' L' U' F2 B2 D2 F' U D2 B' U' B2 R2 D2 B' R' U B2 L U R' U' L'

    # Reverse:
    # L U R U' L' B2 U' R B D2 R2 B2 U B D2 U' F D2 B2 F2 U L U D F

    c = L(c)        # L
    c = U(c)        # U
    c = R(c)        # R
    c = U(U(U(c)))  # U'
    c = L(L(L(c)))  # L'
    c = B(B(c))     # B2
    c = U(U(U(c)))  # U'
    c = R(c)        # R
    c = B(c)        # B
    c = D(D(c))     # D2
    c = R(R(c))     # R2
    c = B(B(c))     # B2
    c = U(c)        # U
    c = B(c)        # B
    c = D(D(c))     # D2
    c = U(U(U(c)))  # U'
    c = F(c)        # F
    c = D(D(c))     # D2
    c = B(B(c))     # B2
    c = F(F(c))     # F2
    c = U(c)        # U
    c = L(c)        # L
    c = U(c)        # U
    c = D(c)        # D
    c = F(c)        # F


if __name__ == '__main__':
    # Color of sides in format:
    # Front, right, up, back, left, down
    starting_pos = "GRWBOY"
    c = []
    for ch in starting_pos:
        c.append(sides[ch])
    solve(c)
    print_cube(c)
~~~
