Domain information is a core component of any penetration test, and it is not just about the subdomains but about the entire presence on the Internet. Therefore, we gather information and try to understand the company's functionality and which technologies and structures are necessary for services to be offered successfully and efficiently.

## Online Presence

Once we have a basic understanding of the company and its services, we can get a first impression of its presence on the Internet. Let us assume that a medium-sized company has hired us to test their entire infrastructure from a black-box perspective. This means we have only received a scope of targets and must obtain all further information ourselve

#### Certificate Transparency

	 AstraX01@htb[/htb]$ curl -s https://crt.sh/\ q\=inlanefreight.com\&output\=json | jq .

If needed, we can also have them filtered by the unique subdomains.

	 AstraX01@htb[/htb]$ curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u

#### Company Hosted Servers

	 AstraX01@htb[/htb]$ for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4;done

Once we see which hosts can be investigated further, we can generate a list of IP addresses with a minor adjustment to the `cut` command and run them through `Shodan`.

#### Shodan - IP List

	 AstraX01@htb[/htb]$ for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done

	 AstraX01@htb[/htb]$ for i in $(cat ip-addresses.txt);do shodan host $i;done

#### DNS Records

	 AstraX01@htb[/htb]$ dig any inlanefreight.com

