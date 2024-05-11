##### Objectives
- What is Powershell
- Basic Powershell commands
- Windows enumeration skills
- Powershell scripting

##### What is Powershell?
Powershell is the Windows Scripting Language and shell environment built using the .NET framework. Most Powershell commands, called cmdlets, are written in .NET. Unlike other scripting languages and shell environments, the output of these cmdlets are objects - making Powershell somewhat object-oriented.
The normal format of a cmdlet is represented using Verb-Noun; for example, the cmdlet to list commands is called Get-Command

Common verbs to use include:
Get
Start
Stop 
Read
Write
New
Out

``Question 1``: What is the command to get a new object?
```r
Get-New
```

##### Basic Power Commands
For beginners learning Powershell, ``Get-Command`` and ``Get-Help`` are the go-to commands to remember.

```powershell
PS C:\Users\Administrator> Get-Help Get-Command -Examples

NAME
    Get-Command

SYNOPSIS
    Gets all commands.


    Example 1: Get cmdlets, functions, and aliases

    PS C:\>Get-Command

    This command gets the Windows PowerShell cmdlets, functions, and aliases that are installed on the computer.
    Example 2: Get commands in the current session

    PS C:\>Get-Command -ListImported

    This command uses the ListImported parameter to get only the commands in the current session.
    Example 3: Get cmdlets and display them in order

    PS C:\>Get-Command -Type Cmdlet | Sort-Object -Property Noun | Format-Table -GroupBy Noun

    This command gets all of the cmdlets, sorts them alphabetically by the noun in the cmdlet name, and then displays
    them in noun-based groups. This display can help you find the cmdlets for a task.
    Example 4: Get commands in a module

    PS C:\>Get-Command -Module Microsoft.PowerShell.Security, PSScheduledJob

    This command uses the Module parameter to get the commands in the Microsoft.PowerShell.Security and PSScheduledJob
    modules.
    Example 5: Get information about a cmdlet

    PS C:\>Get-Command Get-AppLockerPolicy

    This command gets information about the Get-AppLockerPolicy cmdlet. It also imports the AppLocker module, which
    adds all of the commands in the AppLocker module to the current session.
```

###### Using Get-Command

Get-Command gets all the cmdlets installed on the current Computer. The great thing about this cmdlet is that it allows for pattern matching like the following

``Get-Command Verb-* or Get-Command *-Noun``

Running Get-Command New-* to view all the cmdlets for the verb new displays the following: 

Using the Get-Command to list all cmdlets installed
```powershell
PS C:\Users\Administrator> Get-Command New-*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           New-AWSCredentials                                 3.3.563.1  AWSPowerShell
Alias           New-EC2FlowLogs                                    3.3.563.1  AWSPowerShell
Alias           New-EC2Hosts                                       3.3.563.1  AWSPowerShell
Alias           New-RSTags                                         3.3.563.1  AWSPowerShell
Alias           New-SGTapes                                        3.3.563.1  AWSPowerShell
Function        New-AutologgerConfig                               1.0.0.0    EventTracingManagement
Function        New-DAEntryPointTableItem                          1.0.0.0    DirectAccessClientComponents
Function        New-DscChecksum                                    1.1        PSDesiredStateConfiguration
Function        New-EapConfiguration                               2.0.0.0    VpnClient
Function        New-EtwTraceSession                                1.0.0.0    EventTracingManagement
Function        New-FileShare                                      2.0.0.0    Storage
Function        New-Fixture                                        3.4.0      Pester
Function        New-Guid                                           3.1.0.0    Microsoft.PowerShell.Utility
--cropped for brevity--
```

###### Object Manipulation
In the previous task, we saw how the output of every cmdlet is an object. If we want to manipulate the output, we need to figure out a few things:
- passing the output to other cmdlets
- using specific object cmdlets to extract information
The ``Pipeline(|)`` is used to pass output from one cmdlet to another. A major difference compared to other shells is that Powershell passes an object to the next cmdlet instead of passing text or string to the command after the pipe. Like every object in object-oriented frameworks, an object will contain methods and properties.

