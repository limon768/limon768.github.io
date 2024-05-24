---
title: MemLab 1
date: 2022-10-20 09:45:47 +07:00
modified: 2022-10-20 09:45:47 +07:00
tags: [Volatility, forensics, MemLabs , Writeup]
description: Write Up for MemLab 1
image: https://photos.squarezero.dev/file/abir-images/MemLab1/logo.png
---
![](https://photos.squarezero.dev/file/abir-images/MemLab1/logo.png)
# Challenge Description

My sister's computer crashed. We were very fortunate to recover this memory dump. Your job is get all her important files from the system. From what we remember, we suddenly saw a black window pop up with some thing being executed. When the crash happened, she was trying to draw something. Thats all we remember from the time of crash.

Note: This challenge is composed of 3 flags.

# Write Up

From the challenge description, we know there are a total of 3 flags. 

I have to find the profile first. 

`vol.py -f MemoryDump_Lab1.raw imageinfo`

![](https://photos.squarezero.dev/file/abir-images/MemLab1/1.png)

I found the profile! `Win7SP1x64`

Let’s see what process it was running. 

`vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 pslist`

![](https://photos.squarezero.dev/file/abir-images/MemLab1/2.png) 

From the process, 3 things caught my eye. CMD, WINRAR, and MSPaint.

## FLAG 1

Let’s check what the console was running. 

`vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 consoles`

![](https://photos.squarezero.dev/file/abir-images/MemLab1/3.png)

I found a base64 string in the output and after decoding I found the first flag.

**Flag1 → flag{th1s_1s_th3_1st_st4g3!!}**

## FLAG 2

Finding this flag took me some time. 

Let’s dump mspaint.exe.

`vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 memdump -p 2424 -D .`

![](https://photos.squarezero.dev/file/abir-images/MemLab1/4.png)

I know that this is an image file. So let’s try to open the file as raw file in [Gimp]().

![](https://photos.squarezero.dev/file/abir-images/MemLab1/5.png)

After doing some adjustments I find the flag.

![](https://photos.squarezero.dev/file/abir-images/MemLab1/6.png)

TIP: You want to flip the picture before adjusting the file.

**Flag2: flag{G00d_BoY_good_girL}**

## FLAG 3

Let’s find the final flag. In the challenge it was talking about important files. 

Lets scan for `.rar` as we saw winrar was running.

`vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 filescan | grep ".rar"`

![](https://photos.squarezero.dev/file/abir-images/MemLab1/7.png)

I found [Important.rar]() . Lets dump it and check the files.

`vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000003fb48bc0 -D .`

But after trying to unzip the file I see its password protected. Now I have the find NTLM hash.

![](https://photos.squarezero.dev/file/abir-images/MemLab1/8.png)

Lets dump the hash. 

`vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 hashdump`

![](https://photos.squarezero.dev/file/abir-images/MemLab1/9.png)

![](https://photos.squarezero.dev/file/abir-images/MemLab1/10.png)

`F4FF64C8BAAC57D22F22EDC681055BA6`

And finally I got the final flag using the hash as password.

**FLAG 3:  flag{w3IL_3rd_stage_was_easy}**