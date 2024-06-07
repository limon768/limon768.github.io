---
title: Volatility
date: 2022-07-11 11:45:47 +07:00
modified: 2022-07-11 11:45:47 +07:00
tags: [Volatility, forensics , TryHackMe, Writeup]
description: Learn how to perform memory forensics with Volatility!
image: https://photos.squarezero.dev/file/abir-images/Volatility/logo.png
---

![](https://photos.squarezero.dev/file/abir-images/htbasset/thmbanner.png)

<img src="https://photos.squarezero.dev/file/abir-images/Volatility/logo.png" style="margin-left: 20px; zoom: 100%;" align=left />    	<font size="10">Volatility</font>

8<sup>th</sup> November 2022

​		Machine Author(s): [TryHackMe](https://tryhackme.com/p/tryhackme)

### Description: 
Learn how to perform memory forensics with Volatility!

## Intro

Volatility is a free memory forensics tool for incident response developed and maintain by Volatility

**Install Volatility** 

Download executable [Volatility]([https://www.volatilityfoundation.org/releases](https://www.volatilityfoundation.org/releases))

```bash
sudo mv volatility_2.6 /opt/
cd /opt/volatility_2.6/
./volatility -h
```

***P.S: I have rename the executable for efficiency*** 

## Obtaining Memory Samples

To analyze we need a memory sample first. Below are some tools that can help to gain a memory sample easily

- FTK Imager
- Redline
- Dumplt.exe
- win32dd.exe / win64dd.exe

One of the most common memory file types are .raw which contains system memory.  Offline machines can have their memory pulled as long as they aren’t encrypted.

Microsoft Windows uses hiberfil.sys to provide faster boot-up time which we can use for some memory forensics

To get hiberfil.sys 

1. Go to system drive [C:\] 
2. Go to view → select checkbox ***Hidden items***
3. Then click options and uncheck ***Hide protected operating system files***
4. click Apply → click OK

![](https://photos.squarezero.dev/file/abir-images/Volatility/1.png)

Extension for different virtual machine hypervisors →

- VMware → .vmem
- Hyper-V → .bin
- Parallels → .mem
- VirtualBox → .sav

***NOTE: .sav is partial memory file for virtual box and get the full memory file you need to dump memory like bare-metal system.***

***Q1.  What memory format is the most common?***

***A. .raw***

***Q2. The Window's system we're looking to perform memory forensics on was turned off by mistake. What file contains a compressed memory image?***

***A. hiberfil.sys***

***Q3. How about if we wanted to perform memory forensics on a VMware-based virtual machine?***

***A. .vmem***

## Examining Our Patient

***Q4. First, let's figure out what profile we need to use. Profiles determine how Volatility treats our memory image since every version of Windows is a little bit different. Let's see our options now with the command `volatility -f MEMORY_FILE.raw imageinfo`***

***A.*** 

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" imageinfo`

![](https://photos.squarezero.dev/file/abir-images/Volatility/2.png)

***Q5. Running the imageinfo command in Volatility will provide us with a number of profiles we can test with, however, only one will be correct. We can test these profiles using the pslist command, validating our profile selection by the sheer number of returned results. Do this now with the command `volatility -f MEMORY_FILE.raw --profile=PROFILE pslist`. What profile is correct for this memory image?***

***A. WinXPSP2x86***

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" --profile=WinXPSP2x86 pslist`

![](https://photos.squarezero.dev/file/abir-images/Volatility/3.png)

***Q6. Take a look through the processes within our image. What is the process ID for the smss.exe process? If results are scrolling off-screen, try piping your output into less***

***A. 368***

***Q7. In addition to viewing active processes, we can also view active network connections at the time of image creation! Let's do this now with the command `volatility -f MEMORY_FILE.raw --profile=PROFILE netscan`. Unfortunately, something not great is going to happen here due to the sheer age of the target operating system as the command netscan doesn't support it.***

***A.*** 

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" --profile=WinXPSP2x86 netscan`

![](https://photos.squarezero.dev/file/abir-images/Volatility/4.png)

***Q8. It's fairly common for malware to attempt to hide itself and the process associated with it. That being said, we can view intentionally hidden processes via the command `psxview`. What process has only one 'False' listed?***

***A. csrss.exe***

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" --profile=WinXPSP2x86 psxview`

![](https://photos.squarezero.dev/file/abir-images/Volatility/5.png)

***Q9. In addition to viewing hidden processes via psxview, we can also check this with a greater focus via the command 'ldrmodules'. Three columns will appear here in the middle, InLoad, InInit, InMem. If any of these are false, that module has likely been injected which is a really bad thing. On a normal system the grep statement above should return no output. Which process has all three columns listed as 'False' (other than System)?***

***A. csrss.exe***

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" --profile=WinXPSP2x86 ldrmodules | grep False`

![](https://photos.squarezero.dev/file/abir-images/Volatility/6.png)

***Q10. Processes aren't the only area we're concerned with when we're examining a machine. Using the 'apihooks' command we can view unexpected patches in the standard system DLLs. If we see an instance where Hooking module: <unknown> that's really bad. This command will take a while to run, however, it will show you all of the extraneous code introduced by the malware.***

***A.*** 

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" --profile=WinXPSP2x86 apihooks`

![](https://photos.squarezero.dev/file/abir-images/Volatility/7.png)

***Q11. Injected code can be a huge issue and is highly indicative of very very bad things. We can check for this with the command `malfind`. Using the full command `volatility -f MEMORY_FILE.raw --profile=PROFILE malfind -D <Destination Directory>` we can not only find this code, but also dump it to our specified directory. Let's do this now! We'll use this dump later for more analysis. How many files does this generate?***

***A.12***

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" --profile=WinXPSP2x86 malfind -D /home/avi/Downloads/files`

![](https://photos.squarezero.dev/file/abir-images/Volatility/8.png)

![](https://photos.squarezero.dev/file/abir-images/Volatility/9.png)

***Q12. Last but certainly not least we can view all of the DLLs loaded into memory. DLLs are shared system libraries utilized in system processes. These are commonly subjected to hijacking and other side-loading attacks, making them a key target for forensics. Let's list all of the DLLs in memory now with the command `dlllist`***

***A. ***

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" --profile=WinXPSP2x86 dlllist`

![](https://photos.squarezero.dev/file/abir-images/Volatility/10.png)

***Q13. Now that we've seen all of the DLLs running in memory, let's go a step further and pull them out! Do this now with the command `volatility -f MEMORY_FILE.raw --profile=PROFILE --pid=PID dlldump -D <Destination Directory>` where the PID is the process ID of the infected process we identified earlier (questions five and six). How many DLLs does this end up pulling?***

***A. 12 ***

`./volatility -f "/home/avi/Downloads/New Folder/cridex.vmem" --profile=WinXPSP2x86 --pid=584 dlldump -D /home/avi/Downloads/files2/`

![](https://photos.squarezero.dev/file/abir-images/Volatility/11.png)

## Post Actions

Upload the files to [VirusTotal]() and [Hybrid Analysis]() for analysis

***Q14.  Upload the extracted files to VirusTotal for examination.***

***A.***

![](https://photos.squarezero.dev/file/abir-images/Volatility/13.png)

Q15. Upload the extracted files to Hybrid Analysis for examination - *Note, this will also upload to VirusTotal but for the sake of demonstration we have done this separately.*

***A.*** 

![](https://photos.squarezero.dev/file/abir-images/Volatility/12.png)

***Q16. What malware has our sample been infected with? You can find this in the results of VirusTotal and Hybrid Anaylsis.***

***A. cridex***