You can think of methods as functions that can be applied to output from the cmdlet, and you can think of properties as variables in the output from a cmdlet. To view these details, pass the output of a cmdlet to the ``Get-Member`` cmdlet:

``Verb-Noun | Get-Member ``

An example of running this to view the members for ``Get-Command`` is:

``Get-Command | Get-Member -MemberType Method``

```powershell
PS C:\Users\Administrator> Get-Command | Get-Member -MemberType Method


   TypeName: System.Management.Automation.AliasInfo

Name             MemberType Definition
----             ---------- ----------
Equals           Method     bool Equals(System.Object obj)
GetHashCode      Method     int GetHashCode()
GetType          Method     type GetType()
ResolveParameter Method     System.Management.Automation.ParameterMetadata ResolveParameter(string name)
ToString         Method     string ToString()


   TypeName: System.Management.Automation.FunctionInfo

Name             MemberType Definition
----             ---------- ----------
Equals           Method     bool Equals(System.Object obj)
GetHashCode      Method     int GetHashCode()
GetType          Method     type GetType()
ResolveParameter Method     System.Management.Automation.ParameterMetadata ResolveParameter(string name)
ToString         Method     string ToString()


   TypeName: System.Management.Automation.CmdletInfo

Name             MemberType Definition
----             ---------- ----------
Equals           Method     bool Equals(System.Object obj)
GetHashCode      Method     int GetHashCode()
GetType          Method     type GetType()
ResolveParameter Method     System.Management.Automation.ParameterMetadata ResolveParameter(string name)
ToString         Method     string ToString()
```

##### Creating Objects From Previous cmdlets
One way of manipulating objects is pulling out the properties from the output of a cmdlet and creating a new object. This is done using the ``Select-Object`` cmdlet. 

Here's an example of listing the directories and just selecting the mode and the name:
```powershell
PS C:\Users\Administrator> Get-ChildItem | Select-Object -Property Mode, Name

Mode   Name
----   ----
d-r--- Contacts
d-r--- Desktop
d-r--- Documents
d-r--- Downloads
d-r--- Favorites
d-r--- Links
d-r--- Music
d-r--- Pictures
d-r--- Saved Games
d-r--- Searches
d-r--- Videos
```

###### Filtering Objects
When retrieving output objects, you may want to select objects that match a very specific value. You can do this using the ``Where-Object`` to filter based on the value of properties. 

The general format for using this cmdlet is 

``Verb-Noun | Where-Object -Property PropertyName -operator Value``

``Verb-Noun | Where-Object {$_.PropertyName -operator Value}``

The second version uses the $_ operator to iterate through every object passed to the Where-Object cmdlet.

``Powershell is quite sensitive, so don't put quotes around the command!
``
Where ``-operator`` is a list of the following operators:

- ``-Contains``: if any item in the property value is an exact match for the specified value
- ``-EQ``: if the property value is the same as the specified value
- ``-GT``: if the property value is greater than the specified value
```powershell
PS C:\Users\Administrator> Get-Service | Where-Object -Property Status -eq Stopped

Status   Name               DisplayName
------   ----               -----------
Stopped  AJRouter           AllJoyn Router Service
Stopped  ALG                Application Layer Gateway Service
Stopped  AppIDSvc           Application Identity
Stopped  AppMgmt            Application Management
Stopped  AppReadiness       App Readiness
Stopped  AppVClient         Microsoft App-V Client
Stopped  AppXSvc            AppX Deployment Service (AppXSVC)
Stopped  AudioEndpointBu... Windows Audio Endpoint Builder
Stopped  Audiosrv           Windows Audio
Stopped  AxInstSV           ActiveX Installer (AxInstSV)
Stopped  BITS               Background Intelligent Transfer Ser...
Stopped  Browser            Computer Browser
Stopped  bthserv            Bluetooth Support Service
-- cropped for brevity--
```

