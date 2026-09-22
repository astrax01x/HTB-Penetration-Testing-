## SSH Local Port Forwarding

	AstraX01@htb[/htb]$ ssh -L 1234:localhost:3306 ubuntu@10.129.202.64

The `-L` command tells the SSH client to request the SSH server to forward all the data we send via the port `1234` to `localhost:3306` on the Ubuntu server. By doing this, we should be able to access the MySQL service locally on port 1234. We can use Netstat or Nmap to query our local host on 1234 port to verify whether the MySQL service was forwarded.

#### Confirming Port Forward with Netstat

	AstraX01@htb[/htb]$ netstat -antp | grep 1234

	(Not all processes could be identified, non-owned process info will not be shown, you would have to be root to see it all.) 
	tcp        0       0 127.0.0.1:1234       0.0.0.0:*       LISTEN 
	4034/ssh 
	
	tcp6       0       0 ::1:1234             :::*            LISTEN 
	4034/ssh

#### Confirming Port Forward with Nmap

	AstraX01@htb[/htb]$ nmap -v -sV -p1234 localhost

	PORT      STATE     SERVICE    VERSION 
	1234/tcp  open      mysql      MySQL 8.0.28-0ubuntu0.20.04.3

Similarly, if we want to forward multiple ports from the Ubuntu server to your localhost, you can do so by including the `local port:server:port` argument to your ssh command. For example, the below command forwards the apache web server's port 80 to your attack host's local port on `8080`.

#### Forwarding Multiple Ports

	AstraX01@htb[/htb]$ ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64


---

#### Looking for Opportunities to Pivot using ifconfig

	ubuntu@WEB01:~$ ifconfig

We need to scan the 172.16.5.0/23 network, but our attack machine can't reach it directly.

**Solution:** Use the compromised Ubuntu server as a pivot.

1. Set up a **SOCKS proxy** on our local machine.
2. Use **SSH dynamic port forwarding** through the Ubuntu host to route our scan traffic into the target network.
3. Configure our tools (like nmap) to use this proxy, allowing us to discover and scan hosts on `172.16.5.0/23`.

#### Enabling Dynamic Port Forwarding with SSH

	AstraX01@htb[/htb]$ ssh -D 9050 ubuntu@10.129.202.64

The `-D` argument requests the SSH server to enable dynamic port forwarding. Once we have this enabled, we will require a tool that can route any tool's packets over the port `9050`. We can do this using the tool `proxychains`.

#### Checking /etc/proxychains.conf

To inform proxychains that we must use port 9050, we must modify the proxychains configuration file located at `/etc/proxychains.conf`. We can add `socks4 127.0.0.1 9050` to the last line if it is not already there

	AstraX01@htb[/htb]$ tail -4 /etc/proxychains.conf

#### Using Nmap with Proxychains

Now when you start Nmap with proxychains using the below command, it will route all the packets of Nmap to the local port 9050, where our SSH client is listening, which will forward all the packets over SSH to the 172.16.5.0/23 network.

	AstraX01@htb[/htb]$ proxychains nmap -v -sn 172.16.5.1-200

### **IMPORTANT:**

**One more important note to remember here is that we can only perform a `full TCP connect scan` over proxychains. **

We also need to make sure we are aware of the fact that `host-alive` checks may not work against Windows targets because the Windows Defender firewall blocks ICMP requests (traditional pings) by default.

#### Enumerating the Windows Target through Proxychains

	AstraX01@htb[/htb]$ proxychains nmap -v -Pn -sT 172.16.5.19

Similar to the Nmap scan, we can also pivot `msfconsole` via proxychains to perform vulnerable RDP scans using Metasploit auxiliary modules. We can start msfconsole with proxychains.

## Using Metasploit with Proxychains

	AstraX01@htb[/htb]$ proxychains msfconsole

Let's use the `rdp_scanner` auxiliary module to check if the host on the internal network is listening on 3389.

#### Using rdp_scanner Module

	msf6 > search rdp_scanner

	msf6 > use 0
	msf6 auxiliary(scanner/rdp/rdp_scanner) > set rhosts 172.16.5.19 
	rhosts => 172.16.5.19

	msf6 auxiliary(scanner/rdp/rdp_scanner) > run

#### Using xfreerdp with Proxychains

	AstraX01@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123

