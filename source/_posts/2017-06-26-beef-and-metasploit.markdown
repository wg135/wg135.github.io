---
layout: post
title: "beEF and metasploit"
date: 2017-06-26 16:09:40 -0500
comments: true
categories: beef metasploit
---

Today I will take a note about how to use beef with metasploit

###install beef

Although kali includes beef, it still have some issues when I use.

to include latest beef:

`git clone https://github.com/beefproject/beef.git`

`cd beef`
`bundle install`

<!--more-->

in beef/, change configure file config.yaml

{% img  /images/blog/misc/beef/Selection_001.png   [title manually exploit [alt text]] %}

change host and callback host value as well ssl as true in beef/extensions/metasploit/config.yaml

{% img  /images/blog/misc/beef/Selection_002.png   [title manually exploit [alt text]] %}

now start metasploit:

{% img  /images/blog/misc/beef/Selection_003.png   [title manually exploit [alt text]] %}

start beef:

{% img  /images/blog/misc/beef/Selection_004.png   [title manually exploit [alt text]] %}

in http://127.0.0.1:3000/ui/authentication

{% img  /images/blog/misc/beef/Selection_005.png   [title manually exploit [alt text]] %}

login beef

now create index.html:

{% img  /images/blog/misc/beef/Selection_006.png   [title manually exploit [alt text]] %}

after that, in metasploit, I use MS12-063

{% img  /images/blog/misc/beef/Selection_007.png   [title manually exploit [alt text]] %}

use IE7 to access malicious index.html. get IE info from beef. 

{% img  /images/blog/misc/beef/Selection_008.png   [title manually exploit [alt text]] %}

Search command redirect and feed its url which is generated in msf

{% img  /images/blog/misc/beef/Selection_009.png   [title manually exploit [alt text]] %}


get meterpreter:

{% img  /images/blog/misc/beef/Selection_010.png   [title manually exploit [alt text]] %}