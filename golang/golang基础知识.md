# golang 基础知识

## go 程序加载执行顺序



## 数据结构

go的数据结构可以分为4类

- 基础类型
- 聚合类型
- 接口类型
- 引用类型

### 

### 数值类型

整数：

```
1、有符号和无符号整型
有符号整型 ：int8、int16、int32、int64，
无符号整型：uint8、uint16、uint32、uint64

2、int 和 uint
两种大小相同都是32位或者64位。根据硬件平台不同而不同，在同样的平台上不同的编译器可能选择也不同
注：int 和 int32 是两种类型，int 值做 int32 使用，需要显式转换，反之亦然。

rune 和 byte
rune 是 int32 的同义词，常常用来指明一个值是unicode，两个类型可以互换。
byte 类型是 uint8 的同义词，强调一个值是原始数据而非量值。	//什么叫强调是原始数据
```



运算符：

```
1、二元运算符的五个级别
*		/		%		<<		>>		&		&^
+		-		|		^ 		
==	!=	<		<=		>			>=
&&
||

2、+ -	*	/ 可以用于整数、浮点、复数的运算，% 只能用于整数。
3、% 的结果的正负号总是与被除数有关，/ 结果取决于操作数是否都为整数，整数相除舍去小数部分。
4、位运算符：&		|		^		&^		<<		>>
^作为一元运算符的的时候，标识按位取反，二元运算符的时候，标识异或。00000110 ^ 00100010 => {1,2} ^ {1,5} = {2,5}
&^ 标识and not，同理上述也可以使用集合来表示：00000110 ^ 00100010 => {1,2} &^ {1,5} = {2}
```



类型转换的一些注意事项：

```
1、整形<=>整形的时候，不会引起值的变化，只是告知编译器如何解读值
2、缩减大小的整形以及浮点和整形的转换，可能会改变值或者损失精度。
```



使用整型的一些注意事项

```
1、x<<n，其中n必须是无符号整型
2、无符号整型往往只用于位运算和特定算术运算符   极少表示非负值，即可值不可能为负，原因：i==0时，i--会表示对应的无符号类型的最大值如2^64-1
```

浮点类型和复数类型一般使用的比较少，暂时不做整理。



### 字符串

定义：字符串是不可变的字节序列，可包含任何数据，包括0值字节，主要是人类可读的为本。习惯上文本字符串被解读称按照UTF-8编码的Unicode码点序列。

字符串

```
1、可以访问某个字节：s[1]
2、生成字符串子串：s[1:3]，作训左闭右开的原则
3、可以使用 "+" 连接两个字符串：s += "good"
4、字符串不可变，如：s[1] = "a"，操作非法
5、字符串和截取生成的子字符串，公用底层内存，因此生成子字符串的开销比较低廉。
```



Unicode 

```
Unicode
1、最早的计算机只需要处理单一字符集 ASCII，用7位表示128个字符。
2、随着计算机发展，需要表示的字符超出了ASCII 的范围，Unicode被用来表示世界内的全部字符，go 语言中这些文字记号称作文字符号（rune）
3、文字符号目前总数在12W左右，那么int32就天然的符合作为存储单个文字符号的数据类型，正因为如此，rune类型作为int32的别名
4、多数情况下，文本只需要一个字节就够了，广泛使用的字符数目也不超过2^16，所以有了UTF-8的解决方案 
```

UTF-8

```
1、UTF-8 是对 Unicode 码点做变长编码的一种格式。
2、每个Unicode码点使用1~4个字节表示，规则如下：
0xxxxxxx 																	0-127				ASCII	
110xxxxx	10xxxxxx												128-2047
1110xxxx	10xxxxxx	10xxxxxx							2048-65535
11110xxx	10xxxxxx	10xxxxxx	10xxxxxx		65536-2^32-1
3、使用UTF-8字节编码的顺序就是文本的顺序，因此对UTF-8排序，就是对文本排序
4、可以很简单的判断某个字符是不是指定字符串的前缀后缀：len(prefix) > 0 && s[:len(prefix)] == prefix，后缀同理
5、如果需要逐个的处理Unicode字符，那么必须关注他的编码方式，如下：
s := "hello, 世界"
fmt.Println(len(s))	//13
fmt.Println(utf8.RuneCountInString(s))	//9
6、utf8 中的另外一个用来处理逐个unicode的函数：r,size := utf8.DecodeRuneInString(s[i:]).r 符号本身，size 占用的字节数，字节数用来更新i
7、有一个简单的方法，range 函数也可以遍历字符串，按照utf-8隐式解码 for i, r := range s {}
8、字符串和slice之间转换
s := "中国"
r := []rune(s)
s1 := string(r)
```

