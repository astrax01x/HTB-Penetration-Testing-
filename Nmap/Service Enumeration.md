It is essential to determine the application and its version as accurately as possible. We can use this information to scan for known vulnerabilities and analyze the source code for that version if we find it. An exact version number allows us to search for a more precise exploit that fits the service and the operating system of our target.

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV

Another option (`--stats-every=5s`) that we can use is defining how periods of time the status should be shown. Here we can specify the number of seconds (`s`) or minutes (`m`), after which we want to get the status. 

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV --stats-every=5s

## Banner Grabbing

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -p- -sV

#### Tcpdump

	 AstraX01@htb[/htb]$ sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28

#### Nc 

	 AstraX01@htb[/htb]$ nc -nv 10.129.2.28 25

