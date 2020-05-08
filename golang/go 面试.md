# go 面试

##语言

#####1、select 是随机的还是顺序的

```
select 在 goroutine 上进行非阻塞的收发操作。
当 chan 的可读或者可写状态改变的时候，执行对应的case
如果两个 case 同事满足，随机执行其中的一个。
```



#####2、Go 语言的局部变量分配在栈还是堆？

```
a、对于变量，编译器可以使用堆或栈来分配空间，分配为堆还是栈，不依赖于变量的生命方式
b、局部变量使用栈来存储，如果局部变量发生逃逸，那么变量会使用堆来存储，并且变量逃逸的时候会重新分配内存。
c、总结b可得，局部变量使用栈存储，发生逃逸时使用堆来存储，包变量使用堆来存储。
```



#####3、简述一下对go垃圾回收机制的了解

```
a、包级别的变量或者当前执行函数的局部变量，可以作为追溯该变量的源头，通过指针或者其他引用方式可以找到变量。如果变量路径不存在，那么变量不可访问，故其不影响任何计算过程，可回收。
b、go 中的GC 基于三色标记算法，后加入写屏障的。
```

三色标记算法示意图：

```
1、首先把所有的对象都放到白色的集合中
2、从根节点开始遍历对象，遍历到的白色对象从白色集合中放到灰色集合中
3、遍历灰色集合中的对象，把灰色对象引用的白色集合的对象放入到灰色集合中，同时把遍历过的灰色集合中的对象放到黑色的集合中
4、循环步骤3，知道灰色集合中没有对象
5、步骤4结束后，白色集合中的对象就是不可达对象，也就是垃圾，进行回收

```



![img](/Users/iamcyan/Desktop/iamcyan/note/golang/三色标记算法.png)



##### 4、简述一下golang的写成调度原理

##### [golang 协程调度-基本原理与初始化](https://github.com/LeoYang90/Golang-Internal-Notes/blob/master/Go%20%E5%8D%8F%E7%A8%8B%E8%B0%83%E5%BA%A6%E2%80%94%E2%80%94%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86%E4%B8%8E%E5%88%9D%E5%A7%8B%E5%8C%96.md)



##### 5、介绍一下golang 的runtime 机制

[runtime 机制](https://reading.developerlearning.cn/interview/interview-golang-language/)



##### 6、如何获取go运行时的协程数量、gc时间，堆栈使用

[获取go程序运行时信息](https://reading.developerlearning.cn/interview/interview-golang-language/)



##### 7、golang debug

1、panic。

2、pprof。

3、go run - race 竞争检测

4、查看cpu、磁盘、内存的使用情况



##### 8、make 和 new 的区别

make用于内建类型（map、slice 和channel）的内存分配。new用于各种类型的内存分配。

内建函数new本质上说跟其它语言中的同名函数功能一样：new(T)分配了零值填充的T类型的内存空间，并且返回其地址，即一个*T类型的值。用Go的术语说，它返回了一个指针，指向新分配的类型T的零值。有一点非常重要：**new返回指针。**


内建函数make(T, args)与new(T)有着不同的功能，make只能创建slice、map和channel，并且返回一个有初始值(非零)的T类型，而不是*T。



