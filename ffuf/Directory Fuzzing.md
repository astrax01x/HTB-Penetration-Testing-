## Directory Fuzzing

we can pick our wordlist and assign the keyword `FUZZ` to it by adding `:FUZZ` after it:

	AstraX01@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ

Next, as we want to be fuzzing for web directories, we can place the `FUZZ` keyword where the directory would be within our URL, with:

	AstraX01@htb[/htb]$ ffuf -w <SNIP> -u http://SERVER_IP:PORT/FUZZ

run our final command on it:

	AstraX01@htb[/htb]$ AstraX01@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ

