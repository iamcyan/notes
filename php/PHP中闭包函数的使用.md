# 闭包函数

### PHP手册中关于闭包函数

1、闭包函数都是依靠Closure类实现的。

2、闭包函数从父作用域集成变量，闭包函数的父作用域为定义闭包函数的作用域，而不是调用闭包函数的地方。

3、使用父作用域的变量，需要使用use()。变量通常为值拷贝，也可以使用引用传值。

### 闭包函数怎么用呢？

> 1、如果函数中，有一段逻辑是需要由使用者来确定的，传入闭包函数，由闭包函数执行此逻辑，例如laravel中的each()方法

```php
collect([1,2])->each(function ($item) {
        echo $item * 2 . PHP_EOL;
    });	
```

> 2、如果函数中，有bool值需要使用者来确定，使用闭包函数传入，如  laravel 中

```php
$res = collect([1,2])->filter(function ($item) {
        return $item % 2;
    });
```

> 3、如果一个对象必须由使用者更改，而非函数本身更改，那么需要传入对应的闭包函数

```php
$arr = [1,2];
    array_walk($arr, function (&$value) {
        $value *= 2;
    });
```



