+++
author = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-21T00:02:30+01:00"
title = "InternetWache 2016 EquationSolver (Exploit 60) Writeup"

+++

# Problem

> Description: I created a program for an unsolveable equation system. My friend somehow forced it to solve the equations. Can you tell me how he did it?

(exp60, solved by 252)

Service: 188.166.133.53:12049

# Solution

Interacting with the server, we see the following:

~~~
$ nc 188.166.133.53 12049
Solve the following equations:
X > 1337
X * 7 + 4 = 1337
Enter the solution X: 190
You entered: 190
190 is not bigger than 1337
WRONG!!!
Go to school and learn some math!
~~~

I tried a couple of overflow/underflow things but I couldn't get it right. So I bruteforced it instead with this C program:
~~~c++
#include <stdio.h>
#include <stdint.h>

int test(int32_t a) {
  return ((a*7+4) == 1337);
}

void main() {
  int i = 0;
  while(++i != 0) {
    if (test(i)) {
      printf("found it: %d\n", i);
    }
  }

  printf("done. %d\n", i);
}
~~~

~~~
$ ./crack
found it: 613566947

$ nc 188.166.133.53 12049
Solve the following equations:
X > 1337
X * 7 + 4 = 1337
Enter the solution X: 613566947
You entered: 613566947
613566947 is bigger than 1337
1337 is equal to 1337
Well done!
IW{Y4Y_0verfl0w}
~~~

Flag is **IW{Y4Y_0verfl0w}**.
