Password spraying can result in gaining access to systems and potentially gaining a foothold on a target network. The attack involves attempting to log into an exposed service using one common password and a longer list of usernames or email addresses. The usernames and emails may have been gathered during the OSINT phase of the penetration test or our initial enumeration attempts. 

## Enumerating the Password Policy - from Linux

 we can pull the domain password policy in several ways, depending on how the domain is configured and whether or not we have valid domain credentials. With valid domain credentials, the password policy can also be obtained remotely using tools such as [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) or `rpcclient`.

	 crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol 

## Enumerating the Password Policy - from Linux - SMB NULL Sessions

Without credentials, we may be able to obtain the password policy via an SMB NULL session or LDAP anonymous bind. The first is via an SMB NULL session. SMB NULL sessions allow an unauthenticated attacker to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy

*An SMB NULL session can be enumerated easily. For enumeration, we can use tools such as `enum4linux`, `CrackMapExec`, `rpcclient`*

#### Using rpcclient

	 AstraX01@htb[/htb]$ rpcclient -U "" -N 172.16.5.5 
	 
	 rpcclient $> querydominfo Domain: INLANEFREIGHT Server: Comment: Total Users: 3650 Total Groups: 0 Total Aliases: 37 Sequence No: 1 Force Logoff: -1 Domain Server State: 0x1 Server Role: ROLE_DOMAIN_PDC Unknown 3: 0x1


We can also obtain the password policy. We can see that the password policy is relatively weak, allowing a minimum password of 8 characters.

#### Obtaining the Password Policy using rpcclient

	 rpcclient $> querydominfo

#### Using enum4linux

	 AstraX01@htb[/htb]$ enum4linux -P 172.16.5.5

## Enumerating Null Session - from Windows

It is less common to do this type of null session attack from Windows, but we could use the command `net use \\host\ipc$ "" /u:""` to establish a null session from a windows machine and confirm if we can perform more of this type of attack.

#### Establish a null session from windows

	 C:\htb> net use \\DC01\ipc$ "" /u:"" 
	 The command completed successfully.

We can also use a username/password combination to attempt to connect. Let's see some common errors when trying to authenticate:

#### Error: Account is Disabled

	 C:\htb> net use \\DC01\ipc$ "" /u:guest 
	 System error 1331 has occurred. 
	 
	 This user can't sign in because this account is currently disabled.

## Enumerating the Password Policy - from Linux - LDAP Anonymous Bind

[LDAP anonymous binds](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled) allow unauthenticated attackers to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy.

With an LDAP anonymous bind, we can use LDAP-specific enumeration tools such as `windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, etc., to pull the password policy

#### Using ldapsearch

	 ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength

## Enumerating the Password Policy - from Windows

If we can authenticate to the domain from a Windows host, we can use built-in Windows binaries such as `net.exe` to retrieve the password policy. We can also use various tools such as PowerView, CrackMapExec ported to Windows, SharpMapExec, SharpView, etc.

#### Using net.exe

	 C:\htb> net accounts

This password policy is excellent for password spraying. The eight-character minimum means that we can try common weak passwords such as `Welcome1`

#### Using PowerView

	 PS C:\htb> import-module .\PowerView.ps1
	 PS C:\htb> Get-DomainPolicy

