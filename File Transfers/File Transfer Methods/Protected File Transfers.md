As penetration testers, we often gain access to highly sensitive data such as user lists, credentials (i.e., downloading the NTDS.dit file for offline password cracking), and enumeration data that can contain critical information about the organization's network infrastructure, and Active Directory (AD) environment, etc. Therefore, it is essential to encrypt this data or use encrypted data connections such as SSH, SFTP, and HTTPS. However, sometimes these options are not available to us, and a different approach is required.

Data leakage during a penetration test could have severe consequences for the penetration tester, their company, and the client. As information security professionals, we must act professionally and responsibly and take all measures to protect any data we encounter during an assessment.

## File Encryption on Windows

Many different methods can be used to encrypt files and information on Windows systems. One of the simplest methods is the [Invoke-AESEncryption.ps1](https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1) PowerShell script. This script is small and provides encryption of files and strings.

#### Invoke-AESEncryption.ps1

	.EXAMPLE 
	Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Text "Secret Text" 
	
	Description 
	----------- 
	Encrypts the string "Secret Test" and outputs a Base64 encoded ciphertext. 
	
	.EXAMPLE 
	Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Text "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs=" 
	
	Description 
	----------- 
	Decrypts the Base64 encoded string "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs=" and outputs plain text. 
	
	.EXAMPLE 
	Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Path file.bin 
	
	Description 
	----------- 
	Encrypts the file "file.bin" and outputs an encrypted file "file.bin.aes" 
	
	.EXAMPLE 
	Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Path file.bin.aes 
	
	Description 
	----------- 
	Decrypts the file "file.bin.aes" and outputs the decrypted file "file.bin"

#### Import Module Invoke-AESEncryption.ps1

	PS C:\htb> Import-Module .\Invoke-AESEncryption.ps1
After the script is imported, it can encrypt strings or files, as shown in the following examples. This command creates an encrypted file with the same name as the encrypted file but with the extension "`.aes`."

you can download `Invoke-AESEncryption.ps1` script from github 

#### File Encryption Example

	PS C:\htb> Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
	
	File encrypted to C:\htb\scan-results.txt.aes 
	PS C:\htb> ls

Using very `strong` and `unique` passwords for encryption for every company where a penetration test is performed is essential. This is to prevent sensitive files and information from being decrypted using one single password that may have been leaked and cracked by a third party.

---

## File Encryption on Linux

[OpenSSL](https://www.openssl.org/) is frequently included in Linux distributions, with sysadmins using it to generate security certificates, among other tasks. OpenSSL can be used to send files "nc style" to encrypt files.

To encrypt a file using `openssl` we can select different ciphers, see [OpenSSL man page](https://www.openssl.org/docs/man1.1.1/man1/openssl-enc.html). Let's use `-aes256` as an example. We can also override the default iterations counts with the option `-iter 100000` and add the option `-pbkdf2` to use the Password-Based Key Derivation Function 2 algorithm. When we hit enter, we'll need to provide a password.

#### Encrypting /etc/passwd with openssl

	AstraX01@htb[/htb]$ openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc

Remember to use a strong and unique password to avoid brute-force cracking attacks should an unauthorized party obtain the file. To decrypt the file, we can use the following command:

#### Decrypt passwd.enc with openssl

	AstraX01@htb[/htb]$ openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd

