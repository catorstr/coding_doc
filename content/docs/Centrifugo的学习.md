# Centrifugo(离心机)的学习

> 离心机(Centrifugo)是语言无关的，可用于构建聊天应用程序，实时评论，多人游戏，实时数据可视化，协作工具等，与任何后端相结合，它非常适合现代架构，并允许从实时传输层解耦业务逻辑。
>
> Centrifugo 一个用Go语言构建实时消息应用程序的框架。如果您听说过 socket.io，那么您可以将 Centrifuge 视为类似物。我认为现在最流行的应用程序是不同形式的聊天，但我想强调 Centrifuge 不是一个构建聊天的框架 – 它是一个通用工具，可用于创建不同类型的实时应用程序 – 实时图表，多人游戏。
>
> 官方地址: https://centrifugal.dev/docs/getting-started/introduction
>
> #实战教程：
>
> 1. https://medium.com/@fzambia/four-years-in-centrifuge-ce7a94e8b1a8
> 2. https://centrifugal.dev/blog/2021/01/15/centrifuge-intro

## 例子

> https://github.com/centrifugal/centrifuge-go/blob/master/examples/chat/main.go

### chat

```go
package main

import (
	"bufio"
	"context"
	"encoding/json"
	"log"
	"net/http"
	"os"
	"strings"

	_ "net/http/pprof"

	"github.com/centrifugal/centrifuge-go"
	"github.com/golang-jwt/jwt"
)

// In real life clients should never know secret key. This is only for example
// purposes to quickly generate JWT for connection.
const exampleTokenHmacSecret = "secret"

func connToken(user string, exp int64) string {
	// NOTE that JWT must be generated on backend side of your application!
	// Here we are generating it on client side only for example simplicity.
	claims := jwt.MapClaims{"sub": user}
	if exp > 0 {
		claims["exp"] = exp
	}
	t, err := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString([]byte(exampleTokenHmacSecret))
	if err != nil {
		panic(err)
	}
	return t
}

// ChatMessage is chat example specific message struct.
type ChatMessage struct {
	Input string `json:"input"`
}

func main() {
	go func() {
		log.Println(http.ListenAndServe(":6060", nil))
	}()

	client := centrifuge.NewJsonClient(
		"ws://localhost:8000/connection/websocket",
		centrifuge.Config{
			// Sending token makes it work with Centrifugo JWT auth (with `secret` HMAC key).
			Token: connToken("49", 0),
		},
	)
	defer client.Close()

	client.OnConnecting(func(e centrifuge.ConnectingEvent) {
		log.Printf("Connecting - %d (%s)", e.Code, e.Reason)
	})
	client.OnConnected(func(e centrifuge.ConnectedEvent) {
		log.Printf("Connected with ID %s", e.ClientID)
	})
	client.OnDisconnected(func(e centrifuge.DisconnectedEvent) {
		log.Printf("Disconnected: %d (%s)", e.Code, e.Reason)
	})

	client.OnError(func(e centrifuge.ErrorEvent) {
		log.Printf("Error: %s", e.Error.Error())
	})

	client.OnMessage(func(e centrifuge.MessageEvent) {
		log.Printf("Message from server: %s", string(e.Data))
	})

	client.OnSubscribed(func(e centrifuge.ServerSubscribedEvent) {
		log.Printf("Subscribed to server-side channel %s: (was recovering: %v, recovered: %v)", e.Channel, e.WasRecovering, e.Recovered)
	})
	client.OnSubscribing(func(e centrifuge.ServerSubscribingEvent) {
		log.Printf("Subscribing to server-side channel %s", e.Channel)
	})
	client.OnUnsubscribed(func(e centrifuge.ServerUnsubscribedEvent) {
		log.Printf("Unsubscribed from server-side channel %s", e.Channel)
	})

	client.OnPublication(func(e centrifuge.ServerPublicationEvent) {
		log.Printf("Publication from server-side channel %s: %s (offset %d)", e.Channel, e.Data, e.Offset)
	})
	client.OnJoin(func(e centrifuge.ServerJoinEvent) {
		log.Printf("Join to server-side channel %s: %s (%s)", e.Channel, e.User, e.Client)
	})
	client.OnLeave(func(e centrifuge.ServerLeaveEvent) {
		log.Printf("Leave from server-side channel %s: %s (%s)", e.Channel, e.User, e.Client)
	})

	err := client.Connect()
	if err != nil {
		log.Fatalln(err)
	}

	sub, err := client.NewSubscription("chat:index", centrifuge.SubscriptionConfig{
		Recoverable: true,
		JoinLeave:   true,
	})
	if err != nil {
		log.Fatalln(err)
	}

	sub.OnSubscribing(func(e centrifuge.SubscribingEvent) {
		log.Printf("Subscribing on channel %s - %d (%s)", sub.Channel, e.Code, e.Reason)
	})
	sub.OnSubscribed(func(e centrifuge.SubscribedEvent) {
		log.Printf("Subscribed on channel %s, (was recovering: %v, recovered: %v)", sub.Channel, e.WasRecovering, e.Recovered)
	})
	sub.OnUnsubscribed(func(e centrifuge.UnsubscribedEvent) {
		log.Printf("Unsubscribed from channel %s - %d (%s)", sub.Channel, e.Code, e.Reason)
	})

	sub.OnError(func(e centrifuge.SubscriptionErrorEvent) {
		log.Printf("Subscription error %s: %s", sub.Channel, e.Error)
	})

	sub.OnPublication(func(e centrifuge.PublicationEvent) {
		var chatMessage *ChatMessage
		err := json.Unmarshal(e.Data, &chatMessage)
		if err != nil {
			return
		}
		log.Printf("Someone says via channel %s: %s (offset %d)", sub.Channel, chatMessage.Input, e.Offset)
	})
	sub.OnJoin(func(e centrifuge.JoinEvent) {
		log.Printf("Someone joined %s: user id %s, client id %s", sub.Channel, e.User, e.Client)
	})
	sub.OnLeave(func(e centrifuge.LeaveEvent) {
		log.Printf("Someone left %s: user id %s, client id %s", sub.Channel, e.User, e.Client)
	})

	err = sub.Subscribe()
	if err != nil {
		log.Fatalln(err)
	}

	pubText := func(text string) error {
		msg := &ChatMessage{
			Input: text,
		}
		data, _ := json.Marshal(msg)
		_, err := sub.Publish(context.Background(), data)
		return err
	}

	err = pubText("hello")
	if err != nil {
		log.Printf("Error publish: %s", err)
	}

	log.Printf("Print something and press ENTER to send\n")

	// Read input from stdin.
	go func(sub *centrifuge.Subscription) {
		reader := bufio.NewReader(os.Stdin)
		for {
			text, _ := reader.ReadString('\n')
			text = strings.TrimSpace(text)
			switch text {
			case "#subscribe":
				err := sub.Subscribe()
				if err != nil {
					log.Println(err)
				}
			case "#unsubscribe":
				err := sub.Unsubscribe()
				if err != nil {
					log.Println(err)
				}
			case "#disconnect":
				err := client.Disconnect()
				if err != nil {
					log.Println(err)
				}
			case "#connect":
				err := client.Connect()
				if err != nil {
					log.Println(err)
				}
			case "#close":
				client.Close()
			default:
				err = pubText(text)
				if err != nil {
					log.Printf("Error publish: %s", err)
				}
			}
		}
	}(sub)

	// Run until CTRL+C.
	select {}
}
```

