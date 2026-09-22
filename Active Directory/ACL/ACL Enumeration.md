## Enumerating ACLs with PowerView

Let's dig in and see if this user has any interesting ACL rights that we could take advantage of. We first need to get the SID of our target user to search effectively.

Let's focus on the user `wley`

	PS C:\htb> Import-Module .\PowerView.ps1 
	PS C:\htb> $sid = Convert-NameToSid wley

We can then use the `Get-DomainObjectACL` function to perform our targeted search.

This is because the `ObjectAceType` property is returning a GUID value that is not human readable.

#### Using Get-DomainObjectACL

	PS C:\htb> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

#### Performing a Reverse Search & Mapping to a GUID Value

	PS C:\htb> $guid= "00299570-246d-11d0-a768-00aa006e0529" 
	PS C:\htb> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl

#### Using the -ResolveGUIDs Flag

	PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}


let's take a quick look at how we could do this using the [Get-Acl](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.2) and [Get-ADUser](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2022-ps) cmdlets which we may find available to us on a client system.

#### Creating a List of Domain Users

	PS C:\htb> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt

We then read each line of the file using a `foreach` loop, and use the `Get-Acl` cmdlet to retrieve ACL information for each domain user by feeding each line of the `ad_users.txt` file to the `Get-ADUser` cmdlet.

#### A Useful foreach Loop

	PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}

So, to recap, we started with the user `wley` and now have control over the user `damundsen` via the `User-Force-Change-Password` extended right.

#### Further Enumeration of Rights Using damundsen

	PS C:\htb> $sid2 = Convert-NameToSid damundsen 
	PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

Now we can see that our user `damundsen` has `GenericWrite` privileges over the `Help Desk Level 1` group.

#### Investigating the Help Desk Level 1 Group with Get-DomainGroup

	PS C:\htb> Get-DomainGroup -Identity "Help Desk Level 1" | select memberof 
	
	memberof 
	-------- 
	CN=Information Technology,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL

#### Investigating the Information Technology Group

let's look around and see if members of `Information Technology` can do anything interesting. Once again, doing our search using `Get-DomainObjectACL` shows us that members of the `Information Technology` group have `GenericAll` rights over the user `adunn`

	PS C:\htb> $itgroupsid = Convert-NameToSid "Information Technology" 
	PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose

#### Looking for Interesting Access

	PS C:\htb> $adunnsid = Convert-NameToSid adunn 
	PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose


---

## Enumerating ACLs with BloodHound

let's look at how much easier this would have been to identify using the extremely powerful BloodHound tool.

Let's take the data we gathered earlier with the SharpHound ingestor and upload it to BloodHound. Next, we can set the `wley` user as our starting node, select the `Node Info` tab and scroll down to `Outbound Control Rights`. This option will show us objects we have control over directly, via group membership, and the number of objects that our user could lead to us controlling via ACL attack paths under `Transitive Object Control`. If we click on the `1` next to `First Degree Object Control`, we see the first set of rights that we enumerated, `ForceChangePassword` over the `damundsen` user.

#### Viewing Node Info through BloodHound

![[wley_damundsen.png]]

#### Investigating ForceChangePassword Further

![[help_edge.png]]

If we click on the `16` next to `Transitive Object Control`, we will see the entire path that we painstakingly enumerated above. From here, we could leverage the help menus for each edge to find ways to best pull off each attack.

#### Viewing Potential Attack Paths through BloodHound

![[wley_path.png]]

Finally, we can use the pre-built queries in BloodHound to confirm that the `adunn` user has DCSync rights.

#### Viewing Pre-Build queries through BloodHound

![[adunn_dcsync 1.png]]

