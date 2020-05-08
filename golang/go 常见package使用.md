#go 常见package使用

## fmt

```go
i := 23
	fmt.Println(i)	//23
	fmt.Printf("%v\n", i)	//23
	fmt.Printf("%d\n", i)	//23

	fmt.Printf("%T %T\n", i, &i)		//int *int

	t := true
	fmt.Printf("%v %t\n", t, t)	//true true

	answer := 42
	fmt.Printf("%v %d %x %o %b \n", answer, answer, answer, answer, answer)	//42 42 2a 52 101010

	pi := math.Pi
	fmt.Printf("%v %g %.2f (%6.2f) %e\n", pi, pi, pi, pi, pi)
	//3.141592653589793 3.141592653589793 3.14 (  3.14) 3.141593e+00

	smile := '😁'
	fmt.Printf("%v %d %c %q %U %#U\n", smile, smile, smile, smile, smile, smile)
	//3.141592653589793 3.141592653589793 3.14 (  3.14) 3.141593e+00
	//128513 128513 😁 '😁' U+1F601 U+1F601 '😁'

	str := `foo "bar"`
	fmt.Printf("%v %s %q %#q\n", str, str, str, str)
	// foo "bar" foo "bar" "foo \"bar\"" `foo "bar"`

	isLegume := map[string]bool{
		"a":true,
		"b":false,
	}
	fmt.Printf("%v %#v\n", isLegume, isLegume)
	//map[a:true b:false] map[string]bool{"a":true, "b":false}

	person := struct {
		Name string
		Age  int
	}{"kim", 22}
	fmt.Printf("%v %+v %#v\n", person, person, person)
	// {kim 22} {Name:kim Age:22} struct { Name string; Age int }{Name:"kim", Age:22}

	pointer := &person
	fmt.Printf("%v %p\n", pointer, pointer)
	//&{kim 22} 0xc00000c0a0

	greats := [5]string{"Kitano", "Kobayashi", "Kurosawa", "Miyazaki", "Ozu"}
	fmt.Printf("%v %q\n", greats, greats)
	//[Kitano Kobayashi Kurosawa Miyazaki Ozu] ["Kitano" "Kobayashi" "Kurosawa" "Miyazaki" "Ozu"]

	kGreats := greats[:3]
	fmt.Printf("%v %q %#v\n", kGreats, kGreats, kGreats)
	//[Kitano Kobayashi Kurosawa] ["Kitano" "Kobayashi" "Kurosawa"] []string{"Kitano", "Kobayashi", "Kurosawa"}

	cmd := []byte("a⌘")
	fmt.Printf("%v %d %s %q %x % x\n", cmd, cmd, cmd, cmd, cmd, cmd)
	// Result: [97 226 140 152] [97 226 140 152] a⌘ "a⌘" 61e28c98 61 e2 8c 98

	now := time.Unix(123456789, 0).UTC() // time.Time implements fmt.Stringer.
	fmt.Printf("%v %q\n", now, now)
	// Result: 1973-11-29 21:33:09 +0000 UTC "1973-11-29 21:33:09 +0000 UTC"
```



## strings



## bytes



## unicode

> 主要功能是实现了unicode 码点和 按照UTF-8编码的byte之间的转换。



## time

## http

1、http 的GET、POST、POSTFORM

```

```





