### token_subscription

```go
// Private channel subscription example.
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/centrifugal/centrifuge-go"
	"github.com/golang-jwt/jwt"
)

func connToken(user string, exp int64) string {
	// NOTE that JWT must be generated on backend side of your application!
	// Here we are generating it on client side only for example simplicity.
	claims := jwt.MapClaims{"sub": user}
	if exp > 0 {
		claims["exp"] = exp
	}
	t, err := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString([]byte("secret"))
	if err != nil {
		panic(err)
	}
	return t
}

func subscriptionToken(channel string, user string, exp int64) string {
	// NOTE that JWT must be generated on backend side of your application!
	// Here we are generating it on client side only for example simplicity.
	claims := jwt.MapClaims{"channel": channel, "sub": user}
	if exp > 0 {
		claims["exp"] = exp
	}
	t, err := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString([]byte("secret"))
	if err != nil {
		panic(err)
	}
	return t
}

func main() {
	wsURL := "ws://localhost:8000/connection/websocket"
	c := centrifuge.NewJsonClient(wsURL, centrifuge.Config{
		Token: connToken("112", 0),
	})
	defer c.Close()

	c.OnConnected(func(_ centrifuge.ConnectedEvent) {
		log.Println("Connected")
	})
	c.OnConnecting(func(_ centrifuge.ConnectingEvent) {
		log.Println("Connecting")
	})
	c.OnDisconnected(func(e centrifuge.DisconnectedEvent) {
		log.Println("Disconnected", e.Reason)
	})
	c.OnError(func(e centrifuge.ErrorEvent) {
		log.Println("Error", e.Error.Error())
	})

	err := c.Connect()
	if err != nil {
		log.Fatalln(err)
	}

	sub, err := c.NewSubscription("$chat:index", centrifuge.SubscriptionConfig{
		GetToken: func(e centrifuge.SubscriptionTokenEvent) (string, error) {
			log.Println("Getting subscription token")
			token := subscriptionToken(e.Channel, "112", time.Now().Unix()+10)
			return token, nil
		},
	})
	if err != nil {
		log.Fatalln(err)
	}

	sub.OnSubscribed(func(e centrifuge.SubscribedEvent) {
		log.Println(fmt.Sprintf("Successfully subscribed to private channel %s", sub.Channel))
	})
	sub.OnError(func(e centrifuge.SubscriptionErrorEvent) {
		log.Println(fmt.Sprintf("Error subscribing to private channel %s: %v", sub.Channel, e.Error))
	})
	sub.OnUnsubscribed(func(e centrifuge.UnsubscribedEvent) {
		log.Println(fmt.Sprintf("Unsubscribed from private channel %s: %s", sub.Channel, e.Reason))
	})
	sub.OnPublication(func(e centrifuge.PublicationEvent) {
		log.Println(fmt.Sprintf("New message received from channel %s: %s", sub.Channel, string(e.Data)))
	})

	// Subscribe on private channel.
	_ = sub.Subscribe()

	select {}
}
```