###### Sort-Object
When a cmdlet outputs a lot of information, you may need to sort it to extract the information more efficiently. You do this by pipe-lining the output of a cmdlet to the Sort-Object cmdlet.

The format of the command would be:

``Verb-Noun | Sort-Object``

Here's an example of sorting the list of directories:
```powershell
PS C:\Users\Administrator> Get-ChildItem | Sort-Object


    Directory: C:\Users\Administrator


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        10/3/2019   5:11 PM                Contacts
d-r---        10/5/2019   2:38 PM                Desktop
d-r---        10/3/2019  10:55 PM                Documents
d-r---        10/3/2019  11:51 PM                Downloads
d-r---        10/3/2019   5:11 PM                Favorites
d-r---        10/3/2019   5:11 PM                Links
d-r---        10/3/2019   5:11 PM                Music
d-r---        10/3/2019   5:11 PM                Pictures
d-r---        10/3/2019   5:11 PM                Saved Games
d-r---        10/3/2019   5:11 PM                Searches
d-r---        10/3/2019   5:11 PM                Videos
```

``Question:`` What is the location of the file "interesting-file.txt"
```powershell
PS C:\Windows> Get-ChildItem -Path C:\ -Include interesting-file.* -File -Recurse -ErrorAction SilentlyContinue

    Directory: C:\Program Files



Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/3/2019  11:38 PM             23 interesting-file.txt.txt

```

``Question:`` Specify the contents of this file.
```r
PS C:\Windows> more 'C:\Program Files\interesting-file.txt.txt'
notsointerestingcontent

PS C:\Windows>
```

``Question:`` How many cmdlets are installed on the system(only cmdlets, not functions and aliases)?
```powershell
PS C:\Windows> Get-Command | Where-Object -Property CommandType -eq Cmdlet | Measure-Object


Count    : 6638
Average  :
Sum      :
Maximum  :
Minimum  :
Property :
```

``Question:`` Get the ``MD5`` hash of interesting-file.txt
```powershell
PS C:\Windows> Get-FileHash -Path 'C:\Program Files\interesting-file.txt.txt' -Algorithm MD5

Algorithm       Hash                                          Path
---------       ----                                          ---
MD5             49A586A2A9456226F8A1B4CEC6FAB329              C:\Program Files\interesting-file.txt.txt
```

``What is the command to get the current working directory?``
```r
PS C:\Windows> Get-Location

Path
----
C:\Windows
```

``Question:`` Does the path "C:\Users\Administrator\Documents\Passwords" Exist (Y/N)?
```powershell

PS C:\Windows> Get-Location C:\Users\Administrator\Documents\Passwords
Get-Location : A positional parameter cannot be found that accepts argument 'C:\Users\Administrator\Documents\Passwords'.
At line:1 char:1
+ Get-Location C:\Users\Administrator\Documents\Passwords
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (:) [Get-Location], ParameterBindingException
    + FullyQualifiedErrorId : PositionalParameterNotFound,Microsoft.PowerShell.Commands.GetLocationCommand
```

``Question:`` What command would you use to make a request to a web server?
```powershell
PS C:\Windows> Invoke-WebRequest

cmdlet Invoke-WebRequest at command pipeline position 1
Supply values for the following parameters:
Uri:
```

``Question:`` Base64 decode the file b64.txt on Windows.
- Get location of file
```powershell
PS C:\Windows> Get-ChildItem -Path C:\ -Include b64.* -File -Recurse -ErrorAction SilentlyContinue

    Directory: C:\Users\Administrator\Desktop

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/3/2019  11:56 PM            432 b64.txt
```

