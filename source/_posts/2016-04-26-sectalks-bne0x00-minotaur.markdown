---
layout: post
title: "SecTalks: BNE0x00 - Minotaur"
date: 2016-04-26 14:27:14 -0400
comments: true
categories: [vulhub]
---


From [Vulhub](https://www.vulnhub.com/entry/sectalks-bne0x00-minotaur,139/)

###Forces:

* netdiscover
* Nmap
* Burp Suite
* Metasploit

<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.56.0/24`

{% img  /images/blog/vulhub/bne03/Selection_001.png   [title manually exploit [alt text]] %}

192.168.56.223 is the target.


Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.56.223 -p-`

{% img  /images/blog/vulhub/bne03/Selection_002.png   [title manually exploit [alt text]] %}

port 22, 80 and 2020 is opening.

use wfuzz to find more locations

`wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.56.223/FUZZ 2>/dev/null`


{% img  /images/blog/vulhub/bne03/Selection_003.png   [title manually exploit [alt text]] %}

found http://192.168.56.223/bull/

Checked the page, it looks like use wordpress. good. maybe I can find out some old wordpress plugins.

{% img  /images/blog/vulhub/bne03/Selection_004.png   [title manually exploit [alt text]] %}




