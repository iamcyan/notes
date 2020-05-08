# mysql

**Mac 下查看mysql 读取的配置文件**

```
$ mysqld --help --verbose | less
...
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/local/etc/my.cnf ~/.my.cnf
...
```

**mysql 8.0 中创建用户**

```
mysql> CREATE USER 'mike'@'%' IDENTIFIED BY '000000';
mysql> GRANT ALL ON *.* TO 'mike'@'%' WITH GRANT OPTION;
```

**更改MySQL 8.0 的密码验证插件**

```
default_authentication_plugin=mysql_native_password
```

**更改原来用户的密码验证方式**

```
ALTER USER 'root'@'localhost' IDENTIFIED BY '密码' PASSWORD EXPIRE NEVER;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码';
FLUSH PRIVILEGES;
```

