The `Simple Mail Transfer Protocol` (`SMTP`) is a protocol for sending emails in an IP network. It can be used between an email client and an outgoing mail server or between two SMTP servers. SMTP is often combined with the IMAP or POP3 protocols, which can fetch emails and send emails. In principle, it is a client-server-based protocol, although SMTP can be used between a client and a server and between two SMTP servers.

MTP works unencrypted without further measures and transmits all commands, data, or authentication information in plain text. To prevent unauthorized reading of data, the SMTP is used in conjunction with SSL/TLS encryption. Under certain circumstances, a server uses a port other than the standard TCP port `25` for the encrypted connection, for example, TCP port `465`.

An essential function of an SMTP server is preventing spam using authentication mechanisms that allow only authorized users to send e-mails. For this purpose, most modern SMTP servers support the protocol extension ESMTP with SMTP-Auth. After sending his e-mail, the SMTP client, also known as `Mail User Agent` (`MUA`), converts it into a header and a body and uploads both to the SMTP server. This has a so-called `Mail Transfer Agent` (`MTA`), the software basis for sending and receiving e-mails. The MTA checks the e-mail for size and spam and then stores it. To relieve the MTA, it is occasionally preceded by a `Mail Submission Agent` (`MSA`), which checks the validity, i.e., the origin of the e-mail. This `MSA` is also called `Relay` server. These are very important later on, as the so-called `Open Relay Attack` can be carried out on many SMTP servers due to incorrect configuration.


*To interact with the SMTP server, we can use the `telnet` tool to initialize a TCP connection with the SMTP server. The actual initialization of the session is done with the command mentioned above, `HELO` or `EHLO`.*

#### Telnet - HELO/EHLO

	AstraX01@htb[/htb]$ telnet 10.129.14.128 25

The command `VRFY` can be used to enumerate existing users on the system. However, this does not always work. Depending on how the SMTP server is configured, the SMTP server may issue `code 252` and confirm the existence of a user that does not exist on the system.

#### Telnet - VRFY

	AstraX01@htb[/htb]$ telnet 10.129.14.128 25 
	Trying 10.129.14.128... 
	Connected to 10.129.14.128. 
	Escape character is '^]'. 
	220 ESMTP Server 
	
	VRFY root 
	
	252 2.0.0 root 
	
	VRFY cry0l1t3 
	
	252 2.0.0 cry0l1t3 
	
	VRFY testuser 
	
	252 2.0.0 testuser 
	
	VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa 
	252 2.0.0 aaaaaaaaaaaaaaaaaaaaaaaaaaaa

#### Send an Email

	AstraX01@htb[/htb]$ telnet 10.129.14.128 25
	Trying 10.129.14.128... 
	Connected to 10.129.14.128. 
	Escape character is '^]'.
	220 ESMTP Server 
	
	EHLO inlanefreight.htb 
	
	250-mail1.inlanefreight.htb 
	250-PIPELINING 
	250-SIZE 10240000 
	250-ETRN 
	250-ENHANCEDSTATUSCODES 
	250-8BITMIME 
	250-DSN 
	250-SMTPUTF8 
	250 CHUNKING 
	
	MAIL FROM: <cry0l1t3@inlanefreight.htb> 
	250 2.1.0 Ok 
	
	RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure 
	250 2.1.5 Ok 
	
	DATA 
	
	354 End data with <CR><LF>.<CR><LF> 
	
	From: <cry0l1t3@inlanefreight.htb> 
	To: <mrb3n@inlanefreight.htb> 
	Subject: DB 
	Date: Tue, 28 Sept 2021 16:32:51 +0200 
	Hey man, I am trying to access our XY-DB but the creds don't work. 
	Did you make any changes there?
	 .
	  250 2.0.0 Ok: queued as 6E1CF1681AB 
	  QUIT 
	  221 2.0.0 Bye 
	  Connection closed by foreign host.

## Footprinting the Service

#### Nmap

	AstraX01@htb[/htb]$ sudo nmap 10.129.14.128 -sC -sV -p25

However, we can also use the [smtp-open-relay](https://nmap.org/nsedoc/scripts/smtp-open-relay.html) NSE script to identify the target SMTP server as an open relay using 16 different tests
#### Nmap - Open Relay

	AstraX01@htb[/htb]$ sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v