#PHP 连接 MySQL

版本：macOs 	PHP 7.3	MySQL 8.0

my.cnf 位置：/usr/local/etc/my.cnf

**PHP 连接 MySQL 报错**

```
$user = "root";
$pass = "123456";
try {
    $dbh = new PDO('mysql:host=127.0.0.1;dbname=iamcyan', $user, $pass);
} catch (\Exception $exception) {
    echo $exception->getMessage();
}

//异常信息：SQLSTATE[HY000] [2054] The server requested authentication method unknown to the client

//原因：mysql 8.0 更改了验证密码所使用的插件，默认情况下创建的用户使用的加密插件是 caching_sha2_password，过去使用的插件是 mysql_native_password。现在 php 中使用的连接扩展目前还使用的是原有的插件。

//解决方法
//1、在my.cnf 中更改密码验证插件后新建用户，使用新用户来进行连接。
//2、

```



**持久连接**

```php
$dbh = new PDO('mysql:host=localhost;dbname=test', $user, $pass, array(
    PDO::ATTR_PERSISTENT => true
));
```

