To escalate our privileges most efficiently, we can search for passwords or even whole credentials that we can use to log in to our target. There are several sources that can provide us with credentials that we put in four categories. These include, but are not limited to:

- `Files` including configs, databases, notes, scripts, source code, cronjobs, and SSH keys
- `History` including logs, and command-line history
- `Memory` including cache, and in-memory processing
- `Key-rings` such as browser stored credentials

---

## Files

 We should look for, find, and inspect several categories of files one by one.

- Configuration files
- Databases
- Notes
- Scripts
- Cronjobs
- SSH keys

Configuration files are the core of the functionality of services on Linux distributions. Often they even contain credentials that we will be able to read.
Usually, the configuration files are marked with the following three file extensions (`.config`, `.conf`, `.cnf`).

---

#### Searching for configuration files

the first step should be to find all possible configuration files on the system.

	AstraX01@htb[/htb]$ for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done

Optionally, we can save the result in a text file and use it to examine the individual files one after the other.

	AstraX01@htb[/htb]$ for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done

---

#### Searching for databases

We can apply this simple search to the other file extensions as well. Additionally, we can apply this search type to databases stored in files with different file extensions.

	AstraX01@htb[/htb]$ for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done

---

#### Searching for notes

Depending on the environment we are in and the purpose of the host we are on, we can often find notes about specific processes on the system. These often include lists of many different access points or even their credentials.

	AstraX01@htb[/htb]$ find /home/* -type f -name "*.txt" -o ! -name "*.*"

---

#### Searching for scripts

Scripts are files that often contain highly sensitive information and processes. Among other things, these also contain credentials that are necessary to be able to call up and execute the processes automatically.

	AstraX01@htb[/htb]$ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done 

---

#### Enumerating cronjobs

Cronjobs are independent execution of commands, programs, scripts.

These are divided into the system-wide area (`/etc/crontab`) and user-dependent executions. Some applications and scripts require credentials to run and are therefore incorrectly entered in the cronjobs. Furthermore, there are the areas that are divided into different time ranges (`/etc/cron.daily`, `/etc/cron.hourly`, `/etc/cron.monthly`, `/etc/cron.weekly`). The scripts and files used by `cron` can also be found in `/etc/cron.d/` for Debian-based distributions.

	AstraX01@htb[/htb]$ cat /etc/crontab

	AstraX01@htb[/htb]$ ls -la /etc/cron.*/

---

#### Enumerating history files

All history files provide crucial information about the current and past/historical course of processes. We are interested in the files that store users' command history and the logs that store information about system processes.

other files like `.bashrc` or `.bash_profile` can contain important information.

Linux distributions that use Bash as a standard shell, we find the associated files in `.bash_history`.

	AstraX01@htb[/htb]$ tail -n5 /home/*/.bash*

---

#### Enumerating log files

An essential concept of Linux systems is log files that are stored in text files. Many programs, especially all services and the system itself, write such files.

- Application logs
- Event logs
- Service logs
- System logs

Many different logs exist on the system.

|**File**|**Description**|
|---|---|
|`/var/log/messages`|Generic system activity logs.|
|`/var/log/syslog`|Generic system activity logs.|
|`/var/log/auth.log`|(Debian) All authentication related logs.|
|`/var/log/secure`|(RedHat/CentOS) All authentication related logs.|
|`/var/log/boot.log`|Booting information.|
|`/var/log/dmesg`|Hardware and drivers related information and logs.|
|`/var/log/kern.log`|Kernel related warnings, errors and logs.|
|`/var/log/faillog`|Failed login attempts.|
|`/var/log/cron`|Information related to cron jobs.|
|`/var/log/mail.log`|All mail server related logs.|
|`/var/log/httpd`|All Apache related logs.|
|`/var/log/mysqld.log`|All MySQL server related logs.|
here are some strings we can use to find interesting content in the logs:

	AstraX01@htb[/htb]$ for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done

---

## Memory and cache

#### Mimipenguin

Many applications and processes work with credentials needed for authentication and store them either in memory or in files so that they can be reused.

 there is a tool called [mimipenguin](https://github.com/huntergregal/mimipenguin) that makes the whole process easier. However, this tool requires administrator/root permissions.

	AstraX01@htb[/htb]$ sudo python3 mimipenguin.py

#### LaZagne

An even more powerful tool we can use that was mentioned earlier in the Credential Hunting in Windows section is `LaZagne`. This tool allows us to access far more resources and extract the credentials.

#### Browser credentials

Browsers store the passwords saved by the user in an encrypted form locally on the system to be reused.

 the `Mozilla Firefox` browser stores the credentials encrypted in a hidden folder for the respective user. These often include the associated field names, URLs, and other valuable information.

when we store credentials for a web page in the Firefox browser, they are encrypted and stored in `logins.json` on the system.

	[!bash]$ ls -l .mozilla/firefox/ | grep default

	AstraX01@htb[/htb]$ cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

The tool [Firefox Decrypt](https://github.com/unode/firefox_decrypt) is excellent for decrypting these credentials, and is updated regularly. It requires Python 3.9 to run the latest version. Otherwise, `Firefox Decrypt 0.7.0` with Python 2 must be used.

	AstraX01@htb[/htb]$ python3.9 firefox_decrypt.py

