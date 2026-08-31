`pam_unix.so` or `pam_unix2.so`, are typically located in `/usr/lib/x86_64-linux-gnu/security/` on Debian-based systems.

These modules manage user information, authentication, sessions, and password changes.

The `pam_unix.so` module uses standardized API calls from system libraries to update account information. The primary files it reads from and writes to are `/etc/passwd` and `/etc/shadow`.

## Passwd file

The `/etc/passwd` file contains information about every user on the system and is readable by all users and services.

ach entry in the file corresponds to a single user and consists of `seven fields`, which store user-related data in a structured format. These fields are separated by colons (`:`).

	htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash

|Field|Value|
|---|---|
|Username|`htb-student`|
|Password|`x`|
|User ID|`1000`|
|Group ID|`1000`|
|[GECOS](https://en.wikipedia.org/wiki/Gecos_field)|`,,,`|
|Home directory|`/home/htb-student`|
|Default shell|`/bin/bash`|

The most relevant field for our purposes is the `Password` field, as it can contain different types of entries.

**password hashes are stored in the `/etc/shadow` file**

we will find the value `x` in this field, indicating that the passwords are stored in a hashed form within the `/etc/shadow` file. However, it can also be that the `/etc/passwd` file is writeable by mistake. This would allow us to remove the password field for the `root` user entirely.

	AstraX01@htb[/htb]$ head -n 1 /etc/passwd
	
	root::0:0:root:/root:/bin/bash

This results in no password prompt being displayed when attempting to log in as `root`.

	AstraX01@htb[/htb]$ su

## Shadow file

if there is no entry in the `/etc/shadow` file for a user listed in `/etc/passwd`, that user is considered invalid.

The `/etc/shadow` file is also only readable by users with administrative privileges. The format of this file is divided into the following `nine fields`:

	htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::

If the `Password` field contains a character such as `!` or `*`, the user cannot log in using a Unix password. other authentication methods—such as Kerberos or key-based authentication—can still be used.

The `Password` field also follows a particular format, from which we can extract additional information:

- `$<id>$<salt>$<hashed>`

the hashed passwords are divided into three parts. The `ID` value specifies which cryptographic hash algorithm was used

|ID|Cryptographic Hash Algorithm|
|---|---|
|`1`|[MD5](https://en.wikipedia.org/wiki/MD5)|
|`2a`|[Blowfish](https://en.wikipedia.org/wiki/Blowfish_\(cipher\))|
|`5`|[SHA-256](https://en.wikipedia.org/wiki/SHA-2)|
|`6`|[SHA-512](https://en.wikipedia.org/wiki/SHA-2)|
|`sha1`|[SHA1crypt](https://en.wikipedia.org/wiki/SHA-1)|
|`y`|[Yescrypt](https://github.com/openwall/yescrypt)|
|`gy`|[Gost-yescrypt](https://www.openwall.com/lists/yescrypt/2019/06/30/1)|
|`7`|[Scrypt](https://en.wikipedia.org/wiki/Scrypt)|

## Opasswd

The PAM library (`pam_unix.so`) can prevent users from reusing old passwords.

These previous passwords are stored in the `/etc/security/opasswd` file. Administrator (root) privileges are required to read this file, assuming its permissions have not been modified manually.

	AstraX01@htb[/htb]$ sudo cat /etc/security/opasswd
	
	cry0l1t3:1000:2:$1$HjFAfYTG$qNDkF0zJ3v8ylCOrKB0kt0,$1$kcUjWZJX$E9uMSmiQeRh4pAAgzuvkq1

Looking at the contents of this file, we can see that it contains several entries for the user `cry0l1t3`, separated by a comma (`,`). One critical detail to pay attention to is the type of hash that's been used. This is because the `MD5` (`$1$`) algorithm is significantly easier to crack than SHA-512.

## Cracking Linux Credentials

Once we have root access on a Linux machine, we can gather user password hashes and attempt to crack them using various methods to recover the plaintext passwords.

To do this, we can use a tool called [unshadow](https://github.com/pmittaldev/john-the-ripper/blob/master/src/unshadow.c), which is included with John the Ripper (JtR). It works by combining the `passwd` and `shadow` files into a single file suitable for cracking.

	AstraX01@htb[/htb]$ sudo cp /etc/passwd /tmp/passwd.bak 
	AstraX01@htb[/htb]$ sudo cp /etc/shadow /tmp/shadow.bak 
	AstraX01@htb[/htb]$ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes

This "unshadowed" file can now be attacked with either JtR or hashcat.

	AstraX01@htb[/htb]$ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked

