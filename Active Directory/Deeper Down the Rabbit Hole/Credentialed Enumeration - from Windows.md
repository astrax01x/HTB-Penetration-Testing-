
we learn few tools for enumerating from a Windows attack host, such as SharpHound/BloodHound, PowerView/SharpView, Grouper2, Snaffler, and some built-in tools useful for AD enumeration. 

## ActiveDirectory PowerShell Module

The ActiveDirectory PowerShell module is a group of PowerShell cmdlets for administering an Active Directory environment from the command line.

#### Discover Modules

	PS C:\htb> Get-Module

#### Load ActiveDirectory Module

	PS C:\htb> Import-Module ActiveDirectory 
	PS C:\htb> Get-Module

### Get Domain Info

First up, we'll enumerate some basic information about the domain with the [Get-ADDomain](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-addomain?view=windowsserver2022-ps) cmdlet.

	PS C:\htb> Get-ADDomain

This will print out helpful information like the domain SID, domain functional level, any child domains.

#### Get-ADUser

	PS C:\htb> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

#### Checking For Trust Relationships

Another interesting check we can run utilizing the ActiveDirectory module, would be to verify domain trust relationships using the [Get-ADTrust](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adtrust?view=windowsserver2022-ps) cmdlet

	PS C:\htb> Get-ADTrust -Filter *

This will be useful later on when looking to take advantage of child-to-parent trust relationships and attacking across forest trusts.

#### Group Enumeration

we can gather AD group information using the [Get-ADGroup](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adgroup?view=windowsserver2022-ps) cmdlet.

	PS C:\htb> Get-ADGroup -Filter * | select name

#### Detailed Group Info

	PS C:\htb> Get-ADGroup -Identity "Backup Operators"

Now that we know more about the group, let's get a member listing using the [Get-ADGroupMember](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adgroupmember?view=windowsserver2022-ps) cmdlet.

### Group Membership

	PS C:\htb> Get-ADGroupMember -Identity "Backup Operators"

We can see that one account, `backupagent`, belongs to this group. It is worth noting this down because if we can take over this service account through some attack, we could use its membership in the Backup Operators group to take over the domain.

---
## PowerView

Let's examine some of PowerView's capabilities and see what data it returns. The table below describes some of the most useful functions PowerView offers.

|**Command**|**Description**|
|---|---|
|`Export-PowerViewCSV`|Append results to a CSV file|
|`ConvertTo-SID`|Convert a User or group name to its SID value|
|`Get-DomainSPNTicket`|Requests the Kerberos ticket for a specified Service Principal Name (SPN) account|
|**Domain/LDAP Functions:**||
|`Get-Domain`|Will return the AD object for the current (or specified) domain|
|`Get-DomainController`|Return a list of the Domain Controllers for the specified domain|
|`Get-DomainUser`|Will return all users or specific user objects in AD|
|`Get-DomainComputer`|Will return all computers or specific computer objects in AD|
|`Get-DomainGroup`|Will return all groups or specific group objects in AD|
|`Get-DomainOU`|Search for all or specific OU objects in AD|
|`Find-InterestingDomainAcl`|Finds object ACLs in the domain with modification rights set to non-built in objects|
|`Get-DomainGroupMember`|Will return the members of a specific domain group|
|`Get-DomainFileServer`|Returns a list of servers likely functioning as file servers|
|`Get-DomainDFSShare`|Returns a list of all distributed file systems for the current (or specified) domain|
|**GPO Functions:**||
|`Get-DomainGPO`|Will return all GPOs or specific GPO objects in AD|
|`Get-DomainPolicy`|Returns the default domain policy or the domain controller policy for the current domain|
|**Computer Enumeration Functions:**||
|`Get-NetLocalGroup`|Enumerates local groups on the local or a remote machine|
|`Get-NetLocalGroupMember`|Enumerates members of a specific local group|
|`Get-NetShare`|Returns open shares on the local (or a remote) machine|
|`Get-NetSession`|Will return session information for the local (or a remote) machine|
|`Test-AdminAccess`|Tests if the current user has administrative access to the local (or a remote) machine|
|**Threaded 'Meta'-Functions:**||
|`Find-DomainUserLocation`|Finds machines where specific users are logged in|
|`Find-DomainShare`|Finds reachable shares on domain machines|
|`Find-InterestingDomainShareFile`|Searches for files matching specific criteria on readable shares in the domain|
|`Find-LocalAdminAccess`|Find machines on the local domain where the current user has local administrator access|
|**Domain Trust Functions:**||
|`Get-DomainTrust`|Returns domain trusts for the current domain or a specified domain|
|`Get-ForestTrust`|Returns all forest trusts for the current forest or a specified forest|
|`Get-DomainForeignUser`|Enumerates users who are in groups outside of the user's domain|
|`Get-DomainForeignGroupMember`|Enumerates groups with users outside of the group's domain and returns each foreign member|
|`Get-DomainTrustMapping`|Will enumerate all trusts for the current domain and any others seen.|
We will learn a few of them . 

#### Domain User Information

	PS C:\Windows\system32> cd C:\Tools\ 
	PS C:\Tools> Import-Module .\PowerView.ps1 
	PS C:\htb> Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

#### Recursive Group Membership

	PS C:\htb> Get-DomainGroupMember -Identity "Domain Admins" -Recurse

Above we performed a recursive look at the `Domain Admins` group to list its members. Now we know who to target for potential elevation of privileges. Like with the AD PowerShell module, we can also enumerate domain trust mappings.

#### Trust Enumeration

	PS C:\htb> Get-DomainTrustMapping

We can use the [Test-AdminAccess](https://powersploit.readthedocs.io/en/latest/Recon/Test-AdminAccess/) function to test for local admin access on either the current machine or a remote one.

#### Testing for Local Admin Access

	PS C:\htb> Test-AdminAccess -ComputerName ACADEMY-EA-MS01

Now we can check for users with the SPN attribute set, which indicates that the account may be subjected to a Kerberoasting attack.

#### Finding Users With SPN Set

	PS C:\htb> Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName

## SharpView

Another tool worth experimenting with is SharpView, a .NET port of PowerView. Many of the same functions supported by PowerView can be used with SharpView.

Here we can use SharpView to enumerate information about a specific user, such as the user `forend`, which we control.

	PS C:\htb> .\SharpView.exe Get-DomainUser -Identity forend


---

## Snaffler

[Snaffler](https://github.com/SnaffCon/Snaffler) is a tool that can help us acquire credentials or other sensitive data in an Active Directory environment. Snaffler works by obtaining a list of hosts within the domain and then enumerating those hosts for shares and readable directories.

#### Snaffler Execution

	Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data


---

### BloodHound

First, we must authenticate as a domain user from a Windows attack host positioned within the network (but not joined to the domain) or transfer the tool to a domain-joined host.

#### SharpHound

We'll start by running the SharpHound.exe collector from the MS01 attack host.

	PS C:\htb> .\SharpHound.exe -c All --zipfilename ILFREIGHT

