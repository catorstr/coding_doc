# Go信号与管道Chan

> 在Go程序与其它软件与系统进行通信的过程中，免不了要进行数据的发送、等待与接收，所以我们会用到很多的线程去完成，这在任何语言都会遇到.但线程代码处理的不好，稍有不慎就会出很多问题，
>
>  所以Go有了这个信号与通道，极大的方便了这些工作
>
>  信道的英文是channel，在go当中的关键字是chan。用来在goroutine（协程）之间传输数据,关于协程，以后我们会详细讲，你可以简单的认为是线程.
>
>  在并发场景当中，多个线程彼此隔离的情况下，需要一个特殊的结构传输数据。Chan看起来比较怪，在其他语言当中基本没有出现过，但是它的原理和使用都非常简单。我们先来看它的使用，首先是定义一个chan，通过make关键字创建。golang当中的设计原则就是能省则省，能简单则简单。所以当我们要创建一个chan的时候，可以通过make实现。

```go
package main
import (
"fmt"
"time"
)
func main() {
  ch1 := make(chan int)
//启动一个协程
go func() {
    fmt.Println("等待5秒")
    time.Sleep(5 * time.Second)
    ch1 <- 48
    fmt.Println("等待结束")
}()
  fmt.Println("正在等待任务执行")
  //等待直到ch1可用了
  i1 := <-ch1
  fmt.Println(i1)
  fmt.Println("信号结束")
    //select的用法
cha := make(chan int)
chb := make(chan int, 10)
chc := make(chan int, 20)
go func(c1, c2, c3 chan<- int) {
  for {
    time.Sleep(1 * time.Second)
    c1 <- 1
    time.Sleep(1 * time.Second)
    c2 <- 2
    time.Sleep(1 * time.Second)
    c3 <- 3
    time.Sleep(5 * time.Second)
  }
}(cha, chb, chc)
for {
  select {
  case int1 := <-cha:
        fmt.Println("c1 value :", int1)
  case int2 := <-chb:
        fmt.Println("c2 value :", int2)
  case int3 := <-chc:
        fmt.Println("c3 vaule :", int3)
  case <-time.After(2 * time.Second):
        fmt.Println("超时了")
  return
  }
 }
}
```

