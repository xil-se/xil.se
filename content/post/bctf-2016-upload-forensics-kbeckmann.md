+++
authors = "kbeckmann"
categories = [ "writeups" ]
date = "2016-03-21T06:41:21+01:00"
description = ""
title = "BCTF 2016 Upload (Forensics 200) Writeup"

+++


# Problem
>Where are the files I just uploaded?
>
>The files in the links below are the same, download any of them to begin hacking!
>
>disk.img.xz - google drive link
>
>disk.img.xz - dropbox link
>
>disk.img.xz - baidu link

# Solution

We get a packed binary blob. Unpack it using `xz` and use `file` to see what it is.

    $ xz -d disk.img.xz

    $ file disk.img
    disk.img: BTRFS Filesystem sectorsize 4096, nodesize 16384, leafsize 16384, UUID=89011762- aee-4847-9e0f-bca52fd99e0d, 155820032/2147483648 bytes used, 1 devices

Oh a BTRFS image. Nice.

Since this is a forensics challenge I didn't even bother mounting it, it probably has some hidden partition or something.

Let's use `btrfs restore`:

```
$ mkdir restore
$ btrfs restore disk.img restore
Skipping snapshot dc34ada559d579c63d7c50c8348a854e9d9873850987e8d08b6493a2d1921b0a
Skipping snapshot 9e7eba4abf699021c04b4a4ce42972c297b47b7ef2b41bdbbc4f08955c21c3e1
Skipping snapshot b111de287cf2b2daa8392b0d4c7f76a84a5e7cd3687bdd118960f44d8410e9ca
Skipping snapshot ed268c5e6895b2cfa9518b6b2df98a0972207b3e569d758dc07d7e539478962c
Skipping snapshot 3b0bf85b8b3403bcc72f6ad2fdfdc5685e4aa8290b996c54f76435ef14ae19f5
Skipping snapshot 15286535e2415659c01c4e5d28e7b7572bd7c7e1d73b2931880b5e0b291b730b
```

Ok looks interesting. But these snapshots sound interesting, let's dump them!

    $ mkdir restore2
    $ btrfs restore -si disk.img restore2

This takes a while. Now we have the snapshots:

```
$ ls restore2/btrfs/subvolumes/
15286535e2415659c01c4e5d28e7b7572bd7c7e1d73b2931880b5e0b291b730b/
3b0bf85b8b3403bcc72f6ad2fdfdc5685e4aa8290b996c54f76435ef14ae19f5/
3b8877cfaaff2e44fbe24b43c3ea2e2d92b05831c5ce01b8a4928f6ced5d2c42/
9e7eba4abf699021c04b4a4ce42972c297b47b7ef2b41bdbbc4f08955c21c3e1/
b111de287cf2b2daa8392b0d4c7f76a84a5e7cd3687bdd118960f44d8410e9ca/
dc34ada559d579c63d7c50c8348a854e9d9873850987e8d08b6493a2d1921b0a/
ed268c5e6895b2cfa9518b6b2df98a0972207b3e569d758dc07d7e539478962c/
```

Let's diff them and see where files are added and removed. I used find and diff, ended up finding these files:

    > ./data
    > ./SimpleHTTPServerWithUpload.py

`SimpleHTTPServerWithUpload.py` looks really interesting. Turns out it's some kind of file upload server, and the code contains a URL "http://li2z.cn/", so I thought I was supposed to hack that site. Anyway, this was not the case.

After searching for more interesting files and almost giving up, I found another btrfs command.

```
$ btrfs restore -l disk.img
tree key (EXTENT_TREE ROOT_ITEM 0) 54853632 level 1
tree key (DEV_TREE ROOT_ITEM 0) 65847296 level 0
tree key (FS_TREE ROOT_ITEM 0) 54804480 level 1
tree key (CSUM_TREE ROOT_ITEM 0) 54935552 level 1
tree key (UUID_TREE ROOT_ITEM 0) 30048256 level 0
tree key (257 ROOT_ITEM 0) 39223296 level 2
tree key (259 ROOT_ITEM 8) 46202880 level 2
tree key (260 ROOT_ITEM 9) 47054848 level 2
tree key (261 ROOT_ITEM 10) 62816256 level 2
tree key (264 ROOT_ITEM 13) 95059968 level 2
tree key (267 ROOT_ITEM 17) 115474432 level 2
tree key (270 ROOT_ITEM 20) 76054528 level 2
tree key (277 ROOT_ITEM 27) 53788672 level 2
tree key (278 ROOT_ITEM 28) 80281600 level 2
tree key (DATA_RELOC_TREE ROOT_ITEM 0) 29442048 level 0
```

Oh look, more information.

It is possible to extract the trees using `btrfs restore -r <id>`:
```
$ btrfs restore -r 278 disk.img restore3
```

And now, let's see what we have.
```
$ ls restore3/data
cat.jpg  fLaG  flags-clip-art-red-flag.pdf  not_flag.txt
$ ./restore3/data/fLaG
BCTF{it_s_no7_hard_to_r0cov3r_is_n_t_17}
```

And there it was, some fake flags and the actual flag.