字符串操作的常见包

```
strings
unicode
bytes
strconv

这些包都需要看一下，做到对基础的字符串操作有数。
注：字符串转换为数值可以使用
fmt.Sprintf("%d", int_value)	//整数转字符串
```

常量

```
定义：
const PI = 3.14

常量生成器iota
const = (
value_0 = iota
value_1				//1
value_2
value_3
value_4
)
```





###数组

1、定义数组的几种方法

```
//方法1：
var arr [5]int
//初始化数组
arr := [3]int{1,2,3}
```

2、数组有什么特性

```
1、数组长度不可变
2、相同的元素类型
2、数组的长度是类型的一部分，所以[2]int 和 [3]int 是不同的类型
3、在 go 中，当调用一个函数时，每个传入的参数都会创建一个副本，数组也比例外。函数内的修改不影响到外部，如果想修改传入的数组参数，需要使用指针传递。
```

3、操作数组的基本函数

略：实际中很少使用到数组。

4、demo

略：实际中很少使用到数组

### slice

1、什么是slice，slice 和数组的关系，slice 有什么特性

```
1、slice 是拥有相同类型元素的可变程度的序列。
2、slice 的三个属性：指针、长度、容量。其中，指针执行数组的第一个可以从slice中访问的元素与，长度是slice的长度，len(s)，容量是指的从指针指向的第一个数组元素到最后一个数组元素的长度，cap(s)
3、因为slice的三要素中包含了指向数组的指针，所以传递一个slice给函数内部时，内部变动会引起外部变动。
4、s[i:j],左闭右开，其中 s 可以是数组、指向数组的指针、slice.
5、对于一个字符串和一个字节序列slice,都可以使用x[i:j]，区别是如果x是字符串，那么返回的结果是字符串，如果x是字节序列，那么返回的是子一个字节序列slice
6、slice 是无法比较的，虽然可以比较某些特殊情况的slice，但是通常意义上的比较是没有意义的。
7、判断一个slice是否为空的时候，不能使用s != nil，因为不等于nil的情况下，也有可能为空，如下
var s []int		//len(s) == 0, s == nil
s = nil			//len(s) == 0, s == nil
s = []int{nil}	//len(s) == 0, s == nil
s = []int{}		//len(s) == 0, s != nil
8、slice 最大的访问长度为s[len(s)-1]，即使len(s) < cap(s)也是如。当从一个当从一个slice1扩展出另外一个slice2时，slice2的范围为 [cap-len, cap],slice2 中的起始元素的inde不能比slice1中的更小。
```

2、声明slice、初始化一个slice

```
var s []int
s := []int{}
s := []int{1,2,3}
s := make([]int, len)
s := make([]int, len, cap)	
注：make 会创建一个无名的底层数组，返回一个指向该数组的slice，只能通过此slice来访问。
```

3、slice 基本操作函数

```
//append函数 
runes2 = append(runes1, r)
1、在 slice 的末尾添加一个元素。如果没有超过slice的容量，那么会直接在slice末尾进行增加，同事会改变底层数组的值。
2、如果添加的元素超过了数组的容量，那么就会重新生成一个底层数组，runes2指向新的底层数组，rune1的值保持不变。
3、有一点需要注意的是：不能假定原始的slice和调用append之后的slice指向同一个数组，也无法证明他们执行不同的底层数组，不能假设旧slice的操作是否会影响新的slice上面的值。通常做法是讲append的调用结果赋值给传入append的slice

//copy函数 copy(dest, src)
1、用来为两个拥有相同元素的slice复制元素
2、copy 函数的返回值为复制的个数
3、返回值是赋值的个数，个数的取值是两个slice中比较小的那个，所以不会出现索引越界的问题
```

4、[]rune 和 []byte的区别联系

```
在数值类型那一小节可以知道：
1、rune 是 int32 的别名，byte 是 uint8 的别名。
2、[]rune <=> []int32, []byte <=> []uint8
3、如果需要截取中文字符，那么可以先把中文字符转换成 []rune,截取之后在使用string()转换成字符串。
```

### map

1、如何定义一个map类型，有哪些特点

