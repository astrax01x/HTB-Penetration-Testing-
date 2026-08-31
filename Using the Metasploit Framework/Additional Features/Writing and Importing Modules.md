To install any new Metasploit modules which have already been ported over by other users, one can choose to update their `msfconsole` from the terminal, which will ensure that all newest exploits, auxiliaries, and features will be installed in the latest version of `msfconsole`

if we need only a specific module and do not want to perform a full upgrade, we can download that module and install it manually. We will focus on searching ExploitDB for readily available Metasploit modules, which we can directly import into our version of `msfconsole` locally.

[ExploitDB](https://www.exploit-db.com/) is a great choice when searching for a custom exploit. We can use tags to search through the different exploitation scenarios for each available script. One of these tags is [Metasploit Framework (MSF)](https://www.exploit-db.com/?tag=3), which, if selected, will display only scripts that are also available in Metasploit module format. These can be directly downloaded from ExploitDB and installed in our local Metasploit Framework directory, from where they can be searched and called from within the `msfconsole`.

Let's say we want to use an exploit found for `Nagios3`, which will take advantage of a command injection vulnerability. The module we are looking for is `Nagios3 - 'statuswml.cgi' Command Injection (Metasploit)`. So we fire up `msfconsole` and try to search for that specific exploit, but we cannot find it. This means that our Metasploit framework is not up to date or that the specific `Nagios3` exploit module we are looking for is not in the official updated release of the Metasploit Framework.
#### MSF - Search for Exploits

	msf6 > search nagios

We can, however, find the exploit code [inside ExploitDB's entries](https://www.exploit-db.com/exploits/9861). Alternatively, if we do not want to use our web browser to search for a specific exploit within ExploitDB, we can use the CLI version, `searchsploit`.

	AstraX01@htb[/htb]$ searchsploit nagios3

Note that the hosted file terminations that end in `.rb` are Ruby scripts that most likely have been crafted specifically for use within `msfconsole`. We can also filter only by `.rb` file terminations to avoid output from scripts that cannot run within `msfconsole`. Note that not all `.rb` files are automatically converted to `msfconsole` modules. Some exploits are written in Ruby without having any Metasploit module-compatible code in them.

	AstraX01@htb[/htb]$ searchsploit -t Nagios3 --exclude=".py"

We have to download the `.rb` file and place it in the correct directory. The default directory where all the modules, scripts, plugins, and `msfconsole` proprietary files are stored is `/usr/share/metasploit-framework`. The critical folders are also symlinked in our home and root folders in the hidden `~/.msf4/` location.

#### MSF - Directory Structure

	AstraX01@htb[/htb]$ ls /usr/share/metasploit-framework/

Note that our home folder `.msf4` location might not have all the folder structure that the `/usr/share/metasploit-framework/` one might have. So, we will just need to `mkdir` the appropriate folders so that the structure is the same as the original folder so that `msfconsole` can find the new modules. After that, we will be proceeding with copying the `.rb` script directly into the primary location.

Please note that there are certain naming conventions that, if not adequately respected, will generate errors when trying to get `msfconsole` to recognize the new module we installed. Always use snake-case, alphanumeric characters, and underscores instead of dashes.

#### MSF - Loading Additional Modules at Runtime

	AstraX01@htb[/htb]$ cp ~/Downloads/9861.rb /usr/share/metasploit-framework/modules/exploits/unix/webapp/nagios3_command_injection.rb

	AstraX01@htb[/htb]$ msfconsole -m /usr/share/metasploit-framework/modules/

#### MSF - Loading Additional Modules

	msf6> loadpath /usr/share/metasploit-framework/modules/

Alternatively, we can also launch `msfconsole` and run the `reload_all` command for the newly installed module to appear in the list. After the command is run and no errors are reported, try either the `search [name]` function inside `msfconsole` or directly with the `use [module-path]` to jump straight into the newly installed module.

	msf6 > reload_all

## Porting Over Scripts into Metasploit Modules

To adapt a custom Python, PHP, or any type of exploit script to a Ruby module for Metasploit, we will need to learn the Ruby programming language. Note that Ruby modules for Metasploit are always written using hard tabs.