- Show content of file.
```powershell
PS C:\Windows> more C:\Users\Administrator\Desktop\b64.txt

dGhpcyBpcyB0aGUgZmxhZyAtIGlob3BleW91ZGlkdGhpc29ud2luZG93cwp0aGUgcmVzdCBpcyBnYXJiYWdlCnRoZSByZXN0IGlzIGdhcmJhZ2UKdGhlIHJlc3QgaXMgZ2FyYmFnZQp0aGUgcmVzdCB
pcyBnYXJiYWdlCnRoZSByZXN0IGlzIGdhcmJhZ2UKdGhlIHJlc3QgaXMgZ2FyYmFnZQp0aGUgcmVzdCBpcyBnYXJiYWdlCnRoZSByZXN0IGlzIGdhcmJhZ2UKdGhlIHJlc3QgaXMgZ2FyYmFnZQp0aG
UgcmVzdCBpcyBnYXJiYWdlCnRoZSByZXN0IGlzIGdhcmJhZ2UKdGhlIHJlc3QgaXMgZ2FyYmFnZQp0aGUgcmVzdCBpcyBnYXJiYWdlCnRoZSByZXN0IGlzIGdhcmJhZ2U=
```

- Decode Content of b64.txt to base64
```powershell
# Read the Base64 encoded content from the file
PS C:\Users\Administrator\Desktop> $base64Content = Get-Content .\b64.txt -Raw

# Decode the Base64 content
PS C:\Users\Administrator\Desktop> $decodedBytes = [System.Convert]::FromBase64String($base64Content)

# Convert the decoded bytes to string (if applicable)
PS C:\Users\Administrator\Desktop> $decodedText = [System.Text.Encoding]::UTF8.GetString($decodedBytes)

# Output the decoded text
PS C:\Users\Administrator\Desktop> $decodedText
this is the flag - ihopeyoudidthisonwindows
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
the rest is garbage
PS C:\Users\Administrator\Desktop>
```

## Enumeration
The first step when you have gained initial access to any machine would be to enumerate. We'll be enumerating the following:

- users
- basic networking information
- file permissions
- registry permissions
- scheduled and running tasks
- insecure files

``Question:`` How many users are there on the machine?
```powershell
PS C:\Users\Administrator\Desktop> Get-LocalUser

Name           Enabled Description
----           ------- -----------
Administrator  True    Built-in account for administering the computer/domain
DefaultAccount False   A user account managed by the system.
duck           True
duck2          True
Guest          False   Built-in account for guest access to the computer/domain
```

``Question:``  Which local user does this SID(S-1-5-21-1394777289-3961777894-1791813945-501) belong to?
```powershell
PS C:\Users\Administrator> Get-LocalUser -SID "S-1-5-21-1394777289-3961777894-1791813945-501"

Name  Enabled Description
----  ------- -----------
Guest False   Built-in account for guest access to the computer/domain
```

``Question:`` How many users have their password required values set to False?
```powershell

PS C:\Users\Administrator> Get-LocalUser | Select-Object -Property Name, PasswordRequired

Name           PasswordRequired
----           ----------------
Administrator              True
DefaultAccount            False
duck                      False
duck2                     False
Guest                     False
```

``Question:`` How many local groups exist?
```powershell
PS C:\Users\Administrator> Get-LocalGroup | Measure-Object


Count    : 24
Average  :
Sum      :
Maximum  :
Minimum  :
Property :
```

``Question:`` What command did you get the IP address info?
```powershell
PS C:\Users\Administrator> Get-NetIPAddress

IPAddress         : 10.10.168.119
InterfaceIndex    : 5
InterfaceAlias    : Ethernet
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 16
PrefixOrigin      : Dhcp
SuffixOrigin      : Dhcp
AddressState      : Preferred
ValidLifetime     : 00:59:19
PreferredLifetime : 00:59:19
SkipAsSource      : False
PolicyStore       : ActiveStore
```

``Question:`` How many ports are listed as listening?
```powershell
PS C:\Users\Administrator> Get-NetTCPConnection -State Listen | Measure-Object

Count    : 20
Average  :
Sum      :
Maximum  :
Minimum  :
Property :
```

