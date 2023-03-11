# Go连接网络获取网页内容

>什么是 HTTP ？
>
>超文本传输协议（HTTP）的设计目的是保证客户端与服务器之间的通信。
>
>HTTP 的工作方式是客户端与服务器之间的请求-应答协议。
>
>web 浏览器可能是客户端，而计算机上的网络应用程序也可能作为服务器端。
>
>
>
>两种 HTTP 请求方法：GET 和 POST
>
>在客户机和服务器之间进行请求-响应时，两种最常被用到的方法是：GET 和 POST。
>
>GET - 从指定的资源请求数据。
>
>POST - 向指定的资源提交要被处理的数据。
>
>
>
>GET 请求可被缓存
>
>GET 请求保留在浏览器历史记录中
>
>GET 请求可被收藏为书签
>
>GET 请求不应在处理敏感数据时使用
>
>GET 请求有长度限制
>
>GET 请求只应当用于取回数据
>
>
>
>POST 请求不会被缓存
>
>POST 请求不会保留在浏览器历史记录中
>
>POST 不能被收藏为书签
>
>POST 请求对数据长度没有要求（有些服务器可能会有最大长度限制）

```go
package main

import (
"fmt"
"io/ioutil"
"net/http"
)

//获取网页内容，使用GET方式
func fetch(url string) string {
  fmt.Println("Fetch Url", url)
  client := &http.Client{}
  req, _ := http.NewRequest("GET", url, nil)
  req.Header.Set("User-Agent", "Mozilla/5.0")
  resp, err := client.Do(req)
if err != nil {
    fmt.Println("Http get err:", err)
return ""
  }
if resp.StatusCode != 200 {
    fmt.Println("Http status code:", resp.StatusCode)
return ""
  }
defer resp.Body.Close()
  body, err := ioutil.ReadAll(resp.Body)
if err != nil {
    fmt.Println("Read error", err)
return ""
  }
return string(body)
}

func main() {
  s := fetch("http://www.worldwarner.com")
  fmt.Println(s)

}

```

