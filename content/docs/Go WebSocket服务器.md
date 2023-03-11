#  Go WebSocket服务器

> WebSocket是可以持续双向传输数据的一种协议，
>
> 在微信小程序及APP中使用频率很高，他的特点是实时性强、保密性也高
>
> 本节干货，我们就用Go来实现一个简易的WebSocket服务器，同时提供了一个Web访问的实现方式 

```go

package main

import (
"fmt"
"html/template"
"log"
"net/http"
"os"
"strings"

"golang.org/x/net/websocket"
)

func upper(ws *websocket.Conn) {
var err error
for {
var reply string

if err = websocket.Message.Receive(ws, &reply); err != nil {
      fmt.Println(err)
break

    }
    fmt.Println("reveived from client: " + reply)
if err = websocket.Message.Send(ws, strings.ToUpper(reply)); err != nil {
      fmt.Println(err)
continue
    }
  }
}

func index(w http.ResponseWriter, r *http.Request) {
  t, _ := template.ParseFiles("index.html")
  t.Execute(w, nil)
}

func main() {
  http.Handle("/test1", websocket.Handler(upper))
  http.HandleFunc("/", index)

  log.Println("Websocket Server start 127.0.0.1:6432")
if err := http.ListenAndServe(":6432", nil); err != nil {
    fmt.Println(err)
    os.Exit(1)
  }
}
```

```go

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"/>
<title>Websocket</title>
</head>
<body>
<h1>WebSocket测试</h1>
<form>
<p>
            字符串: <input id="content" type="text" placeholder="字符串" value="123456">
</p>
</form>
<label id="result">结果为：</label><br><br>
<button onclick="send()">发送</button>

<script type="text/javascript">
var sock = null;
var wsuri = "ws://127.0.0.1:6432/test1";
        sock = new WebSocket(wsuri);
        sock.onopen=function(e){
            alert("连接成功");
        }
        sock.onclose=function(e){
            alert("关闭了");
        }
        sock.onerror=function(e){
            alert("error "+e);
        }
        sock.onmessage = function(e) {
var result = document.getElementById('result');
            result.innerHTML = "结果为：" + e.data;
        }

function send() {
var msg = document.getElementById('content').value;
            sock.send(msg);
        }
</script>
</body>
</html>
```

> WebSocket是一种在单个TCP连接上进行全双工通信的协议。
>
> WebSocket通信协议于2011年被IETF定为标准RFC 6455，
>
> 并由RFC7936补充规范。
>
> WebSocket API也被W3C定为标准。
>
> WebSocket使得客户端和服务器之间的数据交换变得更加简单，
>
> 允许服务端主动向客户端推送数据。
>
> 在WebSocket API中，
>
> 浏览器和服务器只需要完成一次握手，
>
> 两者之间就直接可以创建持久性的连接，
>
> 并进行双向数据传输。
>
> 在连接创建后，
>
> 服务器和客户端之间交换数据时，
>
> 用于协议控制的数据包头部相对较小。
>
> 在不包含扩展的情况下，
>
> 对于服务器到客户端的内容，
>
> 此头部大小只有2至10字节（和数据包长度有关）；
>
> 对于客户端到服务器的内容，
>
> 此头部还需要加上额外的4字节的掩码。
>
> 相对于HTTP请求每次都要携带完整的头部，
>
> 此项开销显著减少了。
>
> 
>
> 由于协议是全双工的，
>
> 所以服务器可以随时主动给客户端下发数据。
>
> 相对于HTTP请求需要等待客户端发起请求服务端才能响应，
>
> 延迟明显更少；
>
> 
>
> 与HTTP不同的是，
>
> Websocket需要先创建连接，
>
> 这就使得其成为一种有状态的协议，
>
> 之后通信时可以省略部分状态信息。
>
> 而HTTP请求可能需要在每个请求都携带状态信息（如身份认证等）。
>
> 
>
> Websocket定义了二进制帧，
>
> 相对HTTP，
>
> 可以更轻松地处理二进制内容。
>
> 
>
> 相对于HTTP压缩，
>
> Websocket在适当的扩展支持下，
>
> 可以沿用之前内容的上下文，
>
> 在传递类似的数据时，
>
> 可以显著地提高压缩率。