```
1、在go语言中，map是散列表的引用
2、map的类型是map[k]v，其中k的值必须是可比较的值，一般为整形、字符串，v 可以是任何一种类型
3、map 在设置元素之前必须进行初始化
4、map的元素不能获取地址，一个原因是map的厂部是可变的，map长度变化之后，元素可能进行重新散列，这样可能会使地址无效
5、map 中的元素是无效的
```



2、map 类型相关的基本函数

```
//delete 删除一个元素，即使键不存在也是安全的
delete(m, k)	

//检测一个元素在map中是否存在
if v, ok := map[k]; !ok {/***/}
```



3、map类型的使用场景有哪些

```
1、go 没有提供集合类型，因为map的key是不重复的，所以可以使用map的key值来替代集合类型
2、有时候需要一个map并且键为slice,那么可以通过某种方法将slice转换为字符串key。对于任何不能比较的键类型，都可以使用这个思路进行转换。
3、当map的值为符合类型的时候，可以采用延时初始化的思路：
var m1 = make(map[string]map[string]bool)
v := m1[from]
if v == nil {
	v = map[string]boo
	m1[from] = v
}
```



###结构体

1、什么是结构体

```
结构体是将零个或多个任意类型的命名变量组合在一起的聚合数据类型。经典实例就是员工信息
```

2、知识点 && 基本语法

```go
type E struct {
  ID	int
  Name	string
}
//声明变量
var e E				//e 为变量均为对应类型的零值
e := new(E)		//e 为一个变量的指针，变量类型为E。
e := E{1, "NAME"}	//按照变量顺序声明变量，一般不这样使用，结构体变量一般较多
e := E{ID:1, Name:"NAME"}			//通过k:v的形式，来声明全部或者部分变量

//指针
type E struct {
	ID	int
	Name	string
}
func main()  {
	e := E{ID:1, Name:"cheng"}
	e.Name = "iamcyan"
	fmt.Println(e, &e, *&e.Name)		//输出：{1 iamcyan} &{1 iamcyan} iamcyan
}
&：获取变量的指针地址
*：出现在变量前面时，为获取指针所指向的值。出现在函数参数中时，表明需要传递指针类型。

//结构体的成员变量不能够是结构体本身，但是可以是指向结构体的指针。用此特性可以创建一些能够地沟递归的数据结构，如二叉树、链表。
//例子：自行实现二叉树的插入排序。

//结构体嵌套和匿名成员
type Point struct {
  X	int
  Y	int
}
type Circle struct {
  Point
  Radius int
}
var c Circle
c.X = 100
```



3、使用场景

```
1、处于效率考虑，大型结构体通常使用结构体指针的方式直接传递给函数或者从函数中返回。
```



##函数(function)

1、go 中函数由哪些知识点

```
1、形参是函数的局部变量，实参都是按值传递的，如果实参是引用类型，那么当函数使用形参变量的时候可能会间接的改变实参变量。
2、函数可以有过个返回值，通常会有一个参数标识错误
3、函数签名
	a、函数的类型可以称为函数签名
	b、当两个函数拥有相同的形参列表和返回列表时，认为两个函数的类型或者签名相同。
```

2、函数变量

```
1、函数变量：一个变量的值是一个函数
2、函数标量可以作为参数进行传递，如string.Map(func(){}, string)
```

3、错误处理机制

```
go 使用通常的控制流机制来应对错误(例如if 和 return)，这就要求在逻辑处理方面更加谨慎。

1、将错误直返回
2、合理重试
3、输出错误，停止程序，单这种逻辑一般留在调用的主程序中。例如库函数的错误通常传递给调用者。记录日志可以借助log.Fatal或者log包的其他的函数
4、使用log包可以自定义的错误前缀
5、直接忽略错误
```

4、匿名函数

```go
//1、函数字面量：字面量单词为literal，含义为表面的。可以简单的理解为 = 右边的值。
//2、匿名函数举例：string.Map(func(r rune) rune {return r+1}, "name")
//3、匿名函数举例：
func square() func() int {		//square 定义的返回类型是：func() int，参见函数签名定义。
	var x int
	return func() int {
		x++
		return  x*x
	}
}
func main()  {
	f := square()
	fmt.Println(f())	// 1
	fmt.Println(f())	// 4
	fmt.Println(f())	// 9
}
//从这个例子中，需要注意的知识点：函数变量不仅是一段代码，还可以拥有状态。这些隐藏的变量，就是我们把函数当做引用类型的原因。

```

