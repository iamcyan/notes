# PHP 中的类与对象

## 简介

### 变量持有对象的引用

> PHP 对待对象的方式与引用和句柄相同，即每个变量都持有对象的引用，而不是整个对象的拷贝

## static关键字

[文档](<https://www.php.net/manual/zh/oop5.intro.php>)

1、声明类属性或方法为静态，就可以不实例化类而直接访问。静态属性不能通过一个类已实例化的对象来访问（但静态方法可以）。

2、静态方法的执行速度和非静态方法的执行速度比较起来，几乎没有区别。（某些资料中显示静态方法的速度快4倍，实际测试1000W for 循环，两者之间的差距在几毫秒之间）

3、哪些场景下使用静态方法比较好呢？

## Trait

[文档](<https://www.php.net/manual/zh/language.oop5.traits.php>)

Trait 是为类似 PHP 的单继承语言而准备的一种代码复用机制。Trait 为了减少单继承语言的限制，使开发人员能够自由地在不同层次结构内独立的类中复用 method，它为传统继承增加了水平特性的组合。

在我们的项目中，可以把一些含有逻辑的公共方法写到里面，而不总是在 Common 或者 Library 中实现

### 优先级

1、子类、`trait`、父类中的方法优先级顺序：子类 > trait > 父类

2、组合多个 trait： `use Trait1, Trait2`

3、多个 `trait` 组合发生冲突：

```php
class Talker {
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
    }
}
```

4、修改访问控制：可以使用 `as` 来修改方法的访问控制、别名

5、可以在 `trait` 中使用其他的 `trait`

6、静态成员和属性参见[文档](https://www.php.net/manual/zh/language.oop5.traits.php)

7、Laravel 中使用 `trait`例子：lumen 中基础控制器使用了trait `ProvidesConvenienceMethods`



## 类型约束 && 函数返回类型声明

[文档](<https://www.php.net/manual/zh/language.oop5.typehinting.php>)

### 类型约束

1、函数的参数可以指定必须为对象、数组、接口、callable

2、PHP7 中增加了标量类型声明：string、int、float、bool

### 返回类型声明

[文档](<https://www.php.net/manual/zh/functions.returning-values.php#functions.returning-values.type-declaration>)

1、返回类型声明执行了函数返回值的类型，返回类型错误时会产生一个 fatal error。

2、返回值类型可以声明为标量，在默认的弱模式中，如果返回值的类型和声明的类型不一样的话，那么会将类型进行强制转换。如果声明了为强模式，类型不一致时会产生一个 fatal error 。

```php
<?php

function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
/**
Array
(
    [0] => 6
    [1] => 15
    [2] => 24
)
*/
```

## 后期静态绑定

[文档](<https://www.php.net/manual/zh/language.oop5.late-static-bindings.php>)

[理解PHP延迟静态绑定](<https://learnku.com/laravel/t/3844/understanding-php-delayed-static-binding-late-static-bindings>)

### 需要的一些其他概念

> 转发调用：所谓的 “转发调用”（forwarding call）指的是通过以下几种方式进行的静态调用：self::, parent::, static:: 以及 forward_ *static* _call ()。即在进行静态调用时未指名类名的调用属于转发调用。

> 非转发调用：非转发调用其实就是明确指定类名的静态调用（foo::bar ()）和非静态调用 ($foo->bar ())。即明确地指定类名的静态调用和非静态调用。

顾名思义，非转发调用前面有类名所以调用的函数一定是属于 “这个类的”，不需要转到别的类。转发调用就是由于前期的静态绑定导致在后面调用静态方法时可能 “转发到其他的类”

> 后期静态绑定：在 PHP 的官方文档里，对于后期静态绑定是这样说的：后期静态绑定工作原理是存储了在上一个 “非转发调用”（non-forwarding call）中的类名。意思是当我们调用一个转发调用的静态调用时，实际调用的类是上一个非转发调用的类。

例子1：

```php
class A {
    public static function who() {
        echo __CLASS__;
    }
    public static function test() {
        static::who(); // 后期静态绑定从这里开始
    }
}
class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}
B::test();	//A
```

例子2

```php
class A {
    public static function foo() {
        static::who();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}

class B extends A {
    public static function test() {
        A::foo();
        parent::foo();
        self::foo();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}
class C extends B {
    public static function who() {
        echo __CLASS__."\n";
    }
}

C::test();
```

输出结果

```php
A
C
C
```

分析一下例子2：

1、C::test ()，这是一个非转发调用，因为：：前面有类名 C。

2、进入 test () 方法，有三个静态调用 A::foo (),parent::foo (),self::foo (), 对于这三个静态调用来说，他们的非转发调用类就是 C。

3、 现在执行 A::foo (), 这是一个非转发调用。A::foo () 中的代码是 static::who (), 这是一个转发调用，对于这个转发调用来说他的非转发调用类就是不再是 C 而是 A（因为之前执行了 A::foo ()）。因此执行的结果为 A

4、 现在执行 parent::foo (), 这是一个转发调用，转发到哪里呢？就是它的上一个非转发调用的类，也就是类 C（在步骤 2 中提到的）。在这里一定要注意虽然在这之前执行了 A::foo (), 但是 parent::foo () 的上一个非转发调用的类任然是类 C。因此执行的结果是 C.

5、现在执行 self::foo (), 这个和 parent::foo () 一样都是转发调用，因此也输出 C。

### 使用场景：

1、static:: 中的 static 其实是运行时所在类的别名，并不是定义类时所在的那个类名。这个东西可以实现在父类中能够调用子类的方法和属性。

2、使用 (static) 关键字来表示这个别名，和静态方法，静态类没有半毛钱的关系，static:: 不仅支持静态类，还支持对象（动态类）。

备注：参考手册中第一条评论

例子

```php
<?php
// new self 得到的单例都为A。
class A
{
    protected static $_instance = null;

    protected function __construct()
    {
        //disallow new instance
    }

    protected function __clone(){
        //disallow clone
    }

    static public function getInstance()
    {
        if (self::$_instance === null) {
            self::$_instance = new self();
        }
        return self::$_instance;
    }
}

class B extends A
{
    protected static $_instance = null;
}

class C extends A{
    protected static $_instance = null;
}

$a = A::getInstance();
$b = B::getInstance();
$c = C::getInstance();

var_dump($a);
var_dump($b);
var_dump($c);




运行结果：
E:\code\php_test\apply\self.php:37:
class A#1 (0) {
}
E:\code\php_test\apply\self.php:38:
class A#1 (0) {
}
E:\code\php_test\apply\self.php:39:
class A#1 (0) {
}
```

```php
<?php
// new static 得到的单例分别为D，E和F。
class D
{
    protected static $_instance = null;

    protected function __construct(){}
    protected function __clone()
    {
        //disallow clone
    }

    static public function getInstance()
    {
        if (static::$_instance === null) {
            static::$_instance = new static();
        }
        return static::$_instance;
    }
}

class E extends D
{
    protected static $_instance = null;
}

class F extends D{
    protected static $_instance = null;
}

$d = D::getInstance();
$e = E::getInstance();
$f = F::getInstance();

var_dump($d);
var_dump($e);
var_dump($f);




运行结果：
E:\code\php_test\apply\static.php:35:
class D#1 (0) {
}
E:\code\php_test\apply\static.php:36:
class E#2 (0) {
}
E:\code\php_test\apply\static.php:37:
class F#3 (0) {
}
```