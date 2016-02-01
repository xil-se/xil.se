+++
date        = "2016-01-31"
title       = "HackIM 2016 Programming 5 Writeup"
description = "Programming 5 - 500pts"
categories  = [ "writeups" ]
tags        = [ "ctf", "hackim" ]
authors     = "arturo182"
+++

# Problem
> Dont blink your Eyes, you might miss it. But the fatigue and exhaustion rules out any logic, any will to stay awake. What you need now is a slumber. Cat nap will not do. 1 is LIFE and 0 is DEAD. in this GAME OF LIFE sleep is as important food. So... catch some sleep. But Remember...In the world of 10x10 matirx, the Life exists. If you SLOTH, sleep for 7 Ticks, or 7 Generation, In the game of Life can you tell what will be the state of the world?  
> The world- 10x10
> 0000000000,0000000000,0001111100,0000000100,0000001000,0000010000,0000100000,0001000000,0000000000,000000000

# Solution

Obviously we need to seed a [Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) with the specified value and the flag will be the state of the game after 7 generations.

Quick Google search for "game of life 10x10" returns this website: http://www.sitepoint.com/conways-game-life/  
Which allows us to download a JavaScript version, we change the seed to the one specified, run 7 generations and voila!

Flag: 0000000000,0001100000,0001111010,0000001001,0000001010,0000000000,0000000000,0000000000,0000000000,0000000000

