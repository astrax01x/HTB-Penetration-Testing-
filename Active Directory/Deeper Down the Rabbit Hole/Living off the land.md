## Env Commands For Host & Network Recon 

First, we'll cover a few basic environmental commands that can be used to give us more information about the host we are on.

#### Basic Enumeration Commands

| **Command**                                             | **Result**                                                                                 |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `hostname`                                              | Prints the PC's Name                                                                       |
| `[System.Environment]::OSVersion.Version`               | Prints out the OS version and revision level                                               |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Prints the patches and hotfixes applied to the host                                        |
| `ipconfig /all`                                         | Prints out network adapter state and configurations                                        |
| `set`                                                   | Displays a list of environment variables for the current session (ran from CMD-prompt)     |
| `echo %USERDOMAIN%`                                     | Displays the domain name to which the host belongs (ran from CMD-prompt)                   |
| `echo %logonserver%`                                    | Prints out the name of the Domain controller the host checks in with (ran from CMD-prompt) |
## Harnessing PowerShell

Let's look at a few of the ways PowerShell can help us

#### Quick Checks Using PowerShell

	PS C:\htb> Get-Module

	PS C:\htb> Get-ExecutionPolicy -List

	PS C:\htb> whoami 

	PS C:\htb> Get-ChildItem Env: | ft key,value

### Checking Defenses

The next few commands utilize the [netsh](https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts) and [sc](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-query) utilities to help us get a feel for the state of the host when it comes to Windows Firewall settings and to check the status of Windows Defender.

#### Firewall Checks

	PS C:\htb> netsh advfirewall show allprofiles

#### Windows Defender Check (from CMD.exe)

	C:\htb> sc query windefend

We checked if Defender was running . we will check the status and configuration settings with the [Get-MpComputerStatus](https://docs.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus?view=windowsserver2022-ps) cmdlet in PowerShell. 

#### Get-MpComputerStatus

	PS C:\htb> Get-MpComputerStatus

## Am I Alone?

When landing on a host for the first time, one important thing is to check and see if you are the only one logged in. If you start taking actions from a host someone else is on, there is the potential for them to notice you. If a popup window launches or a user is logged out of their session, they may report these actions or change their password, and we could lose our foothold.

#### Using qwinsta

	PS C:\htb> qwinsta

---

## Network Information

Commands such as `ipconfig /all` and `systeminfo` show us some basic networking configurations.

`arp -a` and `route print` will show us what hosts the box we are on is aware of and what networks are known to the host.These two commands can be especially helpful in the discovery phase of a black box assessment where we have to limit our scanning

#### Using arp -a

	PS C:\htb> arp -a

#### Viewing the Routing Table

	PS C:\htb> route print

---

## Windows Management Instrumentation (WMI)


[Windows Management Instrumentation (WMI)](https://docs.microsoft.com/en-us/windows/win32/wmisdk/about-wmi) is a scripting engine that is widely used within Windows enterprise environments to retrieve information and run administrative tasks on local and remote hosts.

#### Quick WMI checks

| **Command**                                                                          | **Description**                                                                                        |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn`                              | Prints the patch level and description of the Hotfixes applied                                         |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List` | Displays basic host information to include any attributes within the list                              |
| `wmic process list /format:list`                                                     | A listing of all processes on host                                                                     |
| `wmic ntdomain list /format:list`                                                    | Displays information about the Domain and Domain Controllers                                           |
| `wmic useraccount list /format:list`                                                 | Displays information about all local accounts and any domain accounts that have logged into the device |
| `wmic group list /format:list`                                                       | Information about all local groups                                                                     |
| `wmic sysaccount list /format:list`                                                  | Dumps information about any system accounts that are being used as service accounts.                   |
This [cheatsheet](https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4) has some useful commands for querying host and domain info using wmic.

	PS C:\htb> wmic ntdomain get Caption,Description,DnsForestName,DomainName,DomainControllerAddress

---

## Net Commands

[Net](https://docs.microsoft.com/en-us/windows/win32/winsock/net-exe-2) commands can be beneficial to us when attempting to enumerate information from the domain.

#### Listing Domain Groups

	PS C:\htb> net group /domain

#### Information about a Domain User

	PS C:\htb> net user /domain wrouse

