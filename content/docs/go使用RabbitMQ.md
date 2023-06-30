# Go使用RabbitMQ

> 官网地址：https://www.rabbitmq.com/tutorials/tutorial-one-go.html
>
> ### 安装
>
> 1. 通过docker image 安装使用
>
>    ```bash
>    #latest RabbitMQ 3.12.1(2023 06 30)
>    docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.12-management
>    ```
> 2. 在linux下安装使用
>
>    ```bash
>    #RabbitMQ的安装方式有很多种，这里我们直接安装它的二进制文件使用。
>    wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.12.1/rabbitmq-server-generic-unix-3.12.1.tar.xz
>    # 在github上下载的会比较慢。
>    ```
>
>    如果代理启动时没有足够的可用磁盘空间(默认情况下，它至少需要200 MB的可用空间)，他会拒绝接受消息。检查代理日志文件以确认并在必要时降低限制。配置文件文档将向您展示如何设置：**disk_free_limit**.  ``https://www.rabbitmq.com/configure.html#config-items``
>
> 先决条件
> 本教程假设RabbitMQ已在标准端口(5672)的本地主机上安装并运行。以防你用的是别的主机。端口或凭证。需要调整连接设置。

RabbitMQ是一个消息代理:它接受和转发消息。你可以把它想象成一个邮局:当你把你想要邮寄的邮件放在一个邮箱里时，你可以肯定的是，邮递员最终会把邮件送到你的收件人手中。在这个类比中，RabbitMQ是一个邮箱、一个邮局和一个邮递员。

RabbitMQ和邮局之间的主要区别在于它不处理纸张，而是接受、存储和转发二进制数据消息。

### 相关术语

> **生产者(P):** 发送消息的程序是生产者。
>
> **队列(queue):** 队列是RabbitMO中邮箱的名称。尽管消息通过RabbitMO和您的应用程序传递。它们只能存储在队列中。队列只受主机内存和磁盘限制的约束，它本质上是一个大的消息缓冲区。许多生产者可以发送到一个队列的消息，许多消费者可以尝试从一个队列接收数据。
>
> **消费者(C):** 消费者是一个主要等待接收消息的程序。
>
> ```
> 请注意，生产者、消费者和代理不必驻留在相同的主机上;事实上，在大多数应用程序中，它们不必驻留在相同的主机上。应用程序也可以是生产者和消费者。
> ```

### Hello 程序

> 在本教程的这一部分，我们将用GO编写两个小程序:一个生产者发送一条消息，一个消费者接收消息并打印出来。我们将在GORabbitMQAPI中失去一些细节，集中在这个非常简单的事情上只是为了开始。

#### 获取Go RabbitMQ客户端包

```bash
get github.com/rabbitmq/amqp091-go
```

