# Go Module 



1、GO111MODUL=on 时，pkg 从哪个目录下面查找？

```
/Users/iamcyan/go/pkg/mod/cache/download
```



2、go mod init example.com/project_name

```
1、init 后面需要的就是一个字符串，并不是一个文件夹
2、执行 go build 后得到的命令行就是这个名字
```



3、如果我们引入了一些第三方的pkg，如何将这些pkg纳入到igt管理的中来

```
1、go mod vendor	//会在项目中创建一个vendor命令
2、使用 go build -mod=vendor 来构建命令。go build 默认是屏蔽vendor机智的。
```