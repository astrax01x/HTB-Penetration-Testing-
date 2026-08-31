Nmap Scripting Engine (`NSE`) is another handy feature of `Nmap`. It provides us with the possibility to create scripts in Lua for interaction with certain services. There are a total of 14 categories into which these scripts can be divided:

#### Default Scripts

	 AstraX01@htb[/htb]$ sudo nmap <target> -sC

#### Specific Scripts Category

	 AstraX01@htb[/htb]$ sudo nmap <target> --script <category>

#### Defined Scripts

	 AstraX01@htb[/htb]$ sudo nmap <target> --For example, 
 
let us keep working with the target SMTP port and see the results we get with two defined scripts.

#### Nmap - Specifying Scripts...

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands

`Nmap` also gives us the ability to scan our target with the aggressive option (`-A`). This scans the target with multiple options as service detection (`-sV`), OS detection (`-O`), traceroute (`--traceroute`), and with the default NSE scripts (`-sC`).

#### Nmap - Aggressive Scan

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -A

## Vulnerability Assessment

#### Nmap - Vuln Category

	 AstraX01@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sV --script vuln

