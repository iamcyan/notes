# 使用php://input获取POST请求数据

### 背景

> 使用lumen系统，使用了前后端分离，在和前端进行交互过程中发现，在中间件中验签的时候发现不能使用$request->all()的方法获取请求中的参数。经过排查发现，前端并没有使用表单形式提交数据，而是以raw的方法请求的后端接口。

###比较form-data、x-www-form-urlencode、raw

接收请求参数并返回

```php
Route::post('/post', function () {

    $post = $_POST;
    $request = $_REQUEST;
    $input = file_get_contents("php://input");
    $data['post'] = $post;
    $data['request'] = $request;
    $data['input'] = $input;

    return $data;
});
```

form-data

```php
//对应的请求头信息是 Content-Type:multipart/form-data。即可以上传文件
//返回结果
{
    "post": {
        "name": "123",
        "age": "100"
    },
    "request": {
        "name": "123",
        "age": "100"
    },
    "input": "",
    "file": {
        "file_name": {
            "name": "backup.html",
            "type": "text/html",
            "tmp_name": "/tmp/php0Ugb3K",
            "error": 0,
            "size": 13229
        }
    }
}
```

x-www-form-urlencode

```php
//对应的请求头信息是 Content-Type:application/x-www-form-urlencoded。只能填写key-value形式的参数
{
    "post": {
        "name": "iam",
        "age": "100"
    },
    "request": {
        "name": "iam",
        "age": "100"
    },
    "input": "name=iam&age=100",
    "file": []
}
```

raw

```php
//对应的请求头信息是 Content-Type:
//可选的值：text、text/xml、text/html、text/plain、application/json、application/xml等
{
    "post": [],
    "request": [],
    "input": "{\"name\":\"iamcyan\",\"age\":\"100\"}",
    "file": []
}
```



### php:input

```
php://input 是个可以访问请求的原始数据的只读流。 POST 请求的情况下，最好使用 php://input 来代替 $HTTP_RAW_POST_DATA，因为它不依赖于特定的 php.ini 指令。 而且，这样的情况下 $HTTP_RAW_POST_DATA 默认没有填充， 比激活 always_populate_raw_post_data 潜在需要更少的内存。 enctype="multipart/form-data" 的时候 php://input 是无效的。
```



###当时错误

```
//使用的是raw的方式，但是设置的请求头是Content-Type:application/x-www-form-urlencoded，所以会出现以请求内容为key的情况。
{
    "post": {
        "{\"name\":12312}": ""
    },
    "request": {
        "{\"name\":12312}": ""
    },
    "input": "{\"name\":12312}",
    "file": []
}
```

### 结论

```
1、当使用multipart/form-data时，php://input无法读取post数据
2、php://input读取的是原始数据
3、所有的请求应该设置合理的请求头
```



参考文章：<http://www.nowamagic.net/academy/detail/12220520>





