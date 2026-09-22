A [trust](https://social.technet.microsoft.com/wiki/contents/articles/50969.active-directory-forest-trust-attention-points.aspx) is used to establish forest-forest or domain-domain (intra-domain) authentication, which allows users to access resources in (or perform administrative tasks) another domain, outside of the main domain where their account resides.A trust creates a link between the authentication systems of two domains and may allow either one-way or two-way (bidirectional) communication.

An organization can create various types of trusts:

* `Parent-child`: Two or more domains within the same forest. The child domain has a two-way transitive trust with the parent domain
* `Cross-link`: A trust between child domains to speed up authentication.
- `External`: A non-transitive trust between two separate domains in separate forests which are not already joined by a forest trust.
* `Tree-root`: A two-way transitive trust between a forest root domain and a new tree root domain.
* `Forest`: A transitive trust between two forest root domains.
- [ESAE](https://docs.microsoft.com/en-us/security/compass/esae-retirement): A bastion forest used to manage Active Directory.

#### Trust Table Side By Side

| Transitive                                                            | Non-Transitive                              |
| --------------------------------------------------------------------- | ------------------------------------------- |
| Shared, 1 to many                                                     | Direct trust                                |
| The trust is shared with anyone in the forest                         | Not extended to next level child domains    |
| Forest, tree-root, parent-child, and cross-link trusts are transitive | Typical for external or custom trust setups |

**Trusts can be set up in two directions: one-way or two-way (bidirectional).**

---

## Enumerating Trust Relationships

We can use the [Get-ADTrust](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adtrust?view=windowsserver2022-ps) cmdlet to enumerate domain trust relationships.

#### Using Get-ADTrust

	PS C:\htb> Import-Module activedirectory 
	PS C:\htb> Get-ADTrust -Filter *

If we cannot authenticate across a trust, we cannot perform any enumeration or attacks across the trust.

Aside from using built-in AD tools such as the Active Directory PowerShell module, both PowerView and BloodHound can be utilized to enumerate trust relationships, the type of trusts established, and the authentication flow.

#### Checking for Existing Trusts using Get-DomainTrust

	PS C:\htb> Get-DomainTrust

PowerView can be used to perform a domain trust mapping and provide information such as the type of trust (parent/child, external, forest) and the direction of the trust (one-way or bidirectional).

#### Using Get-DomainTrustMapping

	PS C:\htb> Get-DomainTrustMapping

#### Checking Users in the Child Domain using Get-DomainUser

	PS C:\htb> Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName

Another tool we can use to get Domain Trust is `netdom`. The `netdom query` sub-command of the `netdom` command-line tool in Windows can retrieve information about the domain, including a list of workstations, servers, and domain trusts.

#### Using netdom to query domain trust

	C:\htb> netdom query /domain:inlanefreight.local trust Direction Trusted\Trusting domain

#### Using netdom to query domain controllers

	C:\htb> netdom query /domain:inlanefreight.local dc

#### Using netdom to query workstations and servers

	C:\htb> netdom query /domain:inlanefreight.local workstation

We can also use BloodHound to visualize these trust relationships by using the `Map Domain Trusts` pre-built query. Here we can easily see that two bidirectional trusts exist.

#### Visualizing Trust Relationships in BloodHound

![[BH_trusts.png]]