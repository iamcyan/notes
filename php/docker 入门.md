#docker 入门

## 一、容器是什么

## 二、docker 是什么

## 三、docker 安装

安装

```
mac 环境下安装
```

检验安装是否成功

```
docker version
or 
docker info
```

docker 是服务器-客户端架构，命令行运行docker命令的时候，需要本机有docker 服务。使用下面命令启动docker 服务

```
# service 命令的方法
sudo service docker start

# systemctl 命令的用法
sudo systemctl start docker.
```



## 四、image 文件

**docker 把运用程序及依赖打包在image文件里面。**只有通过这个文件才能生成docker容器，image 可以看做是容器的模板。同一个image 可以生成多个docker实例。

image 文件时一个二进制文件，通常是继承自另外一个image文件，然后加上一些个性化的设置。

```
# 列出本机所有的 image 文件
docker image ls

# 删除 image 文件
docker image rm [iamge_name]
```

iamge 文件时通用的，一般来说可以在机器之间相互拷贝。为节省时间我们应该尽量使用别人制作好的image文件，而不是自己制作。定制的也应该在别人的基础上进行加工。

docker 的官方仓库 [docker_hub]() 是最重要的。



## 五、实例：hello world

```
# 拉取官方的 hello-world image 文件
docker image pull library/hello-world

# docker 官方的镜像都会放在 library 下，故上面的命令可以简化为：
docker image pull hello-world

# 运行 image 文件
docker container run hello-world

#docker container run 命令具有自动抓取 iamge 文件的功能，如果发现没有指定的 iamge 文件，会自动执行抓取，故上面的docker image pull 命令是非必须的。

# hello-world 容器运行之后自动停止，不能自动停止的容器，如果想结束可以使用下面的命令:
docker container kill [container_id]
```



## 六、容器文件

image 文件生成的容器也是一个文件，称为容器文件。关闭容器并不会删除容器文件，只是容器停止运行而已。

再次生成容器实例的时，原有的容器实例文件会被覆盖。

```
# 列出正在运行的容器实例
docker container ls

# 列出本机上所有的容器，包括终止运行的容器
docker container ls -all

# 移除容器文件
docker container rm [container_id]
```



## 七、Dockerfile 文件

Dockerfile 是一个文本文件，根据这个文件生成二进制的iamge



## 八、制作自己的docker 容器

### 编写dockerfile 文件

创建 .dockeringnore 文本文件，评出不需要打包进入 image 的文件，如果没有此需求可以不新建。

```
.git
```

新建一个文本文件，写入如下内容：

```
FROM node:8.4
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
```

解释上述命令

```
FROM node:8.4：该 image 文件继承官方的 node image，冒号表示标签，这里标签是8.4，即8.4版本的 node。
COPY . /app：将当前目录下的所有文件（除了.dockerignore排除的路径），都拷贝进入 image 文件的/app目录。
WORKDIR /app：指定接下来的工作路径为/app。
RUN npm install：在/app目录下，运行npm install命令安装依赖。注意，安装后所有的依赖，都将打包进入 image 文件。
EXPOSE 3000：将容器 3000 端口暴露出来， 允许外部连接这个端口
```



### 创建 iamge 文件

有了 Dockerfile 之后，可以使用 docker iamge build 名来来创建 iamge 文件

```
docker image build -t koa-demo .
or
docker image build -t koa-demo:0.0.1 .
```

-t：用来指定image 文件的名字，用冒号指定标签，如果不指定，默认的标签是 latest。. 表示Docker文件所在的路径。



###生成容器

````
docker container run -p 8000:3000 -it koa-deomo /bin/bash
# or
docker container run -p 8000:3000 -it koa-deomo:0.0.1 /bin/bash
````

参数释义：

```
-p：容器的3000 端口映射到本机的8000端口
-it：容器的 shell 映射到当前的 shell, 本机窗口输入的命令会映射进容器。
koa-demo:0.0.1：image文件的名字
/bin/bash：容器启动以后，内部执行的第一个命令。这里启动bash，保证用户可以使用shell.
```



### CMD命令

```
FROM node:8.4
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
CMD node demos/01.js
```



CMD && RUN 的区别：

```
cmd 命令是在容器启动之后执行的，run 是在 image 构建的过程中执行的
一个Dockerfile文件中，可以有多个run命令，但是只能有一个cmd命令
```



### 发布 iamge 文件

在命令行登录

```
docker login
```



为本地的 image 标注用户名和版本

```
docker image tag [image_name] [user_name]/[repository]:[tag]
# 实例
docker image tag koa-demo:0.0.1 ruyif/koa-demo:0.0.1

# 也可以不标记，重新构建一个 image
$ docker image build -t [username]/[repository]:[tag] .
```

发布 image 文件

```
docker image push [user_name]/[repository]:[tag]
```



## 九、其他有用命令

**docker container start**

可以用来启动已经生成、停止的容器文件

```
docker container start [container_id]
```



**docker container stop**

前面的`docker container kill`命令终止容器运行，相当于向容器里面的主进程发出 SIGKILL 信号。而`docker container stop`命令也是用来终止容器运行，相当于向容器里面的主进程发出 SIGTERM 信号，然后过一段时间再发出 SIGKILL 信号。

```
docker container stop [container_id]
```



**docker container logs**

用来查看docker容器的输出，即容器里面的shell的标准输出。如果启动容器的时候没有使用 `-it`这个参数，就需要这个名来来查看输出的

````
docker container logs [container_id]
````



**docker container exec**

进入容器，如果容器运行的时候，没有使用`-it`这个参数，就需要使用此名来进入容器。进入容器后可以执行shell命令

```
docker container exec -it [container_id] /bin/bash
```



**docker container cp**

用于将正在运行的docker容器里里面的文件拷贝到本机

```
docker container cp [container_id]:[/path/to/file] .
```





