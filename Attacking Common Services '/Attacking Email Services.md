
## Enumeration

We can use the `Mail eXchanger` (`MX`) DNS record to identify a mail server. The MX record specifies the mail server responsible for accepting email messages on behalf of a domain name. It is possible to configure several MX records, typically pointing to an array of mail servers for load balancing and redundancy.

We can use tools such as `host` or `dig` and online websites such as [MXToolbox](https://mxtoolbox.com/) to query information about the MX records:

#### Host - MX Records

	AstraX01@htb[/htb]$ host -t MX hackthebox.eu 
	
	hackthebox.eu mail is handled by 1 aspmx.l.google.com.

	AstraX01@htb[/htb]$ host -t MX microsoft.com  
	
	microsoft.com mail is handled by 10 microsoft-com.mail.protection.outlook.com.

#### DIG - MX Records

	AstraX01@htb[/htb]$ dig mx plaintext.do | grep "MX" | grep -v ";"

	AstraX01@htb[/htb]$ dig mx inlanefreight.com | grep "MX" | grep -v ";" 
	
	inlanefreight.com.    300    IN    MX     10 mail1.inlanefreight.com.

#### Host - A Records

	AstraX01@htb[/htb]$ host -t A mail1.inlanefreight.htb. 
	
	mail1.inlanefreight.htb has address 10.129.14.128

These `MX` records indicate that the first three mail services are using a cloud services G-Suite (aspmx.l.google.com), Microsoft 365 (microsoft-com.mail.protection.outlook.com), and Zoho (mx.zoho.com), and the last one may be a custom mail server hosted by the company.


If we are targetting a custom mail server implementation such as `inlanefreight.htb`, we can enumerate the following ports:

|**Port**|**Service**|
|---|---|
|`TCP/25`|SMTP Unencrypted|
|`TCP/143`|IMAP4 Unencrypted|
|`TCP/110`|POP3 Unencrypted|
|`TCP/465`|SMTP Encrypted|
|`TCP/587`|SMTP Encrypted/[STARTTLS](https://en.wikipedia.org/wiki/Opportunistic_TLS)|
|`TCP/993`|IMAP4 Encrypted|
|`TCP/995`|POP3 Encrypted|

We can use `Nmap`'s default script `-sC` option to enumerate those ports on the target system:

	AstraX01@htb[/htb]$ sudo nmap -Pn -sV -sC -p25,143,110,465,587,993,995 10.129.14.128

## Misconfigurations

Email services use authentication to allow users to send emails and receive emails. A misconfiguration can happen when the SMTP service allows anonymous authentication or support protocols that can be used to enumerate valid usernames.

#### Authentication

The SMTP server has different commands that can be used to enumerate valid usernames `VRFY`, `EXPN`, and `RCPT TO`. If we successfully enumerate valid usernames, we can attempt to password spray, brute-forcing, or guess a valid password. So let's explore how those commands work.

`VRFY` this command instructs the receiving SMTP server to check the validity of a particular email username. The server will respond, indicating if the user exists or not. This feature can be disabled.

#### VRFY Command

	AstraX01@htb[/htb]$ telnet 10.10.110.20 25 
	
	Trying 10.10.110.20... 
	Connected to 10.10.110.20. 
	Escape character is '^]'. 
	220 parrot ESMTP Postfix (Debian/GNU) 
	
	VRFY root 
	
	252 2.0.0 root 
	
	VRFY www-data 
	
	252 2.0.0 www-data 
	
	VRFY new-user 
	
	550 5.1.1 <new-user>: Recipient address rejected: User unknown in local recipient table

`EXPN` is similar to `VRFY`, except that when used with a distribution list, it will list all users on that list. This can be a bigger problem than the `VRFY` command since sites often have an alias such as "all."

#### EXPN Command

	AstraX01@htb[/htb]$ telnet 10.10.110.20 25 
	
	Trying 10.10.110.20... 
	Connected to 10.10.110.20. 
	Escape character is '^]'. 
	220 parrot ESMTP Postfix (Debian/GNU) 
	
	EXPN john 
	
	250 2.1.0 john@inlanefreight.htb 
	
	EXPN support-team 
	
	250 2.0.0 carol@inlanefreight.htb 
	250 2.1.5 elisa@inlanefreight.htb

`RCPT TO` identifies the recipient of the email message. This command can be repeated multiple times for a given message to deliver a single message to multiple recipients.

#### RCPT TO Command

	AstraX01@htb[/htb]$ telnet 10.10.110.20 25 
	
	Trying 10.10.110.20... 
	Connected to 10.10.110.20. 
	Escape character is '^]'. 
	220 parrot ESMTP Postfix (Debian/GNU) 
	
	MAIL FROM:test@htb.com 
	it is 
	250 2.1.0 test@htb.com... Sender ok 
	
	RCPT TO:julio 
	
	550 5.1.1 julio... 
	
	User unknown RCPT TO:kate 
	
	550 5.1.1 kate... User unknown 
	
	RCPT TO:john 2
	50 2.1.5 john... Recipient ok

#### POP3

We can also use the `POP3` protocol to enumerate users depending on the service implementation. For example, we can use the command `USER` followed by the username, and if the server responds `OK`. This means that the user exists on the server.

#### USER Command

	AstraX01@htb[/htb]$ telnet 10.10.110.20 110 
	Trying 10.10.110.20... 
	Connected to 10.10.110.20. 
	Escape character is '^]'. 
	+OK POP3 Server ready 
	
	USER julio 
	
	-ERR 
	
	USER john 
	
	+OK

#### SMTP-USER-ENUM

To automate our enumeration process, we can use a tool named [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum).

We can specify the enumeration mode with the argument `-M` followed by `VRFY`, `EXPN`, or `RCPT`, and the argument `-U` with a file containing the list of users we want to enumerate. Depending on the server implementation and enumeration mode, we need to add the domain for the email address with the argument `-D`. Finally, we specify the target with the argument `-t`.

	AstraX01@htb[/htb]$ smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7

## Cloud Enumeration

As discussed, cloud service providers use their own implementation for email services. Those services commonly have custom features that we can abuse for operation, such as username enumeration. Let's use Office 365 as an example and explore how we can enumerate usernames in this cloud platform.

#### O365 Spray

 first validate if our target domain is using Office 365.

	AstraX01@htb[/htb]$ python3 o365spray.py --validate --domain msplaintext.xyz

Now, we can attempt to identify usernames.

	AstraX01@htb[/htb]$ python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz

## Password Attacks

We can use `Hydra` to perform a password spray or brute force against email services such as `SMTP`, `POP3`, or `IMAP4`. First, we need to get a username list and a password list and specify which service we want to attack. Let us see an example for `POP3`.

#### Hydra - Password Attack

	AstraX01@htb[/htb]$ hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3

If cloud services support SMTP, POP3, or IMAP4 protocols, we may be able to attempt to perform password spray using tools like `Hydra`, but these tools are usually blocked.

We can instead try to use custom tools such as [o365spray](https://github.com/0xZDH/o365spray) or [MailSniper](https://github.com/dafthack/MailSniper) for Microsoft Office 365 or [CredKing](https://github.com/ustayready/CredKing) for Gmail or Okta.

#### O365 Spray - Password Spraying

	AstraX01@htb[/htb]$ python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz

## Protocol Specifics Attacks

An open relay is a Simple Mail Transfer Protocol (`SMTP`) server, which is improperly configured and allows an unauthenticated email relay.

#### Open Relay

An **Open Relay** is a misconfigured SMTP server that allows anyone on the internet to send emails through it **without authentication**.

- This means attackers can send emails using **any "From" address** (spoofing).

#### Example Misconfiguration

	mynetworks = 0.0.0.0/0

This configuration tells the server to **trust all IP addresses** and skip sender validation. As a result, anyone can connect and send emails from arbitrary addresses.

## From an Attacker Point-of-View

	AstraX01@htb[/htb]# nmap -p25 -Pn --script smtp-open-relay 10.10.11.213

Next, we can use any mail client to connect to the mail server and send our email.

	AstraX01@htb[/htb]# swaks --from notifications@inlanefreight.com --to employees@inlanefreight.com --header


