---
title: MemLab 2
date: 2022-10-20 12:45:47 +07:00
modified: 2022-10-20 12:45:47 +07:00
tags: [Volatility, forensics, MemLabs , Writeup]
description: Write Up for MemLab 2
iamge: https://photos.squarezero.dev/file/abir-images/MemLab2/logo.png
---
![](https://photos.squarezero.dev/file/abir-images/MemLab2/logo.png)

# Challenge Description

One of the clients of our company, lost the access to his system due to an unknown error. He is supposedly a very popular "environmental" activist. As a part of the investigation, he told us 
that his go to applications are browsers, his password managers etc. We hope that you can dig into this memory dump and find his important stuff and give it back to us.

**Note**: This challenge is composed of 3 flags.

# Write Up

I have to find the profile first. 

`vol.py -f MemoryDump_Lab2.raw imageinfo`

![](https://photos.squarezero.dev/file/abir-images/MemLab2/1.png)

Let’s see what process it was running. 

`vol.py -f MemoryDump_Lab2.raw --profile=Win7SP1x64 pslist`

I found Chrome, KeePass, and Notepad.

![](https://photos.squarezero.dev/file/abir-images/MemLab2/2.png)

## FLAG 1

As the challenge highlighted the word “environmental” , let’s take a look at the environmental variables. 

`vol.py -f MemoryDump_Lab2.raw --profile=Win7SP1x64 envars`

![](https://photos.squarezero.dev/file/abir-images/MemLab2/3.png)

We found this New_TMP variable in every process and it looks like base64. After decoding we found our first flag.

**FLAG 1 → flag{w3lc0m3_T0_$T4g3_!_Of_L4B_2}**

## FLAG 2

Lets look at the KeePass. KeePass extension is `.kdbx` and we need to find the password to open the file. Lets look for them.

`vol.py -f MemoryDump_Lab2.raw --profile=Win7SP1x64 filescan | grep .kdbx -i`

`vol.py -f MemoryDump_Lab2.raw --profile=Win7SP1x64 filescan | grep password -i`

![](https://photos.squarezero.dev/file/abir-images/MemLab2/9.png)

![](https://photos.squarezero.dev/file/abir-images/MemLab2/10.png)

Lets dump the [Password.png]() and open [Hidden.kdbx]() with the password.

`vol.py -f MemoryDump_Lab2.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fb112a0 -D .`

`vol.py -f MemoryDump_Lab2.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fce1c70 -D .`

![](https://photos.squarezero.dev/file/abir-images/MemLab2/11.png)
![](https://photos.squarezero.dev/file/abir-images/MemLab2/12.png)

**FLAG 2 → flag{w0w_th1s_1s_Th3_SeC0nD_ST4g3_!!}**

## FLAG 3

Next as per the challenge, we check the browser. Let’s look for interesting files and grep chrome.

`vol.py -f MemoryDump_Lab2.raw --profile=Win7SP1x64 filescan | grep chrome -i`

After looking for a little bit I found history. Lets dump the file and look at the content.

![](https://photos.squarezero.dev/file/abir-images/MemLab2/4.png)

`vol.py -f MemoryDump_Lab2.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fcfb1d0 -D .`

![](https://photos.squarezero.dev/file/abir-images/MemLab2/5.png)

We need to install the SQLite browser in order to open the file. After checking the history I found a [[Mega.nz](http://Mega.nz)]() link. 

`snap install sqlitebrowser`

![](https://photos.squarezero.dev/file/abir-images/MemLab2/6.png)

After visiting the website I found a zip file. The file is password protected but it gives us a note.

![](https://photos.squarezero.dev/file/abir-images/MemLab2/7.png)

Password → `6045dd90029719a039fd2d2ebcca718439dd100a`

And we found our final flag.

![](https://photos.squarezero.dev/file/abir-images/MemLab2/8.png)

**FLAG 3 → flag{oK_So_Now_St4g3_3_is_DoNE!!}**