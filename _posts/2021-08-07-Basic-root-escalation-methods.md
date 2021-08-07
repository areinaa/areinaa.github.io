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
A SUID binary file is an executable file which can be run by other users with the same privileges as the owner. This means that if this file's owner is root, we might be able to abuse those privileges to escalate permissions and run a root bash.
To list SUID files we will have to move to the root directory and run this command:
```bash
find \-perm -4000 2>/dev/null
```