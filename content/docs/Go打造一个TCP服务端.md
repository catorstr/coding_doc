# Go打造一个TCP服务端

>TCP(Transmission Control Protocol 传输控制协议)是一种
>
>面向连接(连接导向)的、可靠的、 基于IP的传输层协议。
>TCP在IP报文的协议号是6。TCP是一个超级麻烦的协议，而它又是互联网的基础，也是每个程序员必备的基本功。
>
>TCP是面向连接的，无论哪一方向另一方发送数据之前，都必须先在双方之间建立一条连接。在TCP/IP协议中，TCP 协议提供可靠的连接服务，连接是通过三次握手进行初始化的。
>
>​	TCP是互联网，尤其是物联网中非常重要的协议，很多协议也是基于TCP协议之上开发来的，在Go标准库net中，我们可以根据给定的端口轻易的创建一个TcpServer,在成功侦听某个端口后，我们用可以用某个连接进行操作了，进行双向的数据收发了。到此，大部分的应用已经足够用了。其它深入的功能在大多数的应用其实无需关心。

```go
package main 
import (
  "fmt"
  "log"
  "net"
)

func InitTcpServer() {

  log.Println("Starting  TCP Server :5558...")
  // 创建 listener
  listener, err := net.Listen("tcp4", ":5558")
  if err != nil {
    fmt.Println("Error listening", err.Error())
    return //终止程序
  }

  // 监听并接受来自客户端的连接
  for {
    conn, err := listener.Accept()
    if err != nil {
      fmt.Println("Client Disconnect ", err.Error())
      return // 终止程序
    }
    go doServerStuff(conn)
  }

}

//业务逻辑
func doServerStuff(conn net.Conn) {
  for {
    buf := make([]byte, 512)
    l, err := conn.Read(buf)
    remote_ip := conn.RemoteAddr().String()
    if err != nil {
      fmt.Println("Client Disconnect ", err.Error())
      return //终止程序
    }
    strin := string(buf[:l])
    fmt.Printf("接收到数据 %s %d data: %v\n", remote_ip, l, strin)
    strRet := "Hello" + strin + "\r\n"
    n, err := conn.Write([]byte(strRet))
    if err != nil {
      fmt.Println("conn.Write err=", err)
    }
    fmt.Printf("Send %d bytes\n", n)

  }
}
func main(){
    InitTcpServer()
}
```

