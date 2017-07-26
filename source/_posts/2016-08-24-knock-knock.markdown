---
layout: post
title: "Knock-knock"
date: 2016-08-24 15:37:28 -0500
comments: true
categories: [vulhub,]
---

##Tools:

* netdiscover
* Nmap
* Wfuzz
* Nikto
* Strings

###Vulnerabilities:


<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.41.0/24`
{% img  /images/blog/vulhub/knock/Selection_001.png   [title manually exploit [alt text]] %}

192.168.41.166 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.41.166 -p-`

{% img  /images/blog/vulhub/pipe/Selection_002.png   [title manually exploit [alt text]] %}

only port 1337 is opening. Based on the nmap's output. I think this is port knocking.

use netcat to check:

`nc -nv 192.168.41.166 1337`

{% img  /images/blog/vulhub/pipe/Selection_003.png   [title manually exploit [alt text]] %}

get the list, looks like port number. I try to knock them, but failed. Then I realized that i should try all permutations, then I wrote script [port_knock_all.py](https://github.com/wg135/script/blob/master/port_knock_all.py). Run that, then rerun nmap

{% img  /images/blog/vulhub/pipe/Selection_004.png   [title manually exploit [alt text]] %}


use nikto 

`nikto -h 192.168.41.166`

use wfuzz

`wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.41.166/FUZZ 2>/dev/null`

Nothing cool shows.

check the page

{% img  /images/blog/vulhub/pipe/Selection_005.png   [title manually exploit [alt text]] %}

{% img  /images/blog/vulhub/pipe/Selection_006.png   [title manually exploit [alt text]] %}

nothing useful. Since it is only one image, I will download it and check the string in it

`strings knockknock.jpg`

{% img  /images/blog/vulhub/pipe/Selection_007.png   [title manually exploit [alt text]] %}

looks like we got `abfnW/sax2Cw9Ow`

try to use this login ssh, failed....

Figure out it is [Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher) and use [Caesar cipher decryption tool](http://www.xarg.org/tools/caesar-cipher/)

get `jason/jB9jP2knf`



got shell:

{% img  /images/blog/vulhub/pipe/Selection_008.png   [title manually exploit [alt text]] %}

now find starting at root (/), SGID or SUID, not Symbolic links, only 3 folders deep, list with more detail and hide any errors (e.g. permission denied)
`find / -perm -g=s -o -perm -6000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null`

got a file `/home/jason/tfc`

{% img  /images/blog/vulhub/pipe/Selection_009.png   [title manually exploit [alt text]] %}

run the file

{% img  /images/blog/vulhub/pipe/Selection_010.png   [title manually exploit [alt text]] %}

looks like it need a input file and output file.

tfc will encrypt input and also decrpt input if its encryped. Now generate a large input file.

{% img  /images/blog/vulhub/pipe/Selection_011.png   [title manually exploit [alt text]] %}

`python -c "print 'A'*5000" >in.tfc`

{% img  /images/blog/vulhub/pipe/Selection_012.png   [title manually exploit [alt text]] %}

get segmentation fault error.

First, I use [checksec.sh](https://github.com/wg135/checksec) to check if there is any protection

`./checksec.sh  --file tfc`
{% img  /images/blog/vulhub/pipe/Selection_013.png   [title manually exploit [alt text]] %}

No protection.

Since gdb is not available on the target, I download tfc to my kali

{% img  /images/blog/vulhub/pipe/Selection_013.png   [title manually exploit [alt text]] %}

the address is 0x0675c916  not 0x41414141. so it should be encryption of 0x41414141. I was able to figure out how many bytes to pass in to overwrite the return address (4124 bytes).

(To be continue...)




