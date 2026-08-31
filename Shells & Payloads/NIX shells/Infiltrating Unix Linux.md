Over `70%` of websites (webservers) run on a Unix-based system. this means we can significantly benefit from continuing to grow our knowledge of Unix/Linux and how we can gain shell sessions on these environments to potentially pivot further within an environment

## Common Considerations

Gaining a shell session with a system can be done in various ways, one common way is through a vulnerability in an application. We will identify a vulnerability and discover an exploit that we can use to gain a shell by delivering a payload.

## Gaining a Shell Through Attacking a Vulnerable Application

As in most engagements, we will start with an initial enumeration of the system using `Nmap`.

#### Enumerate the Host

	AstraX01@htb[/htb]$ nmap -sC -sV 10.129.201.101

Keeping our goal of `gaining a shell session` in mind, we must establish some next steps after examining our scan output.

`What information could we gather from the output?`

Considering we can see the system is listening on ports 80 (`HTTP`), 443 (`HTTPS`), 3306 (`MySQL`), and 21 (`FTP`), it may be safe to assume that this is a web server hosting a web application. We can also see some version numbers revealed associated with the web stack (`Apache 2.4.6` and `PHP 7.2.34` ) and the distribution of Linux running on the system (`CentOS`). Before deciding on a direction to research further (dive down a rabbit hole), we should also try navigating to the IP address through a web browser to discover the hosted application if possible.

Here we discover a network configuration management tool called [rConfig](https://www.rconfig.com/). This application is used by network & system administrators to automate the process of configuring network appliances. One practical use case would be to use rConfig to remotely configure network interfaces with IP addressing information on multiple routers simultaneously. This tool saves admins time but, if compromised, could be used to pivot onto critical network devices that switch & route packets across the network. A malicious attacker could own the entire network through rConfig since it will likely have admin access to all the network appliances used to configure. As pentesters, finding a vulnerability in this application would be considered a very critical discovery.

---

## Discovering a Vulnerability in rConfig

Take a close look at the bottom of the web login page, and we can see the rConfig version number (`3.9.6`). We should use this information to start looking for any `CVEs`, `publicly available exploits`, and `proof of concepts` (`PoCs`). As we research, be sure to look closely at what we find and understand what it is doing. We, of course, want it to lead us to a shell session with the target.

The same thinking could be applied to the Apache and PHP versions, but since the application is running on the web stack, let's see if we can gain a shell through an exploit written for the vulnerabilities found in rConfig.

We can also use Metasploit's search functionality to see if any exploit modules can get us a shell session on the target.

#### Search For an Exploit Module

	msf6 > search rconfig

One detail that can be overlooked when relying on MSF to find an exploit module for a specific application is the version of MSF. There may be useful exploit modules that are not installed on our system or just aren't showing up via search. In these cases, it's good to know that Rapid 7 keeps code for exploit modules in their [repos on github](https://github.com/rapid7/metasploit-framework/tree/master/modules/exploits). We could do an even more specific search using a search engine: `rConfig 3.9.6 exploit metasploit github`

This search can point us to the source code for an exploit module called `rconfig_vendors_auth_file_upload_rce.rb`. This exploit can get us a shell session on a target Linux box running rConfig 3.9.6. If this exploit did not show up in the MSF search, we can copy the code from this repo onto our local attack box and save it in the directory that our local install of MSF is referencing. To do this, we can issue this command on our attack box:

#### Locate

	AstraX01@htb[/htb]$ locate exploits

We can copy the code into a file and save it in `/usr/share/metasploit-framework/modules/exploits/linux/http` similar to where they are storing the code in the GitHub repo.

Once we find the exploit module and download it (we can use wget) or copy it into the proper directory from Github, we can use it to gain a shell session on the target. If we copy it into a file on our local system, make sure the file has `.rb` as the extension. All modules in MSF are written in Ruby.

## Using the rConfig Exploit and Gaining a Shell

In msfconsole, we can manually load the exploit using the command:

#### Select an Exploit

	msf6 > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce

With this exploit selected, we can list the options, input the proper settings specific to our network environment, and launch the exploit.

#### Execute the Exploit

	msf6 exploit(linux/http/rconfig_vendors_auth_file_upload_rce) > exploit
	meterpreter > dir

We can see from the steps outlined in the exploitation process that this exploit:

- Checks for the vulnerable version of rConfig
- Authenticates with the rConfig web login
- Uploads a PHP-based payload for a reverse shell connection
- Deletes the payload
- Leaves us with a Meterpreter shell session

#### Interact With the Shell

	meterpreter > shell

We can drop into a system shell (`shell`) to gain access to the target system as if we were logged in and open a terminal.

---

## Spawning a TTY Shell with Python

When we drop into the system shell, we notice that no prompt is present, yet we can still issue some system commands. This is a shell typically referred to as a `non-tty shell`. These shells have limited functionality and can often prevent our use of essential commands like `su` (`switch user`) and `sudo` (`super user do`), which we will likely need if we seek to escalate privileges. This happened because the payload was executed on the target by the apache user.

We can manually spawn a TTY shell using Python if it is present on the system. We can always check for Python's presence on Linux systems by typing the command: `which python`. To spawn the TTY shell session using Python, we type the following command:

#### Interactive Python

	python -c 'import pty; pty.spawn("/bin/sh")'


