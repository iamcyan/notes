# CGI、FastCGI、PHP-FPM之间的关系

## CGI

CGI，通用网关接口(Common Gateway Interface)，或者叫做通用网关接口协议。可以让一个客户端从网页浏览器向服务器上的程序请求数据。CGI描述了描述了服务器和请求处理程序之间传输数据的一种标准。

CGI为web服务器动态生成web page提供了一套标准协议，就像在服务器上面执行命令行程序一样。这样的程序称之为CGI脚本或者CGIs。这个脚本具体如何执行取决于具体的服务器，通常来说当一个请求被执行的时候，该脚本被执行并且生成HTML.

### 历史

1993年，美国的NCSA([National Center for Supercomputing Applications](https://en.wikipedia.org/wiki/National_Center_for_Supercomputing_Applications))小组提出了调用命令行可执行文件的一些规范，一些其他的web服务器开发者采用了这个规范，从那时起这个规范就成了web 服务器的标准规范。在1997年一个小组更规范的定义了CGI，CGI Version 1.1。

历史上CGI脚本多使用C语言编写，主要是因为C语言能够更容易的访问环境变量。CGI的名称来自网络的早期，用户希望将数据库连接到他们的Web服务器。 CGI是由服务器执行的程序，它在Web服务器和数据库之间提供了一个通用的“网关”。

### CGI标准的目的

每个运行着HPPT服务的服务器，都会相应来自浏览器的请求。每一个HTTP服务都有一个文件夹，这些文件可以发送给连向这台服务器的浏览器。如果一个web 服务器的域名是example.com，它的文件存储在/usr/local/apache/htdocs，如果浏览器请求http://www.example.com/index.html，那么web 服务将会发送/usr/local/apache/htdocs/index.html给浏览器。

对于动态网页，服务器软件单独处理程序并将结果发送给客户端，这样的程序我们称之为脚本。这样的脚本程执行时有时候需要额外的信息，比如用户是否登录等等。

HTTP提供了给浏览器提供了向服务器传递这些信息的方法，服务器必须将这些信息传递给脚本程序。同时在返回的时候脚本必须提供HTTP所需要的所有信息以响应请求，比如状态码。

不同的服务器使用不同的方式和这些脚本程序交换这些信息，使用相同的脚本程序在不同的服务器之间工作时不太可行的。因此需要建立一个交换这些信息的标准，即CGI（定义了服务器与脚本交互的通用方法）。根据CGI标准调用的网页生成程序我们称之为CGI脚本，早期使用CGI是为了处理表单提交。

### 使用CGI脚本

一个web服务器可以配置哪些url使用哪些CGI脚本。当浏览器请求特定的url的时候，指定的CGI脚本程序就会执行，并将标准输出结果发送给浏览器。

可以看出，CGI定义了request中的附属信息如何传递给脚本程序，如果是GET请求，会把对应的信息存储在QUERY_STRING环境变量中。如果是POST请求，会议标准输入的形式传递给脚本程序，脚本会从标准输入中读取这些环境变量并且响应程序。

###CGI缺点

**应用程序的每次请求，都要引发一个全新的进程**。所以，服务器和网关之间的分离会造成性能的 耗费，会限制使用`CGI`的服务器的性能，并且会加重服务端机器资源的负担。



##FastCGI

CGI使外部程序与Web服务器之间交互成为可能。CGI程序运行在独立的进程中，并对每个Web请求创建一个进程，这种方法非常容易实现，但效率很差，难以扩展。面对大量请求，进程的大量创建和消亡使操作系统性能大大下降。此外，由于地址空间无法共享，也限制了资源重用。

与为每个请求创建一个新的进程不同，FastCGI使用持续的进程来处理一连串的请求。这些进程由FastCGI服务器管理，而不是web服务器。 当进来一个请求时，web服务器把[环境变量](https://zh.wikipedia.org/wiki/环境变量)和这个页面请求通过一个[socket](https://zh.wikipedia.org/wiki/Socket)(比如FastCGI进程与web服务器都位于本地）或者一个[TCP](https://zh.wikipedia.org/wiki/传输控制协议) connection（FastCGI进程在远端的[server farm](https://zh.wikipedia.org/w/index.php?title=Server_farm&action=edit&redlink=1)）传递给FastCGI进程。

### FastCGI特点

- FastCGI与语言无关
- FastCGI应用在进程中，独立于核心网络服务器，提供了一个比API环境更安全的环境。 APIs的代码和web服务器的应用的核心是 紧紧关联的。这也就意味着在API应用程序的错误可能会损坏其它应用程序或核心服务器。恶意API应用程序代码甚至可以窃取另一个应用程序或核心服务器密钥。
- FastCGI技术支持PHP,C/C++, Java language, Perl, Tcl, Python, SmallTalk, Ruby etc.. 它在Apache, ISS, Lighttpd和其他流行的 服务器中的相关模块都是可以使用的。FastCGI不依赖于任何服务器体系结构，所以即使服务器在技术上改变了，FastCGI还是稳定的

### FastCGI工作原理

- Web Server 启动时载入FastCGI进程管理器 (IIS ISAPI 或Apache Module)

- FastCGI进程管理器首先初始化自己，启动一批CGI解释器进程（可见多个php-cgi），然后等待来自Web Server的连接。

- 当Web Server中的一个客户端请求达到时， FastCGI进程管理器会选择并连接一个CGI解释器。Web server的CGI环境变量和标准输入会被送达FastCGI进程的php-cgi。

- FastCGI子进程从同一连接完成返还给Web Server的标准输出和错误信息。当请求进程完成后，FastCGI进程会关闭此连接。FastCGI会等待并出来来自FastCGI进程管理器（运行在Web Server中的）的下一个连接。 在CGI模式，php-cgi然后会退出。

  

  示意图：

![WX20190604-093729](/Users/iamcyan/Desktop/iamcyan/notebook/技术相关/技能树/WX20190604-093729.png)



## PHP-CGI

> PHP-CGI是PHP自带的FastCGI管理器。

- php-cgi变更php.ini配置后需重启php-cgi才能让新的php-ini生效，不可以平滑重启
- 直接杀死php-cgi进程,php就不能运行了。(PHP-FPM和Spawn-FCGI就没有这个问题,守护进程会平滑从新生成新的子进程。）

## PHP-FPM

FPM(FastCGI Process Manager)是PHP FastCGI运行模式的一个进程管理器。

在网络应用场景下，PHP并没有实现http网络库，而是实现了FastCGI协议，然后与web服务器配合实现了http的处理，web服务器来处理http请求，然后将解析的结果再通过FastCGI协议转发给处理程序，处理程序处理完成后将结果返回给web服务器，web服务器再返回给用户。



###FPM基本实现

概括来说，fpm的实现就是创建一个master进程，在master进程中创建并监听socket，然后fork出多个子进程，这些子进程各自accept请求，子进程的处理非常简单，它在启动后阻塞在accept上，有请求到达后开始读取请求数据，读取完成后开始处理然后再返回，在这期间是不会接收其它请求的，也就是说fpm的子进程同时只能响应一个请求，只有把这个请求处理完成后才会accept下一个请求。

fpm的master进程与worker进程之间不会直接进行通信，master通过共享内存获取worker进程的信息，比如worker进程当前状态、已处理请求数等，当master进程要杀掉一个worker进程时则通过发送信号的方式通知worker进程。

fpm可以同时监听多个端口，每个端口对应一个worker pool，而每个pool下对应多个worker进程，类似nginx中server概念。

在php-fpm.conf中通过`[pool name]`声明一个worker pool：

```
[web1]
listen = 127.0.0.1:9000
...

[web2]
listen = 127.0.0.1:9001
...
```

启动fpm后查看进程：ps -aux|grep fpm

```
root     27155  0.0  0.1 144704  2720 ?        Ss   15:16   0:00 php-fpm: master process (/usr/local/php7/etc/php-fpm.conf)
nobody   27156  0.0  0.1 144676  2416 ?        S    15:16   0:00 php-fpm: pool web1
nobody   27157  0.0  0.1 144676  2416 ?        S    15:16   0:00 php-fpm: pool web1
nobody   27159  0.0  0.1 144680  2376 ?        S    15:16   0:00 php-fpm: pool web2
nobody   27160  0.0  0.1 144680  2376 ?        S    15:16   0:00 php-fpm: pool web2
```



###php-fpm特点：

- PHP-FPM是一个PHP FastCGI管理器，是只用于PHP的,可以在 [http://php-fpm.org/download下载得到](http://php-fpm.org/download下载得到).
- PHP-FPM其实是PHP源代码的一个补丁，旨在将FastCGI进程管理整合进PHP包中。必须将它patch到你的PHP源代码中，在编译安装PHP后才可以使用。
- PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置，比spawn-fcgi具有更多有点，所以被PHP官方收录了。在./configure的时候带 –enable-fpm参数即可开启PHP-FPM。



## 参考

1、[cgi维基百科](https://zh.wikipedia.org/wiki/通用网关接口)

2、[cgi web项目发展](https://cloud.tencent.com/developer/article/1333686)

3、[fastcgi维基](https://zh.wikipedia.org/wiki/FastCGI)

4、[细说php-fpm](https://github.com/YuanLianDu/YLD-with-Php/blob/master/articles/php/php-fpm.md)

5、[PHP CGI, FastCGI, PHP-CGI and PHP-FPM](http://www.programering.com/a/MDOwADMwATA.html)]