5、变长函数

```go
//如下：
func test(args ...int) {
  for _,val := range args {
    
  }
}
```



6、延迟调用 defer

```
1、defer 调用的函数，会在方法的最后时候执行。
2、打开一个资源时，需要定义与之对应的关闭方法
3、多个defer的时候，先定义后调用。
```



7、捕获迭代变量中需要注意的事项

```go
var funcSlice1 []func()
var funcSlice2 []func()
func main()  {
	s := []int{1,2,3}
	for k,val := range  s {
		v := val
		fmt.Println(v, val, &v, &val, &s[k])
		funcSlice1 = append(funcSlice1, func() {
			fmt.Println(val * val)
		})
		funcSlice2 = append(funcSlice2, func() {
			fmt.Println(v*v)
		})

	}
	for _,value :=range funcSlice1 {
		value()
	}
	for _,value :=range funcSlice2 {
		value()
	}
}

//输出：
1 1 0xc00008c010 0xc00008c008 0xc00008e020	//for 循环时的变量，分配了相同的地址，每次值会变动。
2 2 0xc00008c040 0xc00008c008 0xc00008e028
3 3 0xc00008c058 0xc00008c008 0xc00008e030
9
9
9
1
4
9
```



函数使用demo

```go
//递归


//深度优先的搜索

//广度优先的搜索
```



## 方法

一、基本点

```
1、什么是方法：
	a、方法是定义在某一个类型上面的，而函数是定义在包层面的。方法是可以绑定到任何类型上的，除指针类型和接口类型。
	b、对于指针这一点需要明确，指针类型不能定义方法，但是可以将方法的接受者设定为指针类型。
		
2、在一个包内，使用type定义的每一个类型相当于有自己的命名空间，在不同的类型上可以绑定同名的方法
3、方法接受者设为指针类型：func(p *Point) Distance(){}
4、如果一个类型的方法，接收这有指针类型，那么通常来说，应该将所有的方法都变换成指针接收。
```

二、面向对象

```
组合：
1、个人理解组合的实现实际就是结构体类型的嵌套，其中嵌套的结构体上面定义了方法。


封装：
1、封装的核心含义其实就是可见性，故封装的单元是包
2、要封装一个对象，必须使用结构体。
3、封装的好处：
	a、调用方不能直接修改对象的变量，所以不需要检查变量的值
	b、更加灵活的设计API
	c、防止调用这肆意改变变量的值。
```





## 接口类型

一、举例说明接口类型

```go
type Person struct {
	Name string
	Age  int
}
func (p Person) String() string {
	return fmt.Sprintf("%s: %d", p.Name, p.Age)
}
// ByAge implements sort.Interface for []Person based on
// the Age field.
type ByAge []Person //自定义

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }

func main() {
	//此时声明的people类型只是[]Person,不是ByAge
	people := []Person{
		{"Bob", 31},
		{"John", 42},
		{"Michael", 17},
		{"Jenny", 26},
	}
	fmt.Println(people)
	sort.Sort(ByAge(people))	//将people的类型转换成ByAge类型。
	fmt.Println(people)
}

//关键：sort.Sort(a interface) 参数需要实现一个如下的Interface
// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package. The methods require that the
// elements of the collection be enumerated by an integer index.
type Interface interface {
	// Len is the number of elements in the collection.
	Len() int
	// Less reports whether the element with
	// index i should sort before the element with index j.
	Less(i, j int) bool
	// Swap swaps the elements with indexes i and j.
	Swap(i, j int)
}
```



二、类型断言

> 类型断言：作用在接口值上的操作，写法类似x.(T)，其中x是一个接口类型表达式，而T是一个类型。会检查x的类型是非为T

```go
	var w io.Writer = os.Stdout
	f, ok := w.(*os.File)
	f1, ok1 := w.(*bytes.Buffer)
	fmt.Println(f, ok, f1, ok1)	//&{0xc000088060} true <nil> false
```



三、类型分支

接口的两种风格：

​	1、如排序例子中提到的，接口中定义了需要实现的方法

​	2、第二种风格，利用接口可以容纳各种具体类型的能力，把接口作为这种类型的union使用

