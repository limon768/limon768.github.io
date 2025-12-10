---
cover: https://photos.squarezero.dev/file/abir-images/CrackTheHash/logo.jpeg
date: 2022-08-02 09:45:47+07:00
modified: 2022-08-02 09:45:47+07:00
categories: [ Writeups, TryHackMe ]
machine_author:
  link: https://tryhackme.com/p/ben
  name: Ben
tags: [Hashcat, Brute Force, TryHackMe, Writeup]
title: Crack The Hash - TryHackMe
---

![](https://photos.squarezero.dev/file/abir-images/htbasset/thmbanner.png)



Cracking hashes challenges

### Description → 

## Level 1

***Q1. 48bb6e862e54f2a795ffc4e541caed4d***

***A. easy***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/1.png)

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/2.png)

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/3.png)

`hashcat -m 100 Q1.txt /usr/share/wordlists/rockyou.txt`

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/4.png)

***Q2. CBFDAC6008F9CAB4083784CBD1874F76618D2A97***

***A. password123***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/5.png)

`hashcat -m 100 Q2.txt /usr/share/wordlists/rockyou.txt`

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/6.png)

***Q3. 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032***

***A. letmein***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/7.png)

`hashcat -m 1400 Q3.txt /usr/share/wordlists/rockyou.txt`

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/8.png)

***Q4. \$2y\$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom***

***A. bleh***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/9.png)

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/10.png)

`hashcat -m 3200 Q4.txt /usr/share/wordlists/rockyou.txt`

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/11.png)

***Q5. 279412f945939ba78ce0758d3fd83daa***

***A. Eternity22***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/12.png)

***NOTE: Hashcat didnt worked so I used STH***

`sth -t '279412f945939ba78ce0758d3fd83daa’`

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/13.png)

## LEVEL 2

***Q6. F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85***

***A. paule***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/14.png)

`hashcat -m 1400 Q6.txt /usr/share/wordlists/rockyou.txt`

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/15.png)

***Q7. 1DFECA0C002AE40B8619ECF94819CC1B***

***A. n63umy8lkf4i***

`hashcat -m 1000 Q7.txt /usr/share/wordlists/rockyou.txt`

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/16.png)

***NOTE: Both Hash-identifier and Hash-Analyzer both failed to identify the hash type. SO I checked hint and its NTML***

***Q8. Hash:\$6\$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.***

***Salt: aReallyHardSalt***

***A. waka99***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/17.png)

***hashcat -m 1800 Q8.txt /usr/share/wordlists/rockyou.txt***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/18.png)

***Q9. Hash: e5d8870e5bdd26602cab8dbe07a942c8669e56d6***

***Salt: tryhackme***

***A. 481616481616***

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/19.png)

`hashcat -m 160 Q9.txt /usr/share/wordlists/rockyou.txt`

![](https://photos.squarezero.dev/file/abir-images/CrackTheHash/20.png)