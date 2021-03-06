---
title: "Basic root priviledge escalation methods"
categories:
  - Beginner
tags:
  - Linux
  - SUID
  - Capabilities
  - cron
---
This post enumerates a series of methods which can be used once you have already gained access to a system user. Going through SUID listing, PATH hijacking and cron & capabilities abuse.

### Introduction: CVE and SUID

Once you have already broken through a machine and gained access to a standard user's console in linux, there are several simple ways of achieving root privileges before any deep reconnaissance.
First of all we will want to start by listing the groups our user belongs to, and the system and kernel version. With this information we get to know if we belong to any special or privileged group and we can also look for CVEs if the system is running an old version.

UID and groups
```bash
id
```
Kernel
```bash
uname -a
```
System OS
```bash
lsb_release -a
```
<p align="center">
<img src="/assets/images/examplekernel.png">
</p>

A SUID binary file is an executable file which can be run by other users with the same privileges as the owner. This means that if this file's owner is root, we might be able to abuse those privileges to escalate permissions and run a root bash.
To list SUID files we will have to move to the root directory and run this command:
```bash
find \-perm -4000 2>/dev/null
```
Once listed, we will research for binaries with potential for exploitation. There are lots of compilations of known exploits all throughout the internet; one of them is https://gtfobins.github.io/, an extensive database of different types of exploits.

In this case, there are two specially interesting SUID binaries: find and php.

<p align="center">
<img src="/assets/images/findexample.jpg">
</p>
Looking at https://gtfobins.github.io/gtfobins/find/#suid and https://gtfobins.github.io/gtfobins/php/#suid respectively, we can use oneliners to exploit these binaries and gain instant access to root privileges.

<p align="center">
<img src="/assets/images/rootphp.png">
</p>
<p align="center">
<img src="/assets/images/rootfind.png">
</p>

### Abusing cron tasks

One of the things we can do to escalate privileges once we gain access to the machine is trying to abuse cron tasks.
Cron tasks are binaries running in the background at a set interval of time; if these binaries permissions are not set properly, they can be abused by an attacker.

To look for these tasks, we are going to set up a simple script to monitor commands being run at real time by using `ps -eo command`.

```bash
# !/bin/bash

pold=$(ps -eo command)

while true; do
  pnew=$(ps -eo command)
  diff <(echo "$pold") <(echo "$pnew") | grep "[\>\<]" | grep -v "kworker"
  pold=$pnew
done
```
First this script will scan all commands being run in the system and store them in `pold`. Next, inside an infinite while loop we will continiously store all commands being run inside another variable called `pnew`, we will then compare it with `pold` so we can get the difference in both variables(new commands have been run). We filter the result with a `grep` so only the new commands are being reported to us and update the `pold` variable at the end of the loop. Rinse and repeat until you find any suspicious activity.

<p align="center">
<img src="/assets/images/cronexample.png">
</p>
<p align="center">
<img src="/assets/images/cronfile.png">
</p>

In this example we can see that there is a binary being run periodically at `/home/areina/Desktop/file.sh` whose owner is root and we have writting permission. The rest is as easy as modifying the content to get root privileges. In this case I will go for SUID exploitation.

<p align="center">
<img src="/assets/images/cronroot.png">
</p>


### PATH Hijacking
Binaries can run system commands both with absolute routes `/usr/bin/cat` or relative routes `cat`. When you just type `cat` the system will look for its absolute route through an environment variable called PATH. If a binary with root privileges has a system command with a relative route, an attacker may modify their PATH variable so that the command the binary executes is a malicious one made by them.

<p align="center">
<img src="/assets/images/pathexample.png">
</p>

Example
```C
#include <stdio.h>

void main(){
setuid(0); //set root privileges

printf("\nReading hosts:\n\n");
system("/usr/bin/cat /etc/hosts");

printf("\n\n\nReading hosts:\n\n");
system("cat /etc/hosts");
}
```
This binary will list twice the content of the hosts file with cat using root privileges. The only difference is that one of them uses absolute path and the other depends on the PATH.

<p align="center">
<img src="/assets/images/hostspath1.png">
</p>

Although if this binary were to be compiled we wouldn't know this information, by using the `strings` utility we can try reading the file and discern its content.

<p align="center">
<img src="/assets/images/stringshosts.png">
</p>

Knowing the binary is using `cat` as a relative route we can create a malicious file called `cat` in our home directory with `bash -p` as its content. Once created we can modify our PATH variable so it reads our current directory first with `export PATH=.:$PATH`.
Finally, if we run again the same binary it will read our malicious file in the second `cat` and give us a root bash.

<p align="center">
<img src="/assets/images/rootbash.png">
</p>

### Capabilities

Another possible way of rooting a machine or establishing persistency is through the use of capabilities in binaries.

Scanning
```bash
getcap -r / 2>/dev/null
```
Setting up capabilities in root binaries
```bash
setcap cap_setuid+ep /path/to/file
```