```go
// 参数只是指明了interface，个人理解是可以为各种类型的参数
func main() {
	fmt.Println(test(10))
	fmt.Println(test("aaa"))
	fmt.Println(test(10.1111))
}

func test(a interface{}) string {
	switch a.(type) {
	case string:
		return  fmt.Sprintf("%s", a)
	case int:
		return  fmt.Sprintf("%d", a)
	case float64:
		return  fmt.Sprintf("%f", a)
	default:
		return ""
	}
}
```



##通道(channel)

一、goroutine

```
1、goroutine 类似于线程，但是在数量层面比线程大很多
2、当程序启动的时候，只有一个goroutine调用main函数，称为主goroutine.使用go关键字开启一个新的goroutine，如：go func()
3、除了从main函数返回或者退出之外，没有程序化的方法让一个goroutine来停止另一个goroutine.
```



二、chan

```
1、通道可以使一个goroutine发送消息到另外一个goroutine
2、ch := make(chan int)，生成了一个int类型的chan。复制或者当做参数传递到函数的时候，使用的是引用类型。
3、chan的操作：
	发送(send)：ch <- x
	接收(receive)x = <- ch
	关闭(close)：close(ch)
4、无缓冲通道
	a、ch := make(chan int) 或者 ch := make(chan int, 0)
	b/无缓冲通道上的发送操作将会阻塞，知道另一个goroutine进行接收。反之亦然。
	c、没有一个直接的方式来判断通道是否已经关闭，可以使用接收操作的一种变种：v, ok := <- ch。ok 为true，接收成功，false 时表示通道已经关闭。
	d、range 方法可以在通道上进行迭代
5、单向通道类型
	a、一个通道类型可以转换为单向通道，即只进、或者只出
	b、ch := make(chan int)	func t(ch chan<- int) {}，当把ch作为参数传递进入到t时，ch转换为只进的通道类型。
	c、单向通道类型不能转换为普通通道类型
6、缓冲通道：	
	a、ch := make(chan int, 10)
	b、通道满时堵塞，和无缓冲通道表现一致。
	c、不能使用缓冲通道作为简单队列使用，当消费出现问题的时候，可能导致写入永久堵塞。
	d、当一个goroutine向一个已经关闭的chan写入数据的时候，会造成goroutine泄漏。
```



三、select 多路复用

```
1、select 可以让当前的goroutine 等待多个chan 的可读或可写状态，在多个文件或者channel发生改变前，select 会阻塞当前进程或者goroutine，如果select 中提供了default逻辑，那么会执行default中的逻辑.
2、select 能够在channel上进行非阻塞的收发操作
3、多个case同时发生时，会随机选择一个case执行
```

Demo1：并发时钟服务器

```go
func main() {
	//监听8000端口
	listener, err := net.Listen("tcp", "127.0.0.1:8888")
	if err != nil {
		log.Fatal(err)
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err)
			continue
		}
		go handleconn(conn)
	}
}

func handleconn(conn net.Conn) {
	defer conn.Close()
	for {
		_, err := io.WriteString(conn, time.Now().Format("15:04:05\n"))
		if err != nil {
			return
		}
		time.Sleep(1*time.Second)
	}
}
```

Demo2：select  

```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}
```



## 使用共享变量实现并发

一、数据竞态

> 两个goroutine并发读写一个变量的时候，至少其中的一个goroutine会修改变量

有三种方法可以避免数据静态

1. 不修改变量
2. 避免从多个goroutine访问同一个变量
3. 允许多个goroutine访问同一个变量，但是同一时刻只能有一个goroutine访问变量。



二、互斥锁

```
使用容量为1的channel来实现，保证同一时刻只能有一个goroutine来访问变量
var ch = make(chan struct{}, 1)
ch <- struct{}{}	//获取令牌
do somethings ....
<- ch		//释放令牌

使用sync 提供的互斥锁
sync.Lock
sync.UnLock

读锁
sync.Rlock
sync.RUnLock

延迟初始化 
sync.Once
```

三、内存同步

计算机执行的并发操作和我们想的多个程序同时执行有很大的区别。

编译器可能认为两个语句的执行顺序无关，然后交换了两个语句的执行顺序。多核CPU也存在这个问题，不同的goroutine在不同的核上执行的时候，对于变量的修改只是保存在了CPU的缓存，还没有同步到内存，对于另外一个goroutine来说，对于变量的修改是不可见的。



##json



### fmt

unicode/utf8



##小练习

使用结构体来实现二叉树的插入排序



##参考资料：

[go 语言设计与实现](https://draveness.me/golang/)

