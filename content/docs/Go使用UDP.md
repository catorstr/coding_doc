# Go使用UDP

> UDP 是User Datagram Protocol的简称， 中文名是用户数据报协议，是OSI（Open System Interconnection，开放式系统互联） 参考模型中一种无连接的传输层协议，提供面向事务的简单不可靠信息传送服务，IETF RFC 768 [1] 是UDP的正式规范。
>
> UDP协议与TCP协议一样用于处理数据包，在OSI模型中，两者都位于传输层，处于IP协议的上一层。UDP有不提供数据包分组、组装和不能对数据包进行排序的缺点，也就是说，当报文发送之后，是无法得知其是否安全完整到达的。UDP用来支持那些需要在计算机之间传输数据的网络应用。包括网络视频会议系统在内的众多的客户/服务器模式的网络应用都需要使用UDP协议。UDP协议从问世至今已经被使用了很多年，虽然其最初的光彩已经被一些类似协议所掩盖，但即使在今天UDP仍然不失为一项非常实用和可行的网络传输层协议。
>
> 许多应用只支持UDP，如：多媒体数据流，不产生任何额外的数据，即使知道有破坏的包也不进行重发。当强调传输性能而不是传输的完整性时，如：音频和多媒体应用，UDP是最好的选择。在数据传输时间很短，以至于此前的连接过程成为整个流量主体的情况下，UDP也是一个好的选择。 

以下就是实现的基本代码

**服务端**

```go
package main

import (
"fmt"
"net"
"os"
"strings"
)

func main() {
  fmt.Println("启动UDP Server")
  addr, err := net.ResolveUDPAddr("udp", "0.0.0.0:9009")
if err != nil {
    fmt.Println(err)
    os.Exit(1)
  }

  fmt.Println(addr)
  conn, err := net.ListenUDP("udp", addr)
if err != nil {
    fmt.Println(err)
    os.Exit(1)
  }

defer conn.Close()

for {
    data := make([]byte, 32)
    _, rAddr, err := conn.ReadFromUDP(data)
if err != nil {
      fmt.Println(err)
continue
    }

    strData := string(data)
    fmt.Println("Received:", strData)

    upper := strings.ToUpper(strData)
    _, err = conn.WriteToUDP([]byte(upper), rAddr)
if err != nil {
      fmt.Println(err)
continue
    }

    fmt.Println("Send:", upper)
  }

}
```

**客户端**

```go
package main
import ("bufio""fmt""net""os")
func main() {
//发起一个UDP请求//这里使用的是本机127.0.0.1//如果需要以广播的形式，可以写255.255.255.255  
    conn, _ := net.Dial("udp", "127.0.0.1:9009")
	defer conn.Close()
  	input := bufio.NewScanner(os.Stdin)
	//接收键盘输入for input.Scan() {
	//读取一行文字，用于发送    
    line := input.Text()
	//发送UDP    
    conn.Write([]byte(line))
    fmt.Println("Write:", line)
    msg := make([]byte, 32)
	//接收    
    conn.Read(msg)
    fmt.Println("Response:", string(msg))  }
}
```