``Question:`` What is the remote address of the local port listening on port 445?
```r

PS C:\Users\Administrator> Get-NetTCPConnection -State Listen

LocalAddress                        LocalPort RemoteAddress                       RemotePort State       
------------                        --------- -------------                       ---------- -----       
::                                  47001     ::                                  0          Listen
::                                  5985      ::                                  0          Listen
::                                  3389      ::                                  0          Listen
::                                  445       ::                                  0          Listen
::                                  135       ::                                  0          Listen
0.0.0.0                             49677     0.0.0.0                             0          Listen
0.0.0.0                             49674     0.0.0.0                             0          Listen
0.0.0.0                             49667     0.0.0.0                             0          Listen
0.0.0.0                             49666     0.0.0.0                             0          Listen
```

``Question:`` How many patches have been applied?
```r

PS C:\Users\Administrator> Get-HotFix | Measure-Object

Count    : 20
Average  :
Sum      :
Maximum  :
Minimum  :
Property :
```

``Question:`` When was the patch with ID KB4023834 installed?
```powershell
PS C:\Users\Administrator> Get-HotFix | Where-Object -Property HotFixID -eq KB4023834

Source        Description      HotFixID      InstalledBy          InstalledOn
------        -----------      --------      -----------          -----------
EC2AMAZ-5M... Update           KB4023834     EC2AMAZ-5M13VM2\A... 6/15/2017 12:00:00 AM
```

``Question:`` Find the contents of a backup file.
```powershell
PS C:\Users\Administrator> Get-ChildItem -Path C:\ -Include *.bak* -File -Recurse -ErrorAction SilentlyContinue

    Directory: C:\Program Files (x86)\Internet Explorer

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/4/2019  12:42 AM             12 passwords.bak.txt


PS C:\Users\Administrator> more 'C:\Program Files (x86)\Internet Explorer\passwords.bak.txt'
backpassflag

```

``Question:`` Search for all files containing API_KEY
```powershell
PS C:\Users\Administrator> Get-ChildItem C:\ -Recurse | Select-String -Pattern API_KEY

<< snip >>
H T T P         N O N E  'R E L A T I O N A L _ D A T A B A S E  A L L O W     D E N Y  A L L  E R R
J S O N  S D L  #R D S _ H T T P _ E N D P O I N T  P I P E L I N E    U N I T
C:\Users\Public\Music\config.xml:1:API_KEY=fakekey123
Select-String : The file C:\Windows\appcompat\Programs\Amcache.hve cannot be read: The process cannot a
'C:\Windows\appcompat\Programs\Amcache.hve' because it is being used by another process.
At line:1 char:30
+ Get-ChildItem C:\ -Recurse | Select-String -Pattern API_KEY
+                              ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (:) [Select-String], ArgumentException
    + FullyQualifiedErrorId : ProcessingFile,Microsoft.PowerShell.Commands.SelectStringCommand
```

``Question:`` What command do you do to list all the running proccesses?
```powershell
PS C:\Users\Administrator> Get-Process

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    120       8    20884      12752       0.20   1708   0 amazon-ssm-agent
    189      12     3756      14764       6.86   3276   2 conhost
    191      10     1776       3928       0.16    524   0 csrss
    118       8     1312       3600       0.05    588   1 csrss
    194      11     1740       4400       0.92   2736   2 csrss
    316      19    13232      29296       0.14    968   1 dwm
    351      27    14340      38032       0.89   2832   2 dwm
   1356      64    24000      78512       5.34   3064   2 explorer
      0       0        0          4                 0   0 Idle
     71       6      956       4672       0.00   1748   0 LiteAgent
```

``Question``: What is the path of the scheduled task called new-sched-task?
```powershell
PS C:\Users\Administrator> Get-ScheduledTask -TaskName new-sched-task

TaskPath                                       TaskName                          State
--------                                       --------                          -----
\                                              new-sched-task                    Ready

```

``Question:`` Who is the owner of the C:\
```powershell
PS C:\Users\Administrator> Get-Acl C:\

    Directory:

Path Owner                       Access
---- -----                       ------
C:\  NT SERVICE\TrustedInstaller CREATOR OWNER Allow  268435456...
```