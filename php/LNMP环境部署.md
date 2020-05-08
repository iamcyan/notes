# LNMP环境部署

>  OS：Ubuntu 18 LTS
>
> 源码：/usr/local/src

##安装依赖

####0、安装依赖

> apt-get install build-essential libtool gcc automake autoconf make libpng-dev libtidy-dev libzip-dev libxml2-dev

####1、pcre安装

> 1、wget https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz
>
> 2、tar -zxvf pcre-8.40.tar.gz
>
> 3、./configure && make && make install

####2、安装zlib

> 1、wget https://www.zlib.net/zlib-1.2.11.tar.gz
>
> 2、tar -zxvf zlib-1.2.11.tar.gz
>
> 3、./configure && make && make install

####3、安装openssl

> 1、wget https://www.openssl.org/source/old/1.1.1/openssl-1.1.1.tar.gz
>
> 2、tar -zxvf openssl-1.1.1.tar.gz
>
> 3、./configure && make && make install



##安装Nginx

>1、wget https://nginx.org/download/nginx-1.14.2.tar.gz
>
>2、tar -zxvf nginx-1.14.2.tar.gz
>
>3、./configure --prefix=/usr/local/nginx --pid-path=/usr/local/nginx/logs/nginx.pid --error-log-path=/usr/local/nginx/logs/error.log --http-log-path=/usr/local/nginx/logs/access.log --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.43 --with-zlib=/usr/local/src/zlib-1.2.11 --with-openssl=/usr/local/src/openssl-1.1.1
>
>4、make && make install

## PHP安装

> 1、wget -O php7.3.tar.gz  http://php.net/get/php-7.3.3.tar.gz/from/this/mirror
>
> 2、tar -zxvf php7.3.tar.gz
>
> 3、./configure --prefix=/usr/local/php7.3 --enable-fpm --enable-mysqlnd --enable-mbstring --enable-bcmath --enable-mbstring --enable-pcntl --enable-soap --enable-zip --enable-calendar --enable-ftp --enable-intl --with-curl --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-openssl --with-pdo-mysql --with-xmlrpc --with-gd --with-curl --with-mcrypt --with-tidy --enable-wddx
>
> 4、make && make install



## MySQL安装

### 安装依赖

#####安装依赖cmane、bison、libcurses-5dev

> apt-get install cmake bison libncurses5-dev

##### 安装boost库

>

