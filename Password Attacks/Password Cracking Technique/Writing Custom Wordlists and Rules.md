Knowing that users tend to keep their passwords as simple as possible, we can create rules to generate likely weak passwords.

According to statistics provided by [WP Engine](https://wpengine.com/resources/passwords-unmasked-infographic/), most passwords are no longer than `ten`characters.

One approach is to select familiar terms that are at least five characters long—such as pet names, hobbies, personal preferences, or other common interests.

Let's look at a simple example using a password list with only one entry.

	AstraX01@htb[/htb]$ cat password.list

We can use Hashcat to combine lists of potential names and labels with specific mutation rules to create custom wordlists.

Hashcat uses a specific syntax to define characters, words, and their transformations. The complete syntax is documented in the official [Hashcat rule-based attack documentation](https://hashcat.net/wiki/doku.php?id=rule_based_attack), but the examples below are sufficient to understand how Hashcat mutates input words.

|**Function**|**Description**|
|---|---|
|`:`|Do nothing|
|`l`|Lowercase all letters|
|`u`|Uppercase all letters|
|`c`|Capitalize the first letter and lowercase others|
|`sXY`|Replace all instances of X with Y|
|`$!`|Add the exclamation character at the end|

Each rule is written on a new line and determines how a given word should be transformed.

	AstraX01@htb[/htb]$ cat custom.rule

We can use the following command to apply the rules in `custom.rule` to each word in `password.list` and store the mutated results in `mut_password.list`.

	AstraX01@htb[/htb]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list

In this case, the single input word will produce fifteen mutated variants.

	AstraX01@htb[/htb]$ cat mut_password.list

Hashcat and JtR both come with pre-built rule lists that can be used for password generation and cracking.

 One of the most effective and widely used rulesets is `best64.rule`, which applies common transformations that frequently result in successful password guesses.

It is important to note that password cracking and the creation of custom wordlists are, in most cases, a guessing game. We can narrow this down and perform more targeted guessing if we have information about the password policy, while considering factors such as the company name, geographical region, industry, and other topics or keywords that users might choose when creating their passwords. Exceptions, of course, include cases where passwords have been leaked and directly obtained.

## Generating wordlists using CeWL

We can use a tool called [CeWL](https://github.com/digininja/CeWL) to scan potential words from a company's website and save them in a separate list.

We can then combine this list with the desired rules to create a customized password list—one that has a higher probability of containing the correct password for an employee. We specify some parameters, like the depth to spider (`-d`), the minimum length of the word (`-m`), the storage of the found words in lowercase (`--lowercase`), as well as the file where we want to store the results (`-w`).

	AstraX01@htb[/htb]$ cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
	AstraX01@htb[/htb]$ wc -l inlane.wordlist

	326

