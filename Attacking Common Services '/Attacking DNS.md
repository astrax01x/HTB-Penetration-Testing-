
## DNS Zone Transfer

A DNS zone is a portion of the DNS namespace that a specific organization or administrator manages. Since DNS comprises multiple DNS zones, DNS servers utilize DNS zone transfers to copy a portion of their database to another DNS server.
anyone can ask a DNS server for a copy of its zone information since DNS zone transfers do not require any authentication. In addition, the DNS service usually runs on a UDP port; however, when performing DNS zone transfer, it uses a TCP port for reliable data transmission.

An attacker could leverage this DNS zone transfer vulnerability to learn more about the target organization's DNS namespace, increasing the attack surface. For exploitation, we can use the `dig` utility with DNS query type `AXFR` option to dump the entire DNS namespaces from a vulnerable DNS server:

#### DIG - AXFR Zone Transfer

	AstraX01@htb[/htb]# dig AXFR @ns1.inlanefreight.htb inlanefreight.htb

Tools like [Fierce](https://github.com/mschwager/fierce) can also be used to enumerate all DNS servers of the root domain and scan for a DNS zone transfer:

#### Fierce for Zone Transfer 

	AstraX01@htb[/htb]# fierce --domain zonetransfer.me

## Domain Takeovers & Subdomain Enumeration

`Domain takeover` is registering a non-existent domain name to gain control over another domain. If attackers find an expired domain, they can claim that domain to perform further attacks such as hosting malicious content on a website or sending a phishing email leveraging the claimed domain.

Domain takeover is also possible with subdomains called `subdomain takeover`. A DNS's canonical name (`CNAME`) record is used to map different domains to a parent domain. Many organizations use third-party services like AWS, GitHub, Akamai, Fastly, and other content delivery networks (CDNs) to host their content. In this case, they usually create a subdomain and make it point to those services.
#### Subdomain Enumeration

##### Using Subfinder for Subdomain enumeration '

	AstraX01@htb[/htb]# ./subfinder -d inlanefreight.com -v

An excellent alternative is a tool called [Subbrute](https://github.com/TheRook/subbrute). This tool allows us to use self-defined resolvers and perform pure DNS brute-forcing attacks during internal penetration tests on hosts that do not have Internet access.

#### Subbrute

	AstraX01@htb[/htb]$ git clone https://github.com/TheRook/subbrute.git >> 
	/dev/null 2>&1 
	AstraX01@htb[/htb]$ cd subbrute 
	AstraX01@htb[/htb]$ echo "ns1.inlanefreight.com" > ./resolvers.txt 
	AstraX01@htb[/htb]$ ./subbrute.py inlanefreight.com -s ./names.txt -r 
	./resolvers.txt

The tool has found four subdomains associated with `inlanefreight.com`. Using the `nslookup` or `host` command, we can enumerate the `CNAME` records for those subdomains.

	AstraX01@htb[/htb]# host support.inlanefreight.com 
	
	support.inlanefreight.com is an alias for inlanefreight.s3.amazonaws.com

The `support` subdomain has an alias record pointing to an AWS S3 bucket. However, the URL `https://support.inlanefreight.com` shows a `NoSuchBucket` error indicating that the subdomain is potentially vulnerable to a subdomain takeover. Now, we can take over the subdomain by creating an AWS S3 bucket with the same subdomain name.

![[s3.png]]

## DNS Spoofing

DNS spoofing is also referred to as DNS Cache Poisoning. This attack involves altering legitimate DNS records with false information so that they can be used to redirect online traffic to a fraudulent website. Example attack paths for the DNS Cache Poisoning are as follows:

- An attacker could intercept the communication between a user and a DNS server to route the user to a fraudulent destination instead of a legitimate one by performing a Man-in-the-Middle (`MITM`) attack.
- Exploiting a vulnerability found in a DNS server could yield control over the server by an attacker to modify the DNS records.

#### Local DNS Cache Poisoning

From a local network perspective, an attacker can also perform DNS Cache Poisoning using MITM tools like [Ettercap](https://www.ettercap-project.org/) or [Bettercap](https://www.bettercap.org/).

To exploit the DNS cache poisoning via `Ettercap`, we should first edit the `/etc/ettercap/etter.dns` file to map the target domain name (e.g., `inlanefreight.com`) that they want to spoof and the attacker's IP address (e.g., `192.168.225.110`) that they want to redirect a user to:

	AstraX01@htb[/htb]# cat /etc/ettercap/etter.dns

Next, start the `Ettercap` tool and scan for live hosts within the network by navigating to `Hosts > Scan for Hosts`. Once completed, add the target IP address (e.g., `192.168.152.129`) to Target1 and add a default gateway IP (e.g., `192.168.152.2`) to Target2.

![[etter_plug.png]]

Activate `dns_spoof` attack by navigating to `Plugins > Manage Plugins`. This sends the target machine with fake DNS responses that will resolve `inlanefreight.com` to IP address `192.168.225.110`:

![[target.png]]

After a successful DNS spoof attack, if a victim user coming from the target machine `192.168.152.129` visits the `inlanefreight.com` domain on a web browser, they will be redirected to a `Fake page` that is hosted on IP address `192.168.225.110`:

In addition, a ping coming from the target IP address `192.168.152.129` to `inlanefreight.com` should be resolved to `192.168.225.110` as well:

	C:\>ping inlanefreight.com

 