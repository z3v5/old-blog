---
layout: default
title: "Nebula08"
tags: Nebula
description: "Analyzing .pcap file to find credentials in TCP stream"
---


## Level 08

> World readable files strike again. Check what that user was up to, and use it to log into flag08 account. To do this level, log in as the `level08` account with the password `level08`. Files for this level can be found in `/home/flag08`.

## Source code

There is no source code available for this level.

## Getting the flag

The `.pcap` file is in the `flag08` folder, let's download it to the main machine and analyze it with `Wireshark`. One of the ways could be spawning a `SimpleHTTPServer` on the machine:
```
level08@nebula:/home/flag08$ python -m SimpleHTTPServer 8080
Serving HTTP on 0.0.0.0 port 8080 ...
192.168.232.128 - - [17/Jun/2021 01:11:10] "GET / HTTP/1.1" 200 -
```
Open the `.pcap` file with `Wireshark` and follow the `TCP stream`:

```
..%..%..&..... ..#..'..$..&..... ..#..'..$.. .....#.....'........... .38400,38400....#.SodaCan:0....'..DISPLAY.SodaCan:0......xterm.........."........!........"..".....b........b....	B.
..............................1.......!.."......"......!..........."........"..".............	..
.....................
Linux 2.6.38-8-generic-pae (::ffff:10.1.1.2) (pts/10)

..wwwbugs login: l.le.ev.ve.el.l8.8
..
Password: backdoor...00Rm8.ate
.
..
Login incorrect
wwwbugs login: 
```
What a mess. Let's look at it not in the `ASCII`, but `HEX`:
```
000000B2  6c                                                 l
    000000C9  00 6c                                              .l
000000B3  65                                                 e
    000000CB  00 65                                              .e
000000B4  76                                                 v
    000000CD  00 76                                              .v
000000B5  65                                                 e
    000000CF  00 65                                              .e
000000B6  6c                                                 l
    000000D1  00 6c                                              .l
000000B7  38                                                 8
    000000D3  00 38                                              .8
000000B8  0d                                                 .
    000000D5  01                                                 .
    000000D6  00 0d 0a 50 61 73 73 77  6f 72 64 3a 20            ...Passw ord: 
000000B9  62                                                 b
000000BA  61                                                 a
000000BB  63                                                 c
000000BC  6b                                                 k
000000BD  64                                                 d
000000BE  6f                                                 o
000000BF  6f                                                 o
000000C0  72                                                 r
000000C1  7f                                                 .
000000C2  7f                                                 .
000000C3  7f                                                 .
000000C4  30                                                 0
000000C5  30                                                 0
000000C6  52                                                 R
000000C7  6d                                                 m
000000C8  38                                                 8
000000C9  7f                                                 .
000000CA  61                                                 a
000000CB  74                                                 t
000000CC  65                                                 e
```

Huh, it [seems](https://www.asciihex.com/character/control/127/0x7F/del-delete-character) those `7f`'s are the `delete` charracters. 

In that case, the password will be the `backd00Rmate`. 

Let's try to `su` to the user `flag08` with it:
```
level08@nebula:/home/flag08$ su flag08
Password: 
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
