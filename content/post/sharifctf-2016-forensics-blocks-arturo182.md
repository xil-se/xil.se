+++
authors = "arturo182"
categories = [ "writeups" ]
date = "2016-02-06T20:47:49+01:00"
title = "SharifCTF 2016 Blocks (Forensics 400) Writeup"
+++

# Problem

> I recovered as much data as I could. Can you recover the flag?

Attachment: A data blob

# Solution

To start with, `file` tells us nothing about the file, so we pop it into a hex editor and we can see familiar things:

~~~none
...snip...
0000230: 8114 0407 1715 1501 820b 7461 626c 6564  ..........tabled
0000240: 6174 6164 6174 6105 4352 4541 5445 2054  atadata.CREATE T
0000250: 4142 4c45 2022 6461 7461 2220 280a 0960  ABLE "data" (..`
0000260: 4944 6009 494e 5445 4745 5220 4e4f 5420  ID`.INTEGER NOT
0000270: 4e55 4c4c 2050 5249 4d41 5259 204b 4559  NULL PRIMARY KEY
0000280: 2041 5554 4f49 4e43 5245 4d45 4e54 2055   AUTOINCREMENT U
0000290: 4e49 5155 452c 0a09 6044 6174 6160 0942  NIQUE,..`Data`.B
...snip...
~~~

So, it seems it's a SQLite database, but the header magic (section 1.2 [here](https://www.sqlite.org/fileformat2.html)) is missing, let's look closely.

A proper file:

~~~none
0000000: 5351 4c69 7465 2066 6f72 6d61 7420 3300  SQLite format 3.
0000010: 0400 0101 0040 2020 0000 000b 0000 000b  .....@  ........
0000020: 0000 0000 0000 0000 0000 0002 0000 0004  ................
~~~

Our file:

~~~none
0000000: 2033 0004 0001 0100 4020 2000 0000 0b00   3......@  .....
0000010: 0000 0b00 0000 0000 0000 0000 0000 0200  ................
0000020: 0000 0400 0000 0000 0000 0000 0000 0100  ................
~~~

Seems that "SQLite format " is missing, we add it and get a working database file!  
Now let's check the contents:

~~~none
sqlite> .schema
CREATE TABLE `category` (
        `ID` INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
        `Cat` TEXT NOT NULL
);
CREATE TABLE "data" (
        `ID` INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
        `Data` BLOB NOT NULL,
        `Cat` INTEGER NOT NULL
);

sqlite> SELECT * FROM category;
1|CHRM
2|IDAT
3|ICCP
4|IHDR
5|TEXT
6|TIME
7|PLTE
8|TRNS

sqlite> SELECT ID, length(hex(Data))/2, Cat FROM data;
1|265|2
2|265|2
3|265|2
4|265|2
5|265|2
6|768|7
7|13|4
8|265|2
9|265|2
10|255|2
11|265|2
12|2|8
13|265|2
14|265|2
~~~

So it looks like we have a make-your-own-PNG kit, categorised chunks stored as BLOBs. After looking around more, we could say the following:

- We have one IHDR chunk
- We have one PLTE chunk
- We have one tRNS chunk
- We have 11 IDAT chunks
 
We're in luck, that's just enough to make a PNG!

I have to admit, this challenge, took me a long time to figure out, mostly because I had no idea what order the IDAT chunks should be, maybe there is some way to determine that based on their header flags, but I could not have get it working, so in the end, a good old try and error saved the day.

Before the final code, let's stop a second and talk about the PNG format, because it's a pretty neat one.  
A PNG file consists of parts called chunks, each of which has to be stored like so:

- Length: A four-byte unsigned integer, specifies the size of the Data field
- Type: Four byte ASCII sequence
- Data: Whatever it needs to be, depends on the Type
- CRC: A four byte integer, calculated from Type and Data fields using the crc32 hash function

With that information the code should be easy to understand:

~~~python
import sqlite3
import struct
import zlib
import struct
 
db = sqlite3.connect('data2')
 
def get_chunk(typ, data):
    ret = struct.pack('>l', len(data))
    ret = ret + typ
    ret = ret + data
    ret = ret + struct.pack('>l', zlib.crc32(typ + data))
    return ret
 
def fetch_one(cat):
    q = db.execute('SELECT Data from data WHERE cat=?', [cat])
    r = q.fetchone()
    return ''.join([str(x) for x in r[0]])
 
def fetch_idat(i):
    q = db.execute('SELECT Data from data WHERE ID=?', [i])
    r = q.fetchone()
    return ''.join([str(x) for x in r[0]])
 
png = open('flag.png', 'wb')
 
png.write('\x89\x50\x4E\x47\x0D\x0A\x1A\x0A')  # PNG magic
png.write(get_chunk('IHDR', fetch_one(4)))
png.write(get_chunk('PLTE', fetch_one(7)))
png.write(get_chunk('tRNS', fetch_one(8)))
 
dat = [4, 11, 2, 1, 14, 3, 9, 8, 5, 13, 10]
for d in dat:
    png.write(get_chunk('IDAT', fetch_idat(d)))
 
png.write(get_chunk('IEND', ''))
 
png.close()
~~~

This produces a 600x600 PNG file with the flag mirrored (because what kind of forensic challenge would it be if there was no image mirroring?).

Flag: SharifCTF{A3FBFC944D5CA155B0C04C97823986B6}
