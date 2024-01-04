---
title: Solution for Forgetting Root Password in MySQL
date: 2024-01-04 20:56:00 +0800
categories: [MySQL]
tags: []

---



MySQL version: 8.2.0

System: Mac OS ARM

1. Open a Terminal window, use the command below to stop mysql if it's already running.

```
sudo /usr/local/mysql/support-files/mysql.server stop
```

Note: If the terminal shows this error message:

```
sudo /usr/local/mysql/support-files/mysql.server stop
ERROR! MySQL server PID file could not be found!
```

check the status

```
/usr/local/mysql/support-files/mysql.server status
```

find all running mysql processes

```
ps aux | grep mysql
```

kill off all the mysql pids

```
sudo kill <pid>
```

try to start mysql again

```
sudo /usr/local/mysql/support-files/mysql.server start
```

then try to stop mysql

```
sudo /usr/local/mysql/support-files/mysql.server stop
```

2. start MySQL with this command:

```
sudo /usr/local/mysql/bin/mysqld_safe --skip-grant-tables
```

3. Open a new terminal window

```
sudo /usr/local/mysql/bin/mysql -u root
```

4. reset password

```sql
mysql> USE mysql;
mysql> UPDATE user SET password=PASSWORD("NEWPASSWORD") WHERE User='root';
mysql> FLUSH PRIVILEGES;
mysql> quit
```

for MySQL 8.0+

```sql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass';
```

5. stop MySQL server

```
sudo /usr/local/mysql/support-files/mysql.server stop
```

6. Restart MySQL

```
sudo /usr/local/mysql/support-files/mysql.server start
```



**Reference:**

https://stackoverflow.com/a/36252036

https://serverfault.com/a/767339

https://stackoverflow.com/a/6474890