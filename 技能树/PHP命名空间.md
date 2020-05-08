# 命名空间

## 命名空间概述

### 命名空间的概念

将操作系统中的文件目录的概念应用到程序设计领域

### 解决问题

1、用户编写的代码与PHP内部的类/函数/常量或者第三方类/函数/常量冲突

2、为很长的标识福创建一个别名，提高源代码可读性

### 注意事项

1、名为PHP 或者 php 的命名空间，以及以这些名字开头的命名空间，为PHP 被保留为PHP 内核使用。

## 定义命名空间 && 子命名空间

1、命名空间使用 namespace 关键字声明，如果使用命名空间必须在所有代码前声明，除 declare 关键字。

2、受命名空间影响的代码：类、接口、函数、常量

3、命名空间和目录一样，可以指定层次化的命名空间

4、一个文件中可以定义多个命名空间，但是极不推荐使用这种方法。

##如何使用命名空间

### 基础

使用命名空间引入类的三种方法：

1、非限定名称：在当前命名空间下解析对应的类

2、限定名称：即使用子命名空间

3、完全限定名称：如 \CurrentNamespace\Foo::staticmethod()；

举例：

file1.php

```php
<?php
namespace Foo\Bar\subnamespace;

const FOO = 1;
function foo() {}
class foo
{
    static function staticmethod() {}
}
?>
```

```php
<?php
namespace Foo\Bar;
include 'file1.php';

const FOO = 2;
function foo() {}
class foo
{
    static function staticmethod() {}
}

/* 非限定名称 */
foo(); // 解析为 Foo\Bar\foo resolves to function Foo\Bar\foo
foo::staticmethod(); // 解析为类 Foo\Bar\foo的静态方法staticmethod。resolves to class Foo\Bar\foo, method staticmethod
echo FOO; // resolves to constant Foo\Bar\FOO

/* 限定名称 */
subnamespace\foo(); // 解析为函数 Foo\Bar\subnamespace\foo
subnamespace\foo::staticmethod(); // 解析为类 Foo\Bar\subnamespace\foo,
                                  // 以及类的方法 staticmethod
echo subnamespace\FOO; // 解析为常量 Foo\Bar\subnamespace\FOO
                                  
/* 完全限定名称 */
\Foo\Bar\foo(); // 解析为函数 Foo\Bar\foo
\Foo\Bar\foo::staticmethod(); // 解析为类 Foo\Bar\foo, 以及类的方法 staticmethod
echo \Foo\Bar\FOO; // 解析为常量 Foo\Bar\FOO
?>
```

