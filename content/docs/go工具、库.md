# Go开发中实用的工具、库

### 开发自己的命令行行工具

> urfave/cli 是一个简单、快速、有趣的Go命令行应用程序包。的目标是使开发人员能够以一种富有表现力的方式编写快速且可分布的命令行应用程序。官网：https://cli.urfave.org
>
> * 安装
>   go get github.com/urfave/cli/v2
> * 官方例子
>
> ```go
> //首先创建一个名为greet的目录，并在其中添加一个文件greet.go。即初始话好我们的greet项目。然后添加下面的代码:
> package main
>
> import (
>     "fmt"
>     "log"
>     "os"
>
>     "github.com/urfave/cli/v2"
> )
>
> func main() {
>     app := &cli.App{
>         Name:  "greet",
>         Usage: "fight the loneliness!",
>         Action: func(*cli.Context) error {
>             fmt.Println("Hello friend!")
>             return nil
>         },
>     }
>
>     if err := app.Run(os.Args); err != nil {
>         log.Fatal(err)
>     }
> }
> ```
>
> * 将我们写好的的程序安装到$GOPATH/bin目录中
>   ```bash
>   go install
>   ```
> * 使用我们的命令行工具
>
> ```bash
> $ greet
> Hello friend!
> ```
>
> * 像使用别人的命令行工具一样，查看我们自己编写的命令行工具的帮助信息
>
> ```bash
> $ greet help
> NAME:
>    greet - fight the loneliness!
>
> USAGE:
>    greet [global options] command [command options] [arguments...]
>
> COMMANDS:
>    help, h  Shows a list of commands or help for one command
>
> GLOBAL OPTIONS:
>    --help, -h  show help
> ```
>
> 恭喜，你已经完成了你的第一个命令行工具的编写。
>
> * 实战
>
>> 刚好近期想搭建一个基础开发框架，方便日常生活的开发，于是就用这个包，编写一个命令行工具吧！功能类似于kratos或goctl那样的工具，当然本人水平有限写的不像goctl和kratos那么好啊，纯属为了方便而编写该工具来拼接日常工作中一些重复的工作。xadtctl.
>>
>
> //todo 需要去看一下如何编写项目发布到github上，使别人能用go install 安装这个命令到本地。

### 第三方好用的工具和库

* 富文本解析：https://github.com/russross/blackfriday
*
