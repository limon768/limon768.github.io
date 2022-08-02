---
title: CTF-DumpMe
date: 2022-07-16 11:45:47 +07:00
modified: 2022-07-16 11:45:47 +07:00
tags: [CTF, forensics , Cyber Defender, Writeup, Volatility]
description: Write Up for DumpMe Cyber Defense
---

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/logo.jpg)

# DumpMe

Box Name → DumpMe

Box Link → [https://cyberdefenders.org/blueteam-ctf-challenges/65](https://cyberdefenders.org/blueteam-ctf-challenges/65)

Author → Champlain College

Description → One of the SOC analysts took a memory dump from a machine infected with a meterpreter malware. As a Digital Forensicators, your job is to analyze the dump, extract the available indicators of compromise (IOCs) and answer the provided questions.

## Prerequisites

Download the zip file. Unzip the file with password →  **"cyberdefenders.org"**

***NOTE: For this room I’m going to use Volatility 2.6. I have another writeup for Volatility if you want to check that out.***

## Challenge Questions

***Q1.  What is the SHA1 hash of Triage-Memory.mem (memory dump)?***

***A. c95e8cc8c946f95a109ea8e47a6800de10a27abd***

`shasum Triage-Memory.mem`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/01.png)

***Q2. What volatility profile is the most appropriate for this machine? (ex: Win10x86_14393)***

***A. Win7SP1x64***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" imageinfo`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/02.png)

***Q3. What was the process ID of notepad.exe?***

***A. 3032***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 pslist | grep notepad.exe`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/03.png)

***Q4. Name the child process of wscript.exe***

***A. 3032***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 pstree`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/04.png)

***Q5. What was the IP address of the machine at the time the RAM dump was created?***

***A. 10.0.0.101***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 netscan`

***Q6. Based on the answer regarding the infected PID, can you determine the IP of the attacker?***

***A. 10.0.0.106***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 netscan`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/05.png)


***Q7.  How many processes are associated with VCRUNTIME140.dll?***

***A. 5***

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/06.png)


***NOTE: For some reason its not showing all the Process. I did some research but couldn’t find a solution. I will update the write up if i find a solution.***

***Q8. After dumping the infected process, what is its md5 hash?***

***A. 690ea20bc3bdfb328e23005d9a80c290***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 procdump -p 3496 --dump-dir "/home/avi/Downloads/vol/”`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/7.png)
 

***Q9. What is the LM hash of Bob's account?***

***A. aad3b435b51404eeaad3b435b51404ee***

`/volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 hashdump`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/8.png)


***Q10. What memory protection constants does the VAD node at 0xfffffa800577ba10 have?***

***A. PAGE_READONLY***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 vadinfo | grep 0xfffffa800577ba10 -A 6`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/9.png)


***Q11. What memory protection did the VAD starting at 0x00000000033c0000 and ending at 0x00000000033dffff have?***

***A. PAGE_NOACCESS***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 vadinfo | grep 0x00000000033c0000 -A 6`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/10.png)


***Q12. There was a VBS script that ran on the machine. What is the name of the script? (submit without file extension)***

***A. vhjReUDEuumrX***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 cmdline`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/11.png)


***Q13. An application was run at 2019-03-07 23:06:58 UTC. What is the name of the program? (Include extension)***

***A. Skype.exe***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 shimcache | grep "2019-03-07 23:06:58"`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/12.png)


***Q14. What was written in notepad.exe at the time when the memory dump was captured?***

***A. flag<REDBULL_IS_LIFE>***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 memdump -p 3032 --dump-dir "/home/avi/Downloads/vol/”`

`strings -e l "/home/avi/Downloads/vol/3032.dmp" | grep 'flag<’`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/13.png)


![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/14.png)


***Q15. What is the short name of the file at file record 59045?***

***A. EMPLOY~1.XLS***

`./volatility -f "/home/avi/Downloads/vol/Triage-Memory.mem" --profile=Win7SP1x64 mftparser | grep "Record Number: 59045" -A 20`

![](https://photos.squarezero.dev/file/abir-images/CTF-DumpMe/15.png)


***Q16. This box was exploited and is running meterpreter. What was the infected PID?***

***A. 3496***