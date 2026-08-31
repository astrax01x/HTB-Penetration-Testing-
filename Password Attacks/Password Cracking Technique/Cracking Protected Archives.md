Besides standalone files, we will often run across `archives` and `compressed files`—such as ZIP files—which are protected with a password.

Let us assume the role of an employee at an administrative company and imagine that a client requests a summary of an analysis in various formats, such as Excel, PDF, Word, and a corresponding presentation. One approach would be to send these files individually. However, if we extend this scenario to a large organization managing multiple simultaneous projects, this method of file transfer can become cumbersome and may result in individual files being misplaced. In such cases, employees often rely on archive files, which allow them to organize necessary documents in a structured manner (typically using subfolders) before compressing them into a single, consolidated file.

There are many types of archive files. Some of the more commonly encountered file extensions include `tar`, `gz`, `rar`, `zip`, `vmdb/vmx`, `cpt`, `truecrypt`, `bitlocker`, `kdbx`, `deb`, `7z`, and `gzip`.

A comprehensive list of archive file types can be found on [FileInfo](https://fileinfo.com/filetypes/compressed). Rather than typing them out manually, we can also query the data using a one-liner, apply filters as needed, and save the results to a file. At the time of writing, the website lists `365` archive file types.

	AstraX01@htb[/htb]$ curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt

Note that not all archive types support native password protection, and in such cases, additional tools are often used to encrypt the files. For example, TAR files are commonly encrypted using `openssl` or `gpg`.

iven the wide variety of archive formats and encryption tools, this section will focus only on a selection of methods for cracking specific archive types. For password-protected archives, we typically require specialized scripts to extract password hashes from the files, which can then be used in offline cracking attempts.

## Cracking ZIP files

The `ZIP` format is often heavily used in Windows environments to compress many files into one file.

The process of cracking an encrypted ZIP file is similar to what we have seen already, except for using a different script to extract the hashes.

	AstraX01@htb[/htb]$ zip2john ZIP.zip > zip.hash 
	AstraX01@htb[/htb]$ cat zip.hash

Once we have extracted the hash, we can use JtR to crack it with the desired password list.

	AstraX01@htb[/htb]$ john --wordlist=rockyou.txt zip.hash

	AstraX01@htb[/htb]$ john zip.hash --show

## Cracking OpenSSL encrypted GZIP files

It is not always immediately apparent whether a file is password-protected, particularly when the file extension corresponds to a format that does not natively support password protection.

**`openssl` can be used to encrypt files in the `GZIP` format.**

To determine the actual format of a file, we can use the `file` command, which provides detailed information about its contents. For example:

	AstraX01@htb[/htb]$ file GZIP.gzip 
	
	GZIP.gzip: openssl enc'd data with salted password

When cracking OpenSSL encrypted files, we may encounter various challenges, including numerous false positives or complete failure to identify the correct password.

*To mitigate this, a more reliable approach is to use the `openssl` tool within a `for` loop that attempts to extract the contents directly, succeeding only if the correct password is found.*

The following one-liner may produce several GZIP-related error messages, which can be safely ignored. If the correct password list is used, as in this example, we will see another file successfully extracted from the archive.

	AstraX01@htb[/htb]$ for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done

Once the `for` loop has finished, we can check the current directory for a newly extracted file.

	AstraX01@htb[/htb]$ ls

## Cracking BitLocker-encrypted drives

[BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-device-encryption-overview-windows-10) is a full-disk encryption feature developed by Microsoft for the Windows operating system.

It uses the `AES` encryption algorithm with either 128-bit or 256-bit key lengths.

If the password or PIN used for BitLocker is forgotten, decryption can still be performed using a recovery key—a 48-digit string generated during the setup process.

To crack a BitLocker encrypted drive, we can use a script called `bitlocker2john` to [four different hashes](https://openwall.info/wiki/john/OpenCL-BitLocker):

**The first two correspond to the BitLocker password, while the latter two represent the recovery key.**

Because the recovery key is very long and randomly generated, it is generally not practical to guess—unless partial knowledge is available. Therefore, we will focus on cracking the password using the first hash (`$bitlocker$0$...`).

	AstraX01@htb[/htb]$ bitlocker2john -i Backup.vhd > backup.hashes 
	AstraX01@htb[/htb]$ grep "bitlocker\$0" backup.hashes > backup.hash 
	AstraX01@htb[/htb]$ cat backup.hash

	$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f

Once a hash is generated, either `JtR` or `hashcat` can be used to crack it. For this example, we will look at the procedure with `hashcat`. **The hashcat mode associated with the `$bitlocker$0$...` hash is `-m 22100`.**

We supply the hash, specify the wordlist, and define the hash mode. Since this encryption uses strong AES encryption, cracking may take considerable time depending on hardware performance.

	AstraX01@htb[/htb]$ hashcat -a 0 -m 22100 '$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f' /usr/share/wordlists/rockyou.txt

After successfully cracking the password, we can access the encrypted drive.

#### Mounting BitLocker-encrypted drives in Windows

**The easiest method for mounting a BitLocker-encrypted virtual drive on Windows is to double-click the `.vhd` file.**

Since it is encrypted, Windows will initially show an error. After mounting, simply double-click the BitLocker volume to be prompted for the password.

#### Mounting BitLocker-encrypted drives in Linux (or macOS)

It is also possible to mount BitLocker-encrypted drives in Linux (or macOS). To do this, we can use a tool called [dislocker](https://github.com/Aorimn/dislocker). First, we need to install the package using `apt`

	AstraX01@htb[/htb]$ sudo apt-get install dislocker

Next, we create two folders which we will use to mount the VHD.

	AstraX01@htb[/htb]$ sudo mkdir -p /media/bitlocker 
	AstraX01@htb[/htb]$ sudo mkdir -p /media/bitlockermount

We then use `losetup` to configure the VHD as [loop device](https://en.wikipedia.org/wiki/Loop_device), decrypt the drive using `dislocker`, and finally mount the decrypted volume:

	AstraX01@htb[/htb]$ sudo losetup -f -P Backup.vhd 
	AstraX01@htb[/htb]$ sudo dislocker /dev/loop0p2 -u1234qwer -- /media/bitlocker 
	AstraX01@htb[/htb]$ sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount

If everything was done correctly, we can now browse the files:

	AstraX01@htb[/htb]$ cd /media/bitlockermount/ 
	AstraX01@htb[/htb]$ ls -la

Once we have analyzed the files on the mounted drive, we can unmount it using the following commands:

	AstraX01@htb[/htb]$ sudo umount /media/bitlockermount 
	AstraX01@htb[/htb]$ sudo umount /media/bitlocker