```go
package handler_test

import (
	"context"
	"fmt"
	"log"
	"testing"
	"time"

	amqp "github.com/rabbitmq/amqp091-go"
)

func TestRabbitMQ(t *testing.T) {
	//连接RabbitMQ服务器
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "Failed to connect to RabbitMQ")
	defer conn.Close()
	//接下来，我们创建一个通道，它是完成任务的大部分apI所在的位置
	ch, err := conn.Channel()
	failOnError(err, "Failed to open a channel")
	defer ch.Close()

	//生产和消费消息之前需要一个消息队列。
	//声明一个队列
	queue, err := ch.QueueDeclare(
		"hello", //队列名,可以为空，在为空的情况下，服务器将生成一个唯一的名称，并将名称保存在返回的amqp.Queue结构体的name字段(即queue.Name)中。
		false,   //是否持久化,持久和非自动删除的队列将在服务器重新启动后继续存在，并在没有剩余的使用者或绑定时保留。服务器重新启动时，将在此队列中还原永久发布。这些队列只能绑定到持久的交换机。非持久队列和自动删除队列将不会在服务器重新启动时重新声明，并且将在最后一个使用者被取消或最后一个消费者的通道关闭后的短时间内由服务器删除。具有此生存期的队列也可以使用QueueDelete正常删除。这些持久队列只能绑定到非持久交换机。只要服务器正在运行，无论有多少使用者，非持久队列和非自动删除队列都将保持声明状态。此寿命对于消费者活动之间可能存在长延迟的临时拓扑非常有用。这些队列只能绑定到非持久交换机。
		false,   //是否在不使用时删除，
		false,   //是否独占队列，独占队列只能由声明它们的连接访问，并且将在连接关闭时删除。当试图声明、绑定、使用、清除或删除具有相同名称的队列时，其他连接上的通道将收到错误。
		false,   //是否阻塞，false不阻塞(不等待),当noWait为true时，队列将假定在服务器上声明。如果满足现有队列的条件或尝试从不同连接修改现有队列，则会出现通道异常。

		nil, //参数
	) //只有当队列不存在时才会创建当错误返回值不为nil时，可以假设无法使用这些参数声明队列，并且通道将关闭。
	failOnError(err, "Failed to queueDeclare")

	//x. 消费者接收消息
	//发布者可以随时发送消息，但是接收者这必要在发布者发送消息之前完成监听，才能接收到消息
	message, err := ch.Consume(
		queue.Name,
		"",
		true,
		false,
		false,
		false,
		nil,
	)
	failOnError(err, "Failed to register a consumer")

	go func() {
		for d := range message {
			fmt.Printf("[x] received a message: %s\n", d.Body)

		}
	}()
	//y. 发布者发送消息
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	//将消息发送到队列中
	bodyMsg := "hello world!"
	err = ch.PublishWithContext(ctx,
		"",         //交换机
		queue.Name, //当您希望将单个消息传递到单个队列时，可以使用队列名称的routingKey发布到默认交换机。
		false,      //由于发布是异步的，因此服务器将返回任何无法传递的消息。添加一个具有Channel.NotifyReturn的侦听器，以在调用带有强制参数或立即参数为true的发布时处理任何无法传递的消息。
		false,      //当强制标志为true并且没有绑定与路由关键字匹配的队列时，或者当立即标志为true且匹配队列上没有消费者准备好接受传递时，发布可能无法传递。
		amqp.Publishing{ //捕获发送到服务器的客户端消息。此结构中包含的Headers表之外的字段反映了内容框架中的基础字段。为了方便和高效，他们使用原生类型。
			ContentType: "text/plain",
			Body:        []byte(bodyMsg),
		}) //从客户端发送到服务器上的交换机。
	failOnError(err, "Failed to publish a message")
	log.Printf(" [y] Sent %s\n", bodyMsg)
	time.Sleep(time.Second * 5)

}

// 检查每个amqp调用的返回值
func failOnError(err error, msg string) {
	if err != nil {
		log.Panicf("%s: %s", msg, err)
	}
}

```

### 工作队列

>     工作队列背后的主要思想(又名:任务队列)是为了避免立即执行一个资源密集型任务，并不得不等待它完成。相反，我们将任务安排在稍后完成。我们将一个任务封装为一个消息，并将其发送到队列中。在后台运行的工作进程将弹出任务并最终执行作业。当您运行许多worker时，任务将在它们之间共享。作队列背后的假设是每个任务只被交付给一个工作者。
> 	这个概念在Web应用程序中特别有用，因为在Web应用程序中不可能在很短的HTTP请求窗口中处理复杂的任务。
>
> https://www.rabbitmq.com/tutorials/tutorial-two-go.html

### 发布/订阅

> 向多个消费者传递一个消息。这种模式称为“发布/订阅”。

### 交易所

> 在本教程的前几部分中，我们向队列发送和接收消息。现在是时候在Rabbit中介绍完整的消息传递模型了让我们快速浏览一下之前的教程中所介绍的内容:
>
> * 生产者是一个发送消息的用户应用程序
> * 队列是一个存储消息的缓冲区
> * 消费者是接收消息的用户应用程序
>
> RabbitMQ消息传递模型的核心思想是生产者从不直接向队列发送任何消息。通常情况下，生产者甚至根本不知道消息是否会被传递到任何队列中,相反，生产者只能向交换机发送消息。交换是一件很简单的事情。一边接收来自生产者的消息，另一边把他们推到队列里。交换机必须确切地知道如何处理它收到的消息。是否应该将其附加到特定队列中? 它应该被附加到许多队列中吗?还是应该被丢弃。
>
> 简单来说，RabbitMQ是这样工作的：一个生产者将消息传递给交换机，由交换机决定该消息是发送给队列A还是队列B，还是说将消息直接丢弃。
>
> https://www.rabbitmq.com/tutorials/tutorial-three-go.html

### Routing

>
>
> https://www.rabbitmq.com/tutorials/tutorial-four-go.html
