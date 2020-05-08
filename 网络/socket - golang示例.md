# socket - golang 使用socket编程示例

Golang 网络编程：

1、创建socket实例，bind 到对应的地址、端口，listen, accept, receive/send, close

```go
func main()  {
	server, err := net.Listen("tcp", "127.0.0.1:8888")
	if err != nil {
		fmt.Printf("fail to start server, %s\n", err)
	}
	fmt.Printf("server start\n")

	for {
		conn, err := server.Accept()
		if err != nil {
			fmt.Printf("fail to create connet, %s\n", err)
		}

		go connHandler(conn)
	}

}

func connHandler(c net.Conn)  {
	if c == nil {
		return
	}
	buf := make([]byte, 4096)
	for {
		cot, err := c.Read(buf)
		if err != nil || cot == 0 {
			fmt.Printf("can not read input, %s\n", err)
			c.Close()
			break
		}
		inStr := strings.TrimSpace(string(buf[0:cot]))
		inputStr := strings.Split(inStr, " ")
		switch inputStr[0] {
			case "ping":
				c.Write([]byte("pong\n"))
			case "echo":
				echoStr := strings.Join(inputStr[1:], " ") + "\n"
				c.Write([]byte(echoStr))
		case "quit":
				c.Close()
				break
		default:
			fmt.Printf("unsupport command, %s\n", inputStr[0:])
		}

	}
	fmt.Printf("connent from %s closed\n", c.RemoteAddr())

}

```

