# linux 面试题

1、查看linux 系统几颗物理 cpu 及 每个 cpu 的核数

```
cat /proc/cpuinfo 	//查看cpu信息
cat /proc/cpuinfo | grep -c 'physical id'
cat /proc/cpuinfo | grep -c 'processor'

-c：统计关键词出现的次数
```



2、查看系统负载常用的指令，及对应参数的意义

```
w			//显示当前用户
uptime	// load averanges 显示过去一分钟、五分钟、十五分钟内系统的平均负载
```



3、查看虚拟内存使用情况

```
vmstat (vm 即 virtual memory)

procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 1431980  30184 535304    0    0     6     5   23   31  0  0 100  0  0
 
procs:
	r:				//运行中的进程数量
  b:				//带带IO的进程数量
memory:
	swpd:			//使用虚拟内存的大小
	free:			//空闲物理内存大小
	buff:			//用作缓冲的内存大小
	cache:		//用作缓存的内存大小
swap:
	si				//每秒从交换区写到内存的大小，由磁盘调入内存
	so				//与si相反
io:
	bi				//每秒读取的块数
	bo				//每秒写入的块数
system:
 in					//每秒中断次数
 cs					//每秒上下文切换次数
cpu
	us				//user time 用户进程执行时间百分比
	sy				//system time 内核系统进程执行时间百分比
	id				//空闲时间百分比
	wa				//IO等待时间百分比
	st
	
	注：内存相关的单位是KB
```



4、linux 系统里面 buffer 和 cache 如何区分

```
buffer 和 cache 都是内存中的一块区域
cpu 数据写入到磁盘时，因速度差异，cpu 会先将数据写入到 buffer，再从 buffer 写入到磁盘
cpu 读取磁盘数据时，磁盘会先将数据写入到 cache，cpu 从 cache 读取数据
```



5、top 命令各行的意义

```
top - 15:33:35 up 18:40,  2 users,  load average: 0.13, 0.05, 0.01
Tasks: 102 total,   1 running, 101 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem :  2048092 total,  1431604 free,    50856 used,   565632 buff/cache
KiB Swap:  1003516 total,  1003516 free,        0 used.  1807572 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 2307 chengxi+  20   0   92836   3372   2444 S  0.3  0.2   0:03.26 sshd
    1 root      20   0   37816   5740   3892 S  0.0  0.3   0:02.87 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.32 kthreadd
    3 root      20   0       0      0      0 S  0.0  0.0   0:00.50 ksoftirqd/0
    5 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    
第一行信息：
15:33:35	//当前时间
up 18:40	//系统运行时间，单位是分
2 user		//当前登录用户数
load average 0.13 0.05 0.01	//系统负载 1、5、15 分钟前到现在的平均均值

第2、3 行信息
Tasks: 102 total		//进程总数
1 running						//正在运行的进程
101 sleeping				//睡眠进程
0 stoped						//停止进程数
0 zombie						//僵尸进程数
%Cpu(s) 0.3 us			//用户空间占用cpu百分比
0.3 sy							//内核占用cpu百分比
0.0 ni							//
99.0 id							//空闲cpu百分比
... 后续待补

第4、5行
Mem 2048092 total,  1431604 free,    50856 used,   565632 buff/cache
//内存使用情况：总量 - 空闲 - 已用 - buffer 区大小
Swap:  1003516 total,  1003516 free,        0 used.  1807572 avail Mem
//交换内存使用情况：总量 - 空闲 - 已用 - 待查

进程去情况
进程ID 用户 优先级 
PID 						//进程ID
USER      			//用户
PR  						//优先级
NI    					//nice 负值表示高优先级，正直低优先级
VIRT    				//进程使用虚拟内存总量，KB
RES    					//进程使用的未被换出的物理内存大小
SHR 						//共享内存大小，单位KB
S 							//进程状态 R=运行 S=睡眠 Z=僵尸 T=追踪/停止 D=不可中断的睡眠状态
%CPU 						//上次更新到现在CPU占用的百分比
%MEM     				//进程使用的物理内存比
TIME+						//进程使用cpu时间总计，单位 1/100 秒 
COMMAND					//命令名/命令行
```



6、如何实时查看网卡流量？如何查看历史网卡流量

