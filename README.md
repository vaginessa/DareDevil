# DareDevil

[![](https://img.shields.io/badge/Category-Defense%20Evasion-green)](https://github.com/reveng007/) [![](https://img.shields.io/badge/Language-C%23-green)](https://github.com/reveng007/)

Stealthy Loader-cum-dropper/stage-1/stager targeting Windows10

### MUST Before Public:
1. Update Insider.exe

### Technology behind Insider:

![Insider](https://user-images.githubusercontent.com/61424547/176930902-42f5bd9d-c1cd-4c73-a15e-cc5731d55d63.png)

### Ability:
Apart from the above shown diagram other abilties are:
1. Sensitive string Obfuscation using Environmental Keying [TTP ID: T1480.00](https://attack.mitre.org/techniques/T1480/001/). I became aware of this concept from _[<ins>SANS Offensive WorkShop</ins>](https://www.sans.org/offensive-operations/): ['From zero to hero: Creating a reflective loader in C#'](https://www.youtube.com/watch?v=qeOCZGuVsi4)_ by [@Jean_Maes_1994](https://twitter.com/jean_maes_1994) and from [FalconStrike](https://slaeryan.github.io/posts/falcon-zero-alpha.html) by [@_winterknife_](https://twitter.com/_winterKnife_).
2. All function calls are Obfuscated using delegate, except `LoadLibrary()` and `GetProcAddress()`.\
Tried using DInvoke to Obfuscate `LoadLibrary()` and `GetProcAddress()` but instead, it got detected by 5 more AV engines ;(
4. Abilty to Detect and Detach from debugger by using, `NtQueryInformationProcess()` and `NtRemoveProcessDebug()` respectively.

### Tools: Usage

- Obfuscator/encrypt.cs:
1. [For shellcode extraction and encryption]: place it in the directory in which .bin file is present.
2. [For url encryption]: Nothing! Just paste and run.

Example:
```
It can encrypt 'shellcode', 'url' and string:

// Creating .bin file and Extracting shellcode from .bin file:
// Creating: https://ivanitlearning.wordpress.com/2018/10/14/shellcoding-with-msfvenom/
// Extract: 
cmd> encrypt.exe /file:file.bin /out:aes_b64

// paste the output b64 bytes into a .txt file and upload it to payload server.
// cmd: "mv .\obfuscator\"
cmd> encrypt.exe /shellcodeurl:<url>.txt /out:aes_b64

cmd> encrypt.exe /mscorliburl:<url>.exe /out:aes_b64

// For Sending/ exfiltrating Victim process name and Ids to Operator Gmail via SMTP server
cmd> encrypt.exe /remotewriteurl:<url>.exe /out:aes_b64

// For reading pid from pid.txt from payload server/ remote c2 server
cmd> encrypt.exe /remotereadurl:<url>.txt /out:aes_b64

// Sensitive strings Obfuscation
cmd> encrypt.exe /string:<string1,string2,...,stringn> /xor_key:<usernameoftarget> /out:xor_b64
```

- stage2/remotewrite.cs:\
Why I made RemoteWrite.cs?
```
1. If I wanted to make this loader_cum_dropper invisible (compilation: csc.exe /target:winexe /platform:x64 /out:Insider.exe  .\Insider.cs), the enumerated process name and process id, made by it, can't be seen by me.
2. I also can't create a file on disk, it will not be OPSEC safe.
3. So, I exfiltrate those enumerated process names and ids via Gmail's SMTP server, by sending a string (full of process names and process ids, all appended together).
4. After getting the gmail, the operator will choose the pid, he/she want to victimise.
5. Then he/she will create a `pid.txt` file with target pid written in it and then host it in payload server/ C2 server of his/her choice.
6. Until our loader_cum_dropper gets pid from url containing pid.txt, it will keep on retrying.
7. From there, our dropper will read those pid and perform injection.
```
In this way, I made un-interactive program, interactive.

- stage2/mscorlib.cs:\
It is the re-implementation of AMSI and ETW bypass done in [SharpSploit](https://github.com/cobbr/SharpSploit/blob/master/SharpSploit/Evasion/ETW.cs) and [AmsiScanBufferBypass](https://github.com/rasta-mouse/AmsiScanBufferBypass/blob/main/AmsiBypass.cs) by [@RastaMouse](https://twitter.com/_rastamouse?lang=en). It was actually covered by [@Jean_Maes_1994](https://twitter.com/jean_maes_1994) in his workshop in <ins>SANS Offensive</ins>.

- stage1/Insider.cs:
Usage:
```
1. I have used Developer Powershell/cmd for VS 2019
2. cmd> git clone https://github.com/reveng007/DareDevil
3. Compile: Obfuscator/encrypt.cs with compile.bat, stage2/remotewrite.cs with compile_remotewrite.bat (but at first write the credentials of sender's and receiver's/Operator's gmail) and stage2/mscorlib.cs with compile_mscorlib.bat.
4. Now upload/ host those two stage2 in payload server/ github(github: because it will not be considered as malicious as it is considered to be a legitimate website. So, malware traffic from github will not be considered as creepy stuff, instead of that, it would be considered as legitimate).
5. Encrypt those two urls using "Obfuscator/encrypt.exe" file with the previously mentioned flags and use those in "stage1/Insider.cs".
6. Encrypt your shellcode, by following my previously mentioned flags in "Obfuscator/encrypt.exe" section and paste the encrypted shellcode in a text file host it in payload server/ github. Then again encrypt that url with "Obfuscator/encrypt.exe" and paste that in "stage1/Insider.cs".
7. Now, compile the "stage1/Insider.cs" with compile.bat and put it in an antivirus enabled windows 10 nad test it.
```
#### NOTE:
I have named AMSI&ETW bypass .NET Assembly as "_mscorlib_" because if by chance, it is seen by a Blue Teamer and if that particular member is less experienced, the name `"mscorlib"` can bamboozle, making them think, "Hey, yes!! a .NET binary always loads up something called, mscorlib. It contains the core implementation of the .NET framework." Though there is a very little chance of our "_mscorlib.exe_" of getting caught running as a process in memory, as it is visible only a very little amount of time (probably in ms) in our dropper process memory, unless our dropper is getting debugged ;(.\
BTW, This bamboozle thing was also told by Jean Maes :smile:.

### Video:

https://user-images.githubusercontent.com/61424547/177029414-1b3b09e0-d00c-4b96-882f-aa614f2a44ec.mp4

### <ins>Internal Noticing</ins>:

#### ProcessHacker (only):

https://user-images.githubusercontent.com/61424547/176947727-e37a484c-db28-495f-8cb2-0ab6eb1a3c81.mp4

I saw that even after providing Read-Execute permission to the allocated shellcode memory region, it wasn't shown as RX in ProcessHacker. Strangely enough, the bool value for VirtualProtectEx was also ***True*** while protecting target process memory with 0x20 [PAGE_EXECUTE_READ](https://docs.microsoft.com/en-us/windows/win32/Memory/memory-protection-constants#constants).
I think this is happening because we applied page RW memory protection with `VirtualAllocEx()`. Then before creating the remote thread, we are allocating RX memory protection with `VirtualProtectEx()`. Not that sure of. If anybody seeing this, know about this, please correct me ;(

#### <ins>[Moneta](https://github.com/forrest-orr/moneta)</ins>:
But with moneta, we can see it. 

![moneta](https://user-images.githubusercontent.com/61424547/176948027-7bdc8c7e-7773-48a1-ae9f-06ea54b700be.png)

But without knowing the actual address, it is not getting shown by Process Hacker. It only not be visible from outside. It can bypass BlueTeam, until the BlueTeamer isn't aware of this particular process memory address.

![processHacker](https://user-images.githubusercontent.com/61424547/176948132-1ffce1c6-ac63-472d-b8bc-a217791ab911.png)

#### <ins>[Floss](https://github.com/mandiant/flare-floss)</ins>:

I am able to bypass more or less all sensitive strings except, _Kernel32.dll_ as it is getting used by unobfuscated function call named, `LoadLibrary()` and `GetProcAddress()`. Other strings which caught my eyes were, _PROCESS_BASIC_INFORMATION_ and _PROCESSSINFOCLASS_, but I don't really think those things matter. But if those do, please do correct me.\
As again, I'm learning :)

#### WireShark Capture:

![SMTP_wireshark](https://user-images.githubusercontent.com/61424547/176946689-09192c06-6894-4d3b-a034-1a641d7c4de4.png)

We can see that the text(process infos) sent out are all encrypted by Gmail's TLS encryption. On top of that, the ip address (marked) isn't suspicious at all, or in other words are OPSEC safe.

![MailServer_iplookup](https://user-images.githubusercontent.com/61424547/176946957-60f1dce9-983e-4314-9fd8-6f54cbc04de7.PNG)

#### AV Bypass [Antiscan.me]():

![](https://antiscan.me/images/result/WK9nNvebEk8v.png)

Yupp! It got detected by one-and-only, ***Eset NOD32!***\
No matter what I do, this is flagging me :worried:.

When I commented out Loader code part, I got this: [loader](https://antiscan.me/scan/new/result?id=ixWtfrWl0H3u)\
When I commented out Dropper code part, I got this: [dropper](https://antiscan.me/scan/new/result?id=nhjBNvvssumL)\
When I used the whole binary, I got the above detection stats as I had already shown above :woozy_face:. \
According to mathematics, I  should have got 4 detections, why one and why that particular AV only?\
I am really not getting it, if I get time, I will look into it, again. Not getting a cleansheet is really annoying ;(

### Edit: 
If you Ask me, "Why have I used _AES_ encryption with url why not environmental keying factor?"\
My answer would be, that would also work fine!\
I was feeling lazy to remove AES and apply environmental XOR keying. But this will not do any harm. But I changed the string obfuscation part from AES encryption to environmental XOR keying as if I did not do that, this dropper would be flagged by _Floss_ by outputing sentive strings like hardcoded xor keys. But in this case, we are using trgt machine's username as xor key, so no need of hardcoding.

### Resources and Credits:

1. I learned Reflective loader implementation from watching the _[<ins>SANS Offensive WorkShop</ins>](https://www.sans.org/offensive-operations/): ['From zero to hero: Creating a reflective loader in C#'](https://www.youtube.com/watch?v=qeOCZGuVsi4)_ by [@Jean_Maes_1994](https://twitter.com/jean_maes_1994). Also thanks to him, for answering my questions in DM, so patiently :smile:!
2. Other basics and offensive side of C# by following offensive tradecraft basics from _[<ins>DEFCON 29 Adversary Village</ins>](https://adversaryvillage.org/): ['Tradecraft Development in Adversary Simulations`](https://youtu.be/KJsVVEn4fFw)_ by [@fozavci](https://twitter.com/fozavci).
3. PInvoke:
    - [pinvoke](http://www.pinvoke.net/)
    - [specterops:Matt Hand](https://posts.specterops.io/offensive-p-invoke-leveraging-the-win32-api-from-managed-code-7eef4fdef16d)
4. delegate: [YT:Tech69](https://www.youtube.com/c/Tech69YT). He also a amaizing dude!
5. Obviously, the infamous [RTO:MalwareDevelopmentEssentials](https://institute.sektor7.net/red-team-operator-malware-development-essentials) course by [@SEKTOR7net](https://twitter.com/sektor7net).
6. [@_winterknife_](https://twitter.com/_winterKnife_): For clearly making me understand the difference between stage-0, stage-1, stage-2, stage-3, etc payloads.
7. Took reference from [FalconStrike](https://slaeryan.github.io/posts/falcon-zero-alpha.html) by [@_winterknife_](https://twitter.com/_winterKnife_).
8. [@SoumyadeepBas12](https://twitter.com/SoumyadeepBas12): For helping me out when I got stuck doing this project.

### Author: @reveng007 (Soumyanil Biswas)
---
[![](https://img.shields.io/badge/Twitter-@reveng007-1DA1F2?style=flat-square&logo=twitter&logoColor=white)](https://twitter.com/reveng007)
[![](https://img.shields.io/badge/LinkedIn-@SoumyanilBiswas-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/soumyanil-biswas/)
[![](https://img.shields.io/badge/Github-@reveng007-0077B5?style=flat-square&logo=github&logoColor=black)](https://github.com/reveng007/)
