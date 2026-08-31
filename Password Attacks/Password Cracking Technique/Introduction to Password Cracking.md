Passwords are commonly `hashed` when stored, in order to provide some protection in the event they fall into the hands of an attacker. `Hashing` is a mathematical function which transforms an arbitrary number of input bytes into a (typically) fixed-size output; common examples of hash functions are `MD5`, and `SHA-256`.

Take the password `Soccer06!` for example. The corresponding `MD5` and `SHA-256` hashes can be generated with the following commands:

	echo -n Soccer06! | md5sum 40291c1d19ee11a7df8495c4cccefdfa -

	echo -n Soccer06! | sha256sum a025dc6fabb09c2b8bfe23b5944635f9b68433ebd9a1a09453dd4fee00766d93 -

This means it should not be possible to figure out what the original password was based on the hash alone. When attackers attempt to do this, it is called `password cracking`. Common techniques are to use `rainbow tables`, to perform `dictionary attacks`, and typically as a last resort, to perform `brute-force attacks`.

## Rainbow tables

Rainbow tables are large pre-compiled maps of input and output values for a given hash function.

Because rainbow tables are such a powerful attack, `salting` is used. A `salt`, in cryptographic terms, is a random sequence of bytes added to a password before it is hashed. To maximize impact, salts should not be reused, e.g. for all passwords stored in one database. For example, if the salt `Th1sIsTh3S@lt_` is prepended to the same password, the MD5 hash would now be as follows:

	AstraX01@htb[/htb]$ echo -n Th1sIsTh3S@lt_Soccer06! | md5sum
	
	90a10ba83c04e7996bc53373170b5474  -

A salt is not a secret value — when a system goes to check an authentication request, it needs to know what salt was used so that it can check if the password hash matches

# Authentication ke time salt kyun pata hona chahiye?

Ye paragraph bolta hai:

> system ko salt pata hona chahiye taaki authentication check kar sake.

*Salt password ke saath add ki jaane wali random value hai jo hash ko unique banati hai; salt secret nahi hota, aur even a 1-byte salt rainbow-table ke required precomputed combinations ko 256× badha deta hai.*

## Brute-force attack

A `brute-force` attack involves attempting every possible combination of letters, numbers, and symbols until the correct password is discovered.

Brute-forcing is the only password cracking technique that is `100% effective` - in that, given enough time, any password will be cracked with this technique.

***Note:** Brute-forcing speeds depend heavily on the hashing algorithm and hardware that is used. On a typical company laptop, a tool like `hashcat` might be able to guess over `five million` passwords per second when attacking MD5, while at the same time only managing `ten thousand` per second when targeting a DCC2 hash.*

## Dictionary attack

## Dictionary attack

A `dictionary` attack, otherwise known as a `wordlist` attack, is one of the most `efficient` techniques for cracking passwords, especially when operating under time-constraints

Rather than attempting every possible combination of characters, a list containing statistically likely passwords is used. Well-known wordlists for password cracking are [rockyou.txt](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt) and those included in [SecLists](https://github.com/danielmiessler/SecLists).

	AstraX01@htb[/htb]$ head --lines=20 /usr/share/wordlists/rockyou.txt

