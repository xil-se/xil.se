+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-02-06T23:13:14+01:00"
title = "SharifCTF 2016 Sec-Coding 2 (Misc 300) Writeup"
+++

# Problem

> You should fix vulnerabilities of the given source code, WITHOUT changing its normal behaviour. Link

Points: 300

Solved by 83 team(s)

# Solution

Just like the previous Sec-Coding 1, we get a link to a web UI where we should upload patched C++ code.

We are provided with windows C++ code that prints a repeated message based on the input.

The original code was pretty bad so I simply rewrote it completely and this solved all the security issues. There was a bug that I had to re-implement. If repeat count%10==0 the message should be blank.

The binary also had to scream if the input was invalid. Empty output was not accepted.

~~~c++

#include <math.h>
#include <stdio.h>
#include <cstdlib>
#include <cstdio>
#include <cstring>

void die(const char* s) {
	printf("%s", s);
	exit(-1);
}

int main(int argc, char **argv)
{
	// STRING ECHO
	//
	// Sample usage:
	//   strecho repeat=4,str=pleaseechome

	if (argc != 2) die("1");

	char *line = argv[1];
	int line_len = strlen(line);
	char *line_end = line + line_len;

	if (line_end - line < 7) die ("2");

	if (strncmp(line, "repeat=", 7)) die("3");
	line += 7;

	int repeat = atoi(line);

	if (repeat <= 0) die("4");

	char *comma = strchr(line, ',');
	if (!comma) die("5");
	line = comma + 1;
	if (line >= line_end) die("6");

	if (strncmp(line, "str=", 4)) die("7");
	line += 4;

	if (line >= line_end) die("8");

	const char *str = line;
	if (repeat % 10 == 0) str = ""; // intentional bug

	int i;
	for (i = 0; i < repeat; i++) {
		printf("%s\n", str);
	}

	return -14;
}
~~~

*Flag is 983027cec2889d128529c078eebb6471*
