
---

## Enumerating the Password Policy - from Linux - Credentialed

The password policy can also be obtained remotely using tools such as [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) or `rpcclient`.

	AstraX01@htb[/htb]$ crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol

## Enumerating the Password Policy - from Linux - SMB NULL Sessions

Without credentials, we may be able to obtain the password policy via an SMB NULL session or LDAP anonymous bind. The first is via an SMB NULL session. SMB NULL sessions allow an unauthenticated attacker to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy.

we can use tools such as `enum4linux`, `CrackMapExec`, `rpcclient`

#### Using rpcclient For SMB NULL SESSION 

	AstraX01@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

Once connected, we can issue an RPC command such as `querydominfo` to obtain information about the domain and confirm NULL session access.

	rpcclient $> querydominfo

We can also obtain the password policy

	rpcclient $> getdompwinfo

---

Here are some common enumeration tools and the ports they use:

|Tool|Ports|
|---|---|
|nmblookup|137/UDP|
|nbtstat|137/UDP|
|net|139/TCP, 135/TCP, TCP and UDP 135 and 49152-65535|
|rpcclient|135/TCP|
|smbclient|445/TCP|

#### Using enum4linux for SMB Null Session 

	AstraX01@htb[/htb]$ enum4linux -P 172.16.5.5

#### Using enum4linux-ng

The tool [enum4linux-ng](https://github.com/cddmp/enum4linux-ng) is a rewrite of `enum4linux` in Python, but has additional features such as the ability to export data as YAML or JSON files which can later be used to process the data further or feed it to other tools. It also supports colored output, among other features

	AstraX01@htb[/htb]$ enum4linux-ng -P 172.16.5.5 -oA ilfreight

Enum4linux-ng provided us with a bit clearer output and handy JSON and YAML output using the `-oA` flag.


---

## Enumerating Null Session - from Windows

It is less common to do this type of null session attack from Windows, but we could use the command `net use \\host\ipc$ "" /u:""` to establish a null session from a windows machine and confirm if we can perform more of this type of attack.

#### Establish a null session from windows

	C:\htb> net use \\DC01\ipc$ "" /u:"" 
	The command completed successfully. 

We can also use a username/password combination to attempt to connect.

#### Error: Account is Disabled 

	C:\htb> net use \\DC01\ipc$ "" /u:guest 
	System error 1331 has occurred. 
	
	This user can't sign in because this account is currently disabled. 

#### Error: Password is Incorrect 

	C:\htb> net use \\DC01\ipc$ "password" /u:guest 
	System error 1326 has occurred. 
	
	The user name or password is incorrect.

#### Error: Account is locked out (Password Policy)

	C:\htb> net use \\DC01\ipc$ "password" /u:guest 
	System error 1909 has occurred. 
	
	The referenced account is currently locked out and may not be logged on to. 


## Enumerating the Password Policy - from Linux - LDAP Anonymous Bind 

With an LDAP anonymous bind, we can use LDAP-specific enumeration tools such as `windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, etc., to pull the password policy.

#### Using ldapsearch 

	AstraX01@htb[/htb]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength

## Enumerating the Password Policy - from Windows

we can use built-in Windows binaries such as `net.exe` to retrieve the password policy. We can also use various tools such as PowerView, CrackMapExec ported to Windows, SharpMapExec, SharpView, etc.

#### Using net.exe 

	C:\htb> net accounts

	Force user logoff how long after time expires?:            Never 
	Minimum password age (days):                               1 
	Maximum password age (days):                               Unlimited 
	Minimum password length:                                   8 
	Length of password history maintained:                     24 
	Lockout threshold:                                         5 
	Lockout duration (minutes):                                30 
	Lockout observation window (minutes):                      30 
	Computer role:                                             SERVER 
	The command completed successfully. 

#### Using PowerView For Enumerating Password Policy -  From Windows 

	PS C:\htb> import-module .\PowerView.ps1 
	PS C:\htb> Get-DomainPolicy

PowerView gave us the same output as our `net accounts` command, just in a different format but also revealed that password complexity is enabled (`PasswordComplexity=1`).



## Next Steps

Now that we have the password policy in hand, we need to create a target user list to perform our password spraying attack.


