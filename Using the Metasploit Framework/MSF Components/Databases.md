 `Msfconsole` has built-in support for the PostgreSQL database system. With it, we have direct, quick, and easy access to scan results with the added ability to import and export results in conjunction with third-party tools. Database entries can also be used to configure Exploit module parameters with the already existing findings directly.

## Setting up the Database

First, we must ensure that the PostgreSQL server is up and running on our host machine. To do so, input the following command:

#### PostgreSQL Status

	AstraX01@htb[/htb]$ sudo service postgresql status

#### Start PostgreSQL

	AstraX01@htb[/htb]$ sudo systemctl start postgresql

After starting PostgreSQL, we need to create and initialize the MSF database with `msfdb init`.

#### MSF - Initiate a Database

	AstraX01@htb[/htb]$ sudo msfdb init

If the initialization is skipped and Metasploit tells us that the database is already configured, we can recheck the status of the database.

	AstraX01@htb[/htb]$ sudo msfdb status

If this error does not appear, which often happens after a fresh installation of Metasploit, then we will see the following when initializing the database:

	AstraX01@htb[/htb]$ sudo msfdb init

After the database has been initialized, we can start `msfconsole` and connect to the created database simultaneously.

#### MSF - Connect to the Initiated Database'

	AstraX01@htb[/htb]$ sudo msfdb run

If, however, we already have the database configured and are not able to change the password to the MSF username, proceed with these commands:

#### MSF - Reinitiate the Database

	AstraX01@htb[/htb]$ msfdb reinit 
	AstraX01@htb[/htb]$ cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/ 
	AstraX01@htb[/htb]$ sudo service postgresql restart 
	AstraX01@htb[/htb]$ msfconsole -q \
	
	msf6 > db_status

Now, we are good to go. The `msfconsole` also offers integrated help for the database. This gives us a good overview of interacting with and using the database.

#### MSF - Database Options

	msf6 > help database

	msf6 > db_status


## Using the Database

With the help of the database, we can manage many different categories and hosts that we have analyzed. Alternatively, the information about them that we have interacted with using Metasploit. These databases can be exported and imported. This is especially useful when we have extensive lists of hosts, loot, notes, and stored vulnerabilities for these hosts. After confirming that the database is successfully connected, we can organize our `Workspaces`.

#### Workspaces

We can think of `Workspaces` the same way we would think of folders in a project. We can segregate the different scan results, hosts, and extracted information by IP, subnet, network, or domain.

To view the current Workspace list, use the `workspace` command. Adding a `-a` or `-d` switch after the command, followed by the workspace's name, will either `add` or `delete` that workspace to the database.

	msf6 > workspace
	* default

**Notice that the default Workspace is named `default` and is currently in use according to the `*` symbol.**

Type the `workspace [name]` command to switch the presently used workspace. Looking back at our example, let us create a workspace for this assessment and select it.

	msf6 > workspace -a Target_1  

(-a is to add a workspace)

	msf6 > workspace Target_1

	msf6 > workspace

To see what else we can do with Workspaces, we can use the `workspace -h` command for the help menu related to Workspaces.

	msf6 > workspace -h

## Importing Scan Results

 let us assume we want to import a `Nmap scan` of a host into our Database's Workspace to understand the target better. We can use the `db_import` command for this.

After the import is complete, we can check the presence of the host's information in our database by using the `hosts` and `services` commands. Note that the `.xml` file type is preferred for `db_import`.

#### Stored Nmap Scan

	AstraX01@htb[/htb]$ cat Target.nmap

#### Importing Scan Results

	msf6 > db_import Target.xml

	msf6 > hosts

	msf6 > services

---

## Using Nmap Inside MSFconsole

Alternatively, we can use Nmap straight from msfconsole! To scan directly from the console without having to background or exit the process, use the `db_nmap` command.

#### MSF - Nmap

	msf6 > db_nmap -sV -sS 10.10.10.8

---

## Data Backup

After finishing the session, make sure to back up our data if anything happens with the PostgreSQL service. To do so, use the `db_export` command.

#### MSF - DB Export

	msf6 > db_export -h

	msf6 > db_export -f xml backup.xml

 *Other commands related to data retention are the extended use of `hosts`, `services`, and the `creds` and `loot` commands.*

## Hosts

The `hosts` command displays a database table automatically populated with the host addresses, hostnames, and other information we find about these during our scans and interactions. For example, suppose `msfconsole` is linked with scanner plugins that can perform service and OS detection. In that case, this information should automatically appear in the table once the scans are completed through msfconsole. Again, tools like Nessus, NexPose, or Nmap will help us in these cases.

#### MSF - Stored Hosts

	msf6 > hosts -h

## Services

The `services` command functions the same way as the previous one. It contains a table with descriptions and information on services discovered during scans or interactions. In the same way as the command above, the entries here are highly customizable.

#### MSF - Stored Services of Hosts

	msf6 > services -h

## Credentials

The `creds` command allows you to visualize the credentials gathered during your interactions with the target host. We can also add credentials manually, match existing credentials with port specifications, add descriptions, etc.

#### MSF - Stored Credentials

	msf6 > creds -h

## Loot

The `loot` command works in conjunction with the command above to offer you an at-a-glance list of owned services and users. The loot, in this case, refers to hash dumps from different system types, namely hashes, passwd, shadow, and more.

#### MSF - Stored Loot

	msf6 > loot -h


