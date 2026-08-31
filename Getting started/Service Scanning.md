## NMAP 

	 nmap (target ip)
	 nmap -sC -sV (target ip)

## FTP  

It is worth gaining familiarity with FTP, as it is a standard protocol, and this service can often contain interesting data. 

	 ftp -p (target ip)

*  *In ftp shell , we see that FTP supports common commands such as `cd` and `ls` and allows us to download files using the `get` command. Inspection of the downloaded `login.txt` reveals credentials that we could use to further our access to the system.**

## SMB 

SMB (Server Message Block) is a prevalent protocol on Windows machines that provides many vectors for vertical and lateral movement. Sensitive data, including credentials, can be in network file shares, and some SMB versions may be vulnerable to RCE exploits such as [EternalBlue](https://www.avast.com/c-eternalblue).

==`Nmap` has many scripts for enumerating SMB, such as [smb-os-discovery.nse](https://nmap.org/nsedoc/scripts/smb-os-discovery.html), which will interact with the SMB service to extract the reported operating system version.==

	 nmap --script smb-os-discovery.nse -p445 (target ip)

#### Shares

SMB allows users and administrators to share folders and make them accessible remotely by other users. Often these shares have files in them that contain sensitive information such as passwords. A tool that can enumerate and interact with SMB shares is [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)

	 smbclient -N -L \\\\10.129.42.253

 *   The `-L` flag specifies that we want to retrieve a list of available shares on the remote host
 *   `-N` suppresses the password prompt

**==This reveals the non-default share `**users**`. Let us attempt to connect as the guest user.==**

	 smbclient \\\\10.129.42.253\\users

The `ls` command resulted in an access denied message, indicating that guest access is not permitted. Let us try again using credentials for the user bob (`bob:Welcome1`).

	 smbclient -U bob \\\\10.129.42.253\\users

## SNMP 

SNMP Community strings provide information and statistics about a router or device, helping us gain access to it. The manufacturer default community strings of `public` and `private` are often unchanged .

	 snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0

	 snmpwalk -v 2c -c private 10.129.42.253

==A tool such as [onesixtyone](https://github.com/trailofbits/onesixtyone) can be used to brute force the community string names using a dictionary file of common community strings such as the `dict.txt` file included in the GitHub repo for the tool.==

	 onesixtyone -c dict.txt 10.129.42.254
