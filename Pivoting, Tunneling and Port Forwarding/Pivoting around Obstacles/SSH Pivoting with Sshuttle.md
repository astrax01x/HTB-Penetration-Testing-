[Sshuttle](https://github.com/sshuttle/sshuttle) is another tool written in Python which removes the need to configure proxychains. However, this tool only works for pivoting over SSH and does not provide other options for pivoting over TOR or HTTPS proxy servers.

#### Installing sshuttle

	AstraX01@htb[/htb]$ sudo apt-get install sshuttle

#### Running sshuttle

To use sshuttle, we specify the option `-r` to connect to the remote machine with a username and password. Then we need to include the network or IP we want to route through the pivot host, in our case, is the network 172.16.5.0/23.

	AstraX01@htb[/htb]$ sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v

#### Traffic Routing through iptables Routes

With this command, sshuttle creates an entry in our `iptables` to redirect all traffic to the 172.16.5.0/23 network through the pivot host.

	AstraX01@htb[/htb]$ sudo nmap -v -A -sT -p3389 172.16.5.19 -Pn

