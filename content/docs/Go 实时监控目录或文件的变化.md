#  Go 实时监控目录或文件的变化

> 我们在进行程序开发的过程中，
>
> 有时需要获取目录或文件的变化状态，
>
> 比如配置文件被修改后，
>
> 如果程序能够自动响应变化后的配置，
>
> 这样程序就会很智能，
>
> 那么目录或文件的变化，主要有以下几种：
>
> Write 写
>
> Remove 删除
>
> Create 新建 
>
> 那么我们就用Go来实现这个功能，这段代码同样是跨平台的，
>
> 在windows和linux上都能很好的工作，
>
> 不过在linux上的效率，会更高一些
>
> 以下是全部源代码：

```go
package main

import (
"log"

"github.com/fsnotify/fsnotify"
)

func main() {
  watcher, err := fsnotify.NewWatcher()
if err != nil {
    log.Fatal("启动监控失败: ", err)
  }

defer watcher.Close()

  done := make(chan bool)

go func() {
defer close(done)

for {
select {
case event, ok := <-watcher.Events:
if !ok {
return
        }
        log.Printf("%s %s\n", event.Name, event.Op)
case err, ok := <-watcher.Errors:
if !ok {
return
        }
        log.Println("error:", err)
      }
    }
  }()

  err = watcher.Add("./")
if err != nil {
    log.Fatal("开启监控失败:", err)
  }
  <-done
}

```

> 先调用NewWatcher创建一个监听器；
>
> 然后调用监听器的Add增加监听的文件或目录；
>
> 如果目录或文件有事件产生，
>
> 监听器中的通道Events可以取出事件。
>
> 如果出现错误，
>
> 监听器中的通道Errors可以取出错误信息。
>
> 注意，
>
> fsnotify使用了操作系统接口，
>
> 监听器中保存了系统资源的句柄，
>
> 使用后需要关闭。