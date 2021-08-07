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
```bash
id
```
```bash
uname -a
```
```bash
lsb_release -a
```
