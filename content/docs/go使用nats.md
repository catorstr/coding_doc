# go使用nats

> 什么是NATS
> 软件应用和服务需要交换数据。nats是一个允许以消息形式分段的数据交换的基础设施。我们称之为“面向消息的中间件”。
> 通过NATS，应用程序开发人员可以:
> 1.毫不费力地构建分布式和可扩展的客户端一服务器应用程序
> 2.以通用的方式实时存储和分发数据。这可以在各种环境、语言、云提供商和内部部署系统中灵活实现。
>
> nats的使用很简单，但它很强大，关于nats的基础理论，请移步官网学习：https://nats.io，**理论基础是重点**。
>
> 安装一般需要安装nats-server和natscli两个工具，方便测试和使用：
>
>> nata-server安装：`` GO111MODULE=on go get github.com/nats-io/nats-server@latest``
>>
>> 详情：https://docs.nats.io/running-a-nats-service/introduction/installation
>>
>
>> natscli安装：https://github.com/nats-io/natscli/releases/
>>

### 使用

> 使用的话一般分两种，一种是直接将nats-server集成到我们的应用程序中，另一种是使用client句柄操作

```bash
# Go client latest or explicit version
go get github.com/nats-io/nats.go/@latest
go get github.com/nats-io/nats.go/@v1.26.0

# For latest NATS Server, add /v2 at the end
go get github.com/nats-io/nats-server/v2

# NATS Server v1 is installed otherwise
# go get github.com/nats-io/nats-server
```

#### 将nats集成到我们的应用程序中发布、订阅

```go
func TestNats(t *testing.T) {

	opts := &server.Options{
		Host: "0.0.0.0",
		Port: 2000,
	}

	ns, err := server.NewServer(opts)
	if err != nil {
		panic(err)
	}
	go ns.Start()

	if !ns.ReadyForConnections(4 * time.Second) {
		panic("not ready for connection")
	}

	nc, err := nats.Connect(ns.ClientURL())
	if err != nil {
		panic(err)
	}
	//订阅
	nc.Subscribe("hello", func(msg *nats.Msg) {
		data := string(msg.Data)
		fmt.Println(data)
		ns.Shutdown()
	})
	//发布
	nc.Publish("hello", []byte("test go nats sdk"))
	ns.WaitForShutdown()

}

```

#### 使用nats客户端来发布、订阅

> 需要自行先在本地使用nats-serer命令启应该服务。

```go
package data_test

import (
	"fmt"
	"testing"
	"time"

	"github.com/nats-io/nats.go"
)

func TestNats(t *testing.T) {
	nc, err := nats.Connect(nats.DefaultURL)
	if err != nil {
		panic(err)
	}
	// //异步订阅
	// nc.Subscribe("hello", func(msg *nats.Msg) {
	// 	data := string(msg.Data)
	// 	fmt.Println("订阅者1收到的内容:", data)
	// })
	// //发布主题
	// err = nc.Publish("hello", []byte("test nats connect"))
	// if err != nil {
	// 	panic(err)
	// }

	// //同步订阅
	// sub, err := nc.SubscribeSync("hello")
	// if err != nil {
	// 	panic(err)
	// }
	// m, err := sub.NextMsg(2 * time.Second)
	// if err != nil {
	// 	panic(err)
	// }
	// // err = nc.Publish("hello", []byte("test nats sync msg"))
	// // if err != nil {
	// // 	panic(err)
	// // }
	// fmt.Printf("m.Data: %v\n", string(m.Data))
	// time.Sleep(10 * time.Second)
	//使用channel订阅
	ch := make(chan *nats.Msg, 64) //nats不建议发送大量的消息，大量消息可能要使用高级功能吧，还在学
	sub, err := nc.ChanSubscribe("hello", ch)
	if err != nil {
		panic(err)
	}
	msg := <-ch
	fmt.Printf("msg.Data: %v\n", string(msg.Data))
	 time.Sleep(10 * time.Second)
	//取消订阅
	sub.Unsubscribe()
	//安全的断开连接，但还可以继续订阅（任务未执行完）。调用此函数不需要Close
	//Close立即断开连接
	sub.Drain()

	//待回复的订阅
	nc.Subscribe("request", func(msg *nats.Msg) {
		fmt.Printf("订阅msg.Data: %v\n", string(msg.Data))
		msg.Respond([]byte("收到"))
	})
	nc.Request("request", []byte("收到请回复"), 5*time.Second)

	nc.Subscribe("request", func(msg *nats.Msg) {
		nc.Publish("hello", []byte("002收到"))
	})

}

```
