## Misconfigurations

Since RDP takes user credentials for authentication, one common attack vector against the RDP protocol is password guessing. Although it is not common, we could find an RDP service without a password if there is a misconfiguration.

#### Crowbar - RDP Password Spraying

Using the [Crowbar](https://github.com/galkan/crowbar) tool, we can perform a password spraying attack against the RDP service. As an example below, the password `password123` will be tested against a list of usernames in the `usernames.txt` file. The attack found the valid credentials as `administrator` : `password123` on the target RDP host.

	AstraX01@htb[/htb]# crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'

#### Hydra - RDP Password Spraying

We can also use `Hydra` to perform an RDP password spray attack.

	AstraX01@htb[/htb]# hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp

#### RDP Login

We can RDP into the target system using the `rdesktop` client or `xfreerdp` client with valid credentials.

`rdesktop login: `

	AstraX01@htb[/htb]# rdesktop -u admin -p password123 192.168.2.143

## Protocol Specific Attacks

Let's imagine we successfully gain access to a machine and have an account with local administrator privileges. If a user is connected via RDP to our compromised machine, we can hijack the user's remote desktop session to escalate our privileges and impersonate the account.

In an Active Directory environment, this could result in us taking over a Domain Admin account or furthering our access within the domain.

#### RDP Session Hijacking

As shown in the example below, we are logged in as the user `juurena` (UserID = 2) who has `Administrator` privileges. Our goal is to hijack the user `lewen` (User ID = 4), who is also logged in via RDP.

To successfully impersonate a user without their password, we need to have `SYSTEM` privileges and use the Microsoft [tscon.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tscon) binary that enables users to connect to another desktop session.

	C:\htb> tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}

the following command will open a new console as the specified `SESSION_ID` within our current RDP session:

If we have local administrator privileges, we can use several methods to obtain `SYSTEM` privileges, such as [PsExec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec) or [Mimikatz](https://github.com/gentilkiwi/mimikatz).

A simple trick is to create a Windows service that, by default, will run as `Local System` and will execute any binary with `SYSTEM` privileges. We will use [Microsoft sc.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-create) binary. First, we specify the service name (`sessionhijack`) and the `binpath`, which is the command we want to execute. Once we run the following command, a service named `sessionhijack` will be created.

	C:\htb> query user

	C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"

![[rdp_session-2-2.jpg]]


To run the command, we can start the `sessionhijack` service :

	C:\htb> net start sessionhijack

Once the service is started, a new terminal with the `lewen` user session will appear. With this new account, we can attempt to discover what kind of privileges it has on the network, and maybe we'll get lucky, and the user is a member of the Help Desk group with admin rights to many hosts or even a Domain Admin.


## RDP Pass-the-Hash (PtH)


If we have plaintext credentials for the target user, it will be no problem to RDP into the system. However, what if we only have the NT hash of the user obtained from a credential dumping attack such as [SAM](https://en.wikipedia.org/wiki/Security_Account_Manager) database, and we could not crack the hash to reveal the plaintext password? In some instances, we can perform an RDP PtH attack to gain GUI access to the target system using tools like `xfreerdp`.

There are a few caveats to this attack:

- `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, we will be prompted with the following error:

![[rdp_session-4.png]]

This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa`. It can be done using the following command:

#### Adding the DisableRestrictedAdmin Registry Key

	C:\htb> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f

Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access:

	AstraX01@htb[/htb]# xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9

If it works, we'll now be logged in via RDP as the target user without knowing their cleartext password.

