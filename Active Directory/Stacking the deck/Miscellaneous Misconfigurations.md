There are many other misconfigurations that can be used to further your access within a domain.

### Password in Description Field

Sensitive information such as account passwords are sometimes found in the user account `Description` or `Notes` fields and can be quickly enumerated using PowerView.

#### Finding Passwords in the Description Field using Get-Domain User

	PS C:\htb> Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}


---
## PASSWD_NOTREQD Field

It is possible to come across domain accounts with the [passwd_notreqd](https://ldapwiki.com/wiki/Wiki.jsp?page=PASSWD_NOTREQD) field set in the userAccountControl attribute. If this is set, the user is not subject to the current password policy length, meaning they could have a shorter password or no password at all (if empty passwords are allowed in the domain).

#### Checking for PASSWD_NOTREQD Setting using Get-DomainUser

	PS C:\htb> Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol


---
## Group Policy Preferences (GPP) Passwords

When a new GPP is created, an .xml file is created in the SYSVOL share, which is also cached locally on endpoints that the Group Policy applies to. These files can include those used to:

- Map drives (drives.xml)
- Create local users
- Create printer config files (printers.xml)
- Creating and updating services (services.xml)
- Creating scheduled tasks (scheduledtasks.xml)
- Changing local admin passwords.
#### Viewing Groups.xml
![[GPP.png]]

If you retrieve the cpassword value more manually, the `gpp-decrypt` utility can be used to decrypt the password as follows:

#### Decrypting the Password with gpp-decrypt

	AstraX01@htb[/htb]$ gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE


---
## ASREPRoasting

t's possible to obtain the Ticket Granting Ticket (TGT) for any account that has the [Do not require Kerberos pre-authentication](https://www.tenable.com/blog/how-to-stop-the-kerberos-pre-authentication-attack-in-active-directory) setting enabled. Many vendor installation guides specify that their service account be configured in this way. The authentication service reply (AS_REP) is encrypted with the account’s password, and any domain user can request it.

ASREPRoasting is similar to Kerberoasting, but it involves attacking the AS-REP instead of the TGS-REP. An SPN is not required.

#### Enumerating for DONT_REQ_PREAUTH Value using Get-DomainUser

	PS C:\htb> Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl

#### Retrieving AS-REP in Proper Format using Rubeus

	PS C:\htb> .\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat

We can then crack the hash offline using Hashcat with mode `18200`.

#### Cracking the Hash Offline with Hashcat

	AstraX01@htb[/htb]$ hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt

When performing user enumeration with `Kerbrute`, the tool will automatically retrieve the AS-REP for any users found that do not require Kerberos pre-authentication.

#### Retrieving the AS-REP Using Kerbrute

	AstraX01@htb[/htb]$ kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt

With a list of valid users, we can use [Get-NPUsers.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py) from the Impacket toolkit to hunt for all users with Kerberos pre-authentication not required.

#### Hunting for Users with Kerberos Pre-auth Not Required

	AstraX01@htb[/htb]$ GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users

---
## Group Policy Object (GPO) Abuse

Group Policy provides administrators with many advanced settings that can be applied to both user and computer objects in an AD environment.

