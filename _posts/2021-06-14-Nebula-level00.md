---
layout: default
title: "Nebula00"
tags: Nebula
description: "Finding the binary with SUID bit set in the Nebula form exploit.education"
---

## Intro

Hello world!  
After a long break from posting anything I'm pleased to start the new series - `Exploit Development`.  
I decided to start with the `Nebula` challenge from [exploit.education](https://exploit.education/nebula/).  
The idea is to walk through and document my journey in the order `Nebula` -> `Phoenix` -> `Fusian`.  

## Getting started

To get started you need to download a `Nebula` .iso file, you can find it [here](https://exploit.education/downloads/).  
I'm currently running the live version of that VM. To login, you can use `nebula`/`nebula` as credentials.  
To access the first level, log in as `level00` with the password of `level00`. For the sake of usability I `ssh` to the machine as that user.
The objective of the first level is to find the binary with the `SUID` bit set. 

## Finding the flag

To find that binary we will use the `find` command.  
```
level00@nebula:/home/level00$ cd ~
level00@nebula:~$ find / -u=s 2>/dev/null | grep flag*
```

`/` will start the search in the root directory  
`-perm` will look for specific permissions for the file  
`-u=s` will find only files with SUID bit set  
`2>/dev/null` redirecting all the errors to `/dev/null` to have a readable output  
`| grep flag*` to display only results that start with the `flag` in the name  

our `find` command will find a binary located in `/bin/.../level00`, let's run it:

```
level00@nebula:~$ find / -perm -u=s 2>/dev/null | grep flag*
/bin/.../flag00
/rofs/bin/.../flag00

level00@nebula:~$ /bin/.../flag00 
Congrats, now run getflag to get your flag!

flag00@nebula:~$ getflag 
You have successfully executed getflag on a target account

flag00@nebula:~$ id
uid=999(flag00) gid=1001(level00) groups=999(flag00),1001(level00)
```

## Other Nebula levels 
- [level00](https://hackish.space/Nebula-level00)
- [level01](https://hackish.space/Nebula-level01)
- [level02](https://hackish.space/Nebula-level02)
- [level03](https://hackish.space/Nebula-level03)
- [level04](https://hackish.space/Nebula-level04)
- [level05](https://hackish.space/Nebula-level05)
- [level06](https://hackish.space/Nebula-level06)
- [level07](https://hackish.space/Nebula-level07)
- [level08](https://hackish.space/Nebula-level08)
- [level09](https://hackish.space/Nebula-level09)
- [level10](https://hackish.space/Nebula-level10)
- [level11](https://hackish.space/Nebula-level11)
- [level12](https://hackish.space/Nebula-level12)
- [level13](https://hackish.space/Nebula-level13)
- [level14](https://hackish.space/Nebula-level14)
- [level15](https://hackish.space/Nebula-level15)
- [level16](https://hackish.space/Nebula-level16)
- [level17](https://hackish.space/Nebula-level17)
- [level18](https://hackish.space/Nebula-level18)
- [level19](https://hackish.space/Nebula-level19)