### RPC

> https://centrifugal.dev/docs/server/server_api#grpc-example-for-go

```go
package main

import (
	"context"
	"log"
	"net/http"
	_ "net/http/pprof"

	"github.com/centrifugal/centrifuge-go"
)

func newClient() *centrifuge.Client {
	wsURL := "ws://localhost:8000/connection/websocket"
	c := centrifuge.NewJsonClient(wsURL, centrifuge.Config{})

	c.OnConnecting(func(_ centrifuge.ConnectingEvent) {
		log.Println("Connecting")
	})
	c.OnConnected(func(_ centrifuge.ConnectedEvent) {
		log.Println("Connected")
	})
	c.OnDisconnected(func(_ centrifuge.DisconnectedEvent) {
		log.Println("Disconnected")
	})
	c.OnError(func(e centrifuge.ErrorEvent) {
		log.Println("Error", e.Error.Error())
	})
	c.OnMessage(func(e centrifuge.MessageEvent) {
		log.Println("Message received", string(e.Data))

		// When issue blocking requests from inside event handler we must use
		// a goroutine. Otherwise, connection read loop will be blocked.
		go func() {
			result, err := c.RPC(context.Background(), "method", []byte("{}"))
			if err != nil {
				log.Println(err)
				return
			}
			log.Printf("RPC result 2: %s", string(result.Data))
		}()
	})
	return c
}

func main() {
	log.Println("Start program")

	go func() {
		log.Println(http.ListenAndServe(":6060", nil))
	}()

	c := newClient()
	defer c.Close()

	err := c.Connect()
	if err != nil {
		log.Fatalln(err)
	}

	result, err := c.RPC(context.Background(), "method", []byte("{}"))
	if err != nil {
		log.Fatalln(err)
		return
	}
	log.Printf("RPC result: %s", string(result.Data))

	select {}
}
```

### refresh

```go
// Demonstrate how to resque from credentials expiration
// (when connection_lifetime set in Centrifugo).
package main

import (
	"log"
	"time"

	"github.com/centrifugal/centrifuge-go"
	"github.com/golang-jwt/jwt"
)

func connToken(user string, exp int64) string {
	// NOTE that JWT must be generated on backend side of your application!
	// Here we are generating it on client side only for example simplicity.
	claims := jwt.MapClaims{"sub": user}
	if exp > 0 {
		claims["exp"] = exp
	}
	t, err := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString([]byte("secret"))
	if err != nil {
		panic(err)
	}
	return t
}

func main() {
	wsURL := "ws://localhost:8000/connection/websocket"
	c := centrifuge.NewJsonClient(wsURL, centrifuge.Config{
		Token: connToken("113", time.Now().Unix()+10),
		// GetToken will be called to get new connection token when
		// original token set above expires. You can also skip setting initial
		// token – in this case client will call GetToken before connecting
		// to a server.
		GetToken: func(_ centrifuge.ConnectionTokenEvent) (string, error) {
			log.Println("Refresh connection token")
			token := connToken("113", time.Now().Unix()+10)
			return token, nil
		},
	})
	defer c.Close()

	c.OnConnected(func(_ centrifuge.ConnectedEvent) {
		log.Println("Connected")
	})
	c.OnConnecting(func(_ centrifuge.ConnectingEvent) {
		log.Println("Connecting")
	})
	c.OnDisconnected(func(_ centrifuge.DisconnectedEvent) {
		log.Println("Disconnected")
	})
	c.OnError(func(e centrifuge.ErrorEvent) {
		log.Println("Error", e.Error.Error())
	})

	err := c.Connect()
	if err != nil {
		log.Fatalln(err)
	}

	select {}
}

```
