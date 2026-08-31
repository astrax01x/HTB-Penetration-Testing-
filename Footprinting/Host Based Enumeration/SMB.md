`Server Message Block` (`SMB`) is a client-server protocol that regulates access to files and entire directories and other network resources such as printers, routers, or interfaces released for the network. Information exchange between different system processes can also be handled based on the SMB protocol

An SMB server can provide arbitrary parts of its local file system as shares. Therefore the hierarchy visible to a client is partially independent of the structure on the server. Access rights are defined by `Access Control Lists` (`ACL`). They can be controlled in a fine-grained manner based on attributes such as `execute`, `read`, and `full access` for individual users or user groups. The ACLs are defined based on the shares and therefore do not correspond to the rights assigned locally on the server.

## Samba

As mentioned earlier, there is an alternative implementation of the SMB server called Samba, which is developed for Unix-based operating systems. Samba implements the Common Internet File System (`CIFS`) network protocol. [CIFS](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-cifs/934c2faa-54af-4526-ac74-6a24d126724e) is a dialect of SMB, meaning it is a specific implementation of the communicate effectively with newer Windows systems. Therefore, it is often referred to as SMB/CIFS.
#### Restart Samba

	root@samba:~# sudo systemctl restart smbd

Now we can display a list (`-L`) of the server's shares with the `smbclient` command from our host. We use the so-called `null session` (`-N`), which is `anonymous` access without the input of existing users or valid passwords.

#### SMBclient - Connecting to the Share

	 AstraX01@htb[/htb]$ smbclient -N -L //10.129.14.128

We can see that we now have five different shares on the Samba server from the result. Thereby `print$` and an `IPC$` are already included by default in the basic setting, as we have already seen. Since we deal with the `[notes]` share, let us log in and inspect it using the same client program. If we are not familiar with the client program, we can use the `help` command on successful login, listing all the possible commands we can execute.

	 AstraX01@htb[/htb]$ smbclient //10.129.14.128/notes
	 smb: \> help
	 smb: \> ls

#### Download Files from SMB

	smb: \> get prep-prod.txt
	smb: \> !ls
	smb: \> !cat prep-prod.txt

#### Samba Status

	 root@samba:~# smbstatus

## Footprinting the Service

First, however, let us see what Nmap can find on our target Samba server, where we created the `[notes]` share for testing purposes.

#### Nmap

	 AstraX01@htb[/htb]$ sudo nmap 10.129.14.128 -sV -sC -p139,445

*We can see from the results that it is not very much that Nmap provided us with here. Therefore, we should resort to other tools that allow us to interact manually with the SMB and send specific requests for the information. One of the handy tools for this is `rpcclient`. This is a tool to perform MS-RPC functions.*

The [Remote Procedure Call](https://www.geeksforgeeks.org/remote-procedure-call-rpc-in-operating-system/) (`RPC`) is a concept and, therefore, also a central tool to realize operational and work-sharing structures in networks and client-server architectures. The communication process via RPC includes passing parameters and the return of a function value.
#### RPCclient

	 AstraX01@htb[/htb]$ rpcclient -U "" 10.129.14.128

The `rpcclient` offers us many different requests with which we can execute specific functions on the SMB server to get information. A complete list of all these functions can be found on the [man page](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) of the rpcclient.

|**Query**|**Description**|
|---|---|
|`srvinfo`|Server information.|
|`enumdomains`|Enumerate all domains that are deployed in the network.|
|`querydominfo`|Provides domain, server, and user information of deployed domains.|
|`netshareenumall`|Enumerates all available shares.|
|`netsharegetinfo <share>`|Provides information about a specific share.|
|`enumdomusers`|Enumerates all domain users.|
|`queryuser <RID>`|Provides information about a specific user.|
#### RPCclient - Enumeration

	rpcclient $> srvinfo
	rpcclient $> enumdomains
	rpcclient $> querydominfo
	rpcclient $> netshareenumall
	rpcclient $> netsharegetinfo notes

*These examples show us what information can be leaked to anonymous users. Once an `anonymous` user has access to a network service, it only takes one mistake to give them too many permissions or too much visibility to put the entire network at significant risk.*

Most importantly, anonymous access to such services can also lead to the discovery of other users, who can be attacked with brute-forcing in the most aggressive case. Humans are more error-prone than properly configured computer processes, and the lack of security awareness and laziness often leads to weak passwords that can be easily cracked.
#### Rpcclient - User Enumeration

	rpcclient $> enumdomusers
	
	user:[mrb3n] rid:[0x3e8] 
	user:[cry0l1t3] rid:[0x3e9]

	rpcclient $> queryuser 0x3e9
	rpcclient $> queryuser 0x3e8

We can then use the results to identify the group's RID, which we can then use to retrieve information from the entire group.
#### Rpcclient - Group Information

	 rpcclient $> querygroup 0x201

However, it can also happen that not all commands are available to us, and we have certain restrictions based on the user. However, the query `queryuser <RID>` is mostly allowed based on the RID. So we can use the rpcclient to brute force the RIDs to get information. Because we may not know who has been assigned which RID, we know that we will get information about it as soon as we query an assigned RID. There are several ways and tools we can use for this. To stay with the tool, we can create a `For-loop` using `Bash` where we send a command to the service using rpcclient and filter out the results.
#### Brute Forcing User RIDs

	AstraX01@htb[/htb]$ for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

#### SMBmap

	 AstraX01@htb[/htb]$ smbmap -H 10.129.14.128

#### CrackMapExec

	AstraX01@htb[/htb]$ crackmapexec smb 10.129.14.128 --shares -u '' -p ''

#### Enum4Linux-ng - Enumeration

	AstraX01@htb[/htb]$ ./enum4linux-ng.py 10.129.14.128 -A

