
LLMNR & NBT-NS poisoning is possible from a Windows host as well. In the last section, we utilized Responder to capture hashes. This section will explore the tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) and attempt to capture another set of credentials.

## Inveigh - Overview

*our client provides us with a Windows box to test from, or we land on a Windows host as a local admin via another attack method and would like to look to further our access, the tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) works similar to Responder, but is written in PowerShell and C#. Inveigh can listen to IPv4 and IPv6 and several other protocols, including `LLMNR`, DNS, `mDNS`, NBNS, `DHCPv6`, ICMPv6, `HTTP`, HTTPS, `SMB`, LDAP, `WebDAV`, and Proxy Auth. The tool is available in the `C:\Tools` directory on the provided Windows attack host.*

## Using Inveigh

	  PS C:\htb> Import-Module .\Inveigh.ps1
	  PS C:\htb> (Get-Command Invoke-Inveigh).Parameters

Let's start Inveigh with LLMNR and NBNS spoofing, and output to the console and write to a file. We will leave the rest of the defaults

	 PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y

## C# Inveigh (InveighZero)

The PowerShell version of Inveigh is the original version and is no longer updated. The tool author maintains the C# version, which combines the original PoC C# code and a C# port of most of the code from the PowerShell version.

Let's go ahead and run the C# version with the defaults and start capturing hashes.

	PS C:\htb> .\Inveigh.exe

the tool starts and shows which options are enabled by default and which are not. The options with a `[+]` are default and enabled by default and the ones with a `[ ]` before them are disabled.

**Option to use C# inveigh** 

* We can hit the `esc` key to enter the console while Inveigh is running.

* We can quickly view unique captured hashes by typing `GET NTLMV2UNIQUE`.

* We can type in `GET NTLMV2USERNAMES` and see which usernames we have collected.



While it is not possible to disable NBT-NS directly via GPO, we can create a PowerShell script under Computer Configuration --> Windows Settings --> Script (Startup/Shutdown) --> Startup with something like the following:

#### AFTER THIS : 

At this point in our assessment, we would want to perform enumeration using a tool such as BloodHound to determine whether any or all of these hashes are worth cracking. If we get lucky and crack a hash for a user account with some privileged access or rights, we can begin expanding our reach into the domain.