```
需要按照 sysstat 包
sysstat -n DEV	//查看网卡流量，默认10min更新一次

可以查看指定日期的流量日志
```



7、如何查看当前系统有哪些进程

```
ps -aux 或者 ps -elf 	//ps(process status) 

-aux	//显示所有包含其他使用者的进程
-elf	//

chengxiaoshuo@ubuntu:~$ ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.2  37812  5740 ?        Ss   Apr02   0:03 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Apr02   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Apr02   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   Apr02   0:00 [kworker/0:0H]

chengxiaoshuo@ubuntu:~$ ps -elf
F S UID        PID  PPID  C PRI  NI ADDR SZ WCHAN  STIME TTY          TIME CMD
4 S root         1     0  0  80   0 -  9453 -      Apr02 ?        00:00:03 /sbin/init
1 S root         2     0  0  80   0 -     0 -      Apr02 ?        00:00:00 [kthreadd]
1 S root         3     2  0  80   0 -     0 -      Apr02 ?        00:00:00 [ksoftirqd/0]
1 S root         5     2  0  60 -20 -     0 -      Apr02 ?        00:00:00 [kworker/0:0H]


```



8、ps 查看系统进程时，有一列为STAT， 如果当前进程的stat为Ss 表示什么含义？如果为Z表示什么含义？

```
S: 休眠 Z:僵尸 s:主进程
```



9、如何看系统都开启了哪些端口？

```
需要使用netstat 包
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]

-a or --all								//显示所有连线中的socket
-l 或者 --listening：			//显示监控中的服务器的socket
-n 或者 --number：					//直接使用ip地址，而不是域名
-p 或者 --programs				//显示正在使用socket的程序识别码和程序名称
-t 或者 --tcp							//显示TCP传输协议的连线情况
-u 或者 --udp							//显示UDP传输协议的连线情况
-i or --interface					//显示网络界面信息表单

例子：
netstat -lnp							//查看系统开启了哪些端口 (显示监听的套接口)
netstat -an								//查看网络连接情况
netstat -apu							//查看UDP端口使用情况
netstat -s								//显示网络统计信息
```



10、如何查看网络连接情况

```
netstat -an
```



11、查看当前的主机名

```
hostname
hostname name1	//将主机名修改为name1
```



12、显示开机信息。(网卡或者硬盘有问题时，可以通过哪个命令查看相关信息)

```
dmesg
```



13、在linux系统下按照下面的要求抓包：只过滤出访问http服务的，目标ip为192.168.0.111，一共 1000 个包，保存在 1.cap文件中

```
tcpdump -nn -s0 host 192.169.0.111 and port 80 -c 1000 -w 1.cap
```



14、有一天突然发现网站速度很慢，如何处理

```
可以两个方面分析：系统负载，网卡流量分析
1、使用 w 命令或者uptime命令查看系统负载，如果负载高，使用top 命令查看cpu mem等问题，看是否cpu繁忙或者内存占用高。
2、使用sar命令分析网卡流量，分析是否遇到共计。
```

























##### 1、查询指定时间段的日志

```
grep -E '2018/07/10 12:0[0-9]|2018/07/10 12:10:00' quant_yii_access.log
```



##### 2、统计ip出现的次数

```
日志格式:
2018/06/07 10:22:01 [error] 2548#0: *9540395 connect() failed (110: Connection timed out) while connecting to upstream, client: 10.27.82.172, server: www.joinquant.com, request: "POST /api/aistare.postAistareTask HTTP/1.1", upstream: "fastcgi://127.0.0.1:9001", host: "10.27.82.172"

cat nginx_error.log| awk -F ',' '{print$5}'|sort|uniq -c|sort -k1nr| head -10
```

##### 3、统计某个关键字出现的次数

```
grep -o 127.0.0.1 nginx_error.log | wc -l
grep -o '127.0.0.1\|60.205.249.89'  nginx_error.log | wc -l
wc -l(行数) -c(字节) -w(单词数)
```

##### 4、查看某一个端口是否被占用，被谁占用？

```
方法1：
	lsof -i:80
2:
	netstat -anp | grep 80
	
	
```

##### 5、服务器负载太高，如何定位

```
1、top,找出cpu使用最高的程序
2、找到对应的进程之后，使用ps -mp PID号 -o THREAD,tid,time | sort -rn
	定位到哪一个线程出了问题

```

