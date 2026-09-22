## LLMNR & NBT-NS Primer

[Link-Local Multicast Name Resolution](https://datatracker.ietf.org/doc/html/rfc4795) (LLMNR) and [NetBIOS Name Service](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc940063\(v=technet.10\)?redirectedfrom=MSDN) (NBT-NS) are Microsoft Windows components that serve as alternate methods of host identification that can be used when DNS fails.

## TTPs 

*We are performing these actions to collect authentication information sent over the network in the form of NTLMv1 and NTLMv2 password hashes. As discussed in the [Introduction to Active Directory](https://academy.hackthebox.com/course/preview/introduction-to-active-directory) module, NTLMv1 and NTLMv2 are authentication protocols that utilize the LM or NT hash. We will then take the hash and attempt to crack them offline using tools such as [Hashcat](https://hashcat.net/hashcat/) or [John](https://www.openwall.com/john/) with the goal of obtaining the account's cleartext password to be used to gain an initial foothold or expand our access within the domain if we capture a password hash for an account with more privileges than an account that we currently possess .*



| Responder      | Responder is a purpose-built tool to poison LLMNR, NBT-NS, and MDNS, with many different functions.    |
| -------------- | ------------------------------------------------------------------------------------------------------ |
| **Inveigh**    | **Inveigh is a cross-platform MITM platform that can be used for spoofing and poisoning attacks.**     |
| **Metasploit** | **Metasploit has several built-in scanners and spoofing modules made to deal with poisoning attacks.** |
### Responder In Action

Responder is a relatively straightforward tool, but is extremely powerful and has many different functions. In the `Initial Enumeration` section earlier, we utilized Responder in Analysis (passive) mode. This means it listened for any resolution requests, but did not answer them or send out poisoned packets . . Let's look at some options available by typing `responder -h` into our console . 

	 responder -h 

#### Responder Logs

	 AstraX01@htb[/htb]$ ls

 #### Starting Responder with Default Settings

	 sudo responder -I ens224 

Once we are ready, we can pass these hashes to Hashcat using hash mode `5600` for NTLMv2 hashes that we typically obtain with Responder.

*We may at times obtain NTLMv1 hashes and other types of hashes and can consult the [Hashcat example hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) page to identify them and find the proper hash mode* 

 #### **Cracking an NTLMv2 Hash With Hashcat** 

	 AstraX01@htb[/htb]$ hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt

