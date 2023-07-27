# Cobra 使用详解

> https://github.com/spf13/cobra-cli/blob/main/README.md
>
> https://github.com/spf13/cobra
>
> https://github.com/spf13/cobra/blob/main/user_guide.md
>
> https://www.topgoer.cn/docs/goday/goday-1crg2gl84t0vp

### 安装

> go install github.com/spf13/cobra-cli@latest

查看帮助：cobra-cli --help

### 创建cli项目

> cobra-cli init [appName]

```bash
cd $HOME/code 
mkdir myapp
cd myapp
go mod init myapp
cobra-cli init
#cobra-cli init --author "作者"  #给程序添加作者
#cobra-cli init --viper "作者"  #与viper搭配，viper后续在学：https://github.com/spf13/viper；https://www.topgoer.cn/docs/goday/goday-1crg2dneqeek8
```

### 在一个命令行项目下添加另一个命令行项目

> 在原项目main.go的同级目录下：
>
> ```bash
> cobra-cli add serve
> cobra-cli add config
> cobra-cli add create -p 'configCmd' #cobra 使用的是小驼峰命名法
> #For example, cobra-cli add add-user is incorrect, but cobra-cli add addUser is valid.
> ```
>
> 个最后的命令有一个-p标志，它用于将一个父命令分配给新添加的命令，在本例中，我们希望将“create”命令分配给“onfig”命令。如果末指定，则所有命令都具有rootCmd的默认父级
> 默认情况下，cobra-cli将在提供的名称后面追加Cmd，并将此名称用作内部变量名。指定父项时，请务必与代码中使用的变量名相匹配。
>
> 项目结构会变成这样子：
>
> ```
>   ▾ app/
>     ▾ cmd/
>         config.go
>         create.go
>         serve.go
>         root.go
>       main.go
> ```
>
> 组织子命令
> 命令可以具有子命令，子命令又可以具有其它子命令。这是通过使用AddCommand实现的。在某些情况下，特别是在较大的应用程序中每个子命令可以在自己的go包中定义
> 建议的方法是让父命令使用AddCommand添加其最直接的子命令。例如，考虑以下目录结构
>
> ```
> ├── cmd
> │   ├── root.go
> │   └── sub1
> │       ├── sub1.go
> │       └── sub2
> │           ├── leafA.go
> │           ├── leafB.go
> │           └── sub2.go
> └── main.go
> ```
>
> 在这种情况下:
> ·root.go的init函数将sub1.go中定义的命令添加到根命令中
> sub1.go的init函数将sub2.go中定义的命令添加到sub1命令中
> sub2.go的init函数将leafA.go和leafB.go中定义的命令添加到sub2命令中
> 这种方法确保子命令总是在编译时被包含，同时避免了循环引用.
>
> 在编写/使用命令行程序时要注意一条规则：命令行参数值>环境变量参数值>大于配置文件参数值>大于默认值。
>
> ### 更多移步官网学习吧

## 例子

以下是一个示例代码，展示了如何使用Cobra和Viper来实现这种双向绑定。

```go
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

var rootCmd = &cobra.Command{
	Use:   "consumer",
	Short: "gsp notice consumer server",
	PersistentPreRunE: func(cmd *cobra.Command, args []string) error {
		return nil
	},
	RunE: func(cmd *cobra.Command, args []string) error {
		fmt.Printf("name: %v\n", name)
		//注意这样写的话，name的默认值会被环境变量的值或是配置文件中的时覆盖掉,这看起来是没有什么问题：符合命令行参数>环境变量的参数>配置文件的参数>大于默认值。但是当符合命令行参数、环境变量的参数、配置文件的参数都为空时，你希望默认值有效，但是此时默认值已经被viper.GetString()函数取到的空值覆盖掉了，开发中一不注意就是一个坑。
		name = viper.GetString("name") //a
		fmt.Println("Hello", name)
		//那么对有默认值的命令行参数该如何在令行参数、环境变量参数、配置文件参数、默认值之间合理的取值呢
		// fmt.Printf("http_addr: %v\n", http_addr)
		// if http_addr == "" {
		// 	http_addr = viper.GetString("http-addr")
		// }
		// fmt.Printf("http_addr: %v\n", http_addr) //很明显这样的话只有令行参数和默认值可用，而必读取不到环境变量参数、配置文件参数
		//emm... 我也想不出好方法，所以建议：有默认值值的参数，如果要改的话，直接命令行参数来改。
		//没有默认值的,用以上a方式来读取环境变量和命令行参数。a方式提供了一种cobra与viper结合读取环境变量的的方式。可能还有其他的方式，但是官方没有看到详细的介绍，以上是通过测试实践的得出的建议。
		return nil
	},
}
var (
	debug     bool
	grpc_addr string
	http_addr string
	name      string
)

func init() {
	rootCmd.PersistentFlags().BoolVar(&debug, "debug", false, "enable debug")
	rootCmd.PersistentFlags().StringVar(&http_addr, "http-addr", ":8090", "sever http listen addr")
	rootCmd.PersistentFlags().StringVar(&grpc_addr, "grpc-addr", ":8090", "sever grpc listen addr")

	// 添加命令行参数
	rootCmd.PersistentFlags().StringVarP(&name, "name", "n", "bob", "Your name")
	// 与viper参数参数绑定 当用户提供 --name 标志时，变量 name 将不会设置为配置中的值。
	viper.BindPFlag("name", rootCmd.PersistentFlags().Lookup("name"))
	//与环境变量绑定
	viper.SetEnvPrefix("App") //设置环境变量的前缀不管key是App，还是app、环境变量的前端都会被命名为 APP
	viper.AutomaticEnv()      //它将检查名称与大写键匹配的环境变量，并以 SetEnvPrefix 为前缀（如果已设置）。
	/*
		BindEnv 接受一个或多个参数。第一个参数是键名，其余参数是绑定到该键的环境变量的名称。如果提供了多个，则它们将按指定的顺序优先。环境变量的名称区分大小写。
	*/
	// 如果未提供 ENV 变量名称，则 Viper 将自动假定 ENV 变量符合以下格式：前缀 +“_”+全部大写的键名。
	//viper.BindEnv("name") // APP_NAME="Bob" ./consumer  -->Hello Bob  | APP_NAME="Bob" ./consumer --name "alce" -->Hello alce  | ./consumer --name "alce"  -->Hello alce
	// 当您显式提供 ENV 变量名称（第二个参数）时，它不会自动添加前缀。例如，如果第二个参数是“ID”，Viper 将查找 ENV 变量“ID”。
	//viper.BindEnv("name", "ID") //将环境变量的key=MYAPP_NAME 绑定到name上 =>APP_ID="Bob" ./consumer 加前缀读取不到值
	//ID="Bob" ./consumer 这样才可以读取值
	//但是，这样写的话,加不加前缀都可以读取到，有点离谱
	viper.BindEnv("name", "NAME") //NAME="Bob" ./consumer ->Hello Bob  | APP_NAME="Bob" ./consumer -->Hello Bob
	viper.BindPFlag("http-addr", rootCmd.PersistentFlags().Lookup("http-addr"))
	//viper.BindEnv("http-addr", "HTTP_ADDR")
}

func Execute() error {
	return rootCmd.Execute()
}

```

> 上述代码中，我们首先创建了一个名为 `consumer`的Cobra根命令。然后，我们添加了一个名为 `name`的命令行参数。接下来，我们使用 `viper`的 `BindPFlag`方法将该命令行参数与Viper实例中的键 `name`绑定。
>
> 为了绑定环境变量，我们使用 `viper`的 `BindEnv`方法来绑定名为 `MYAPP_NAME`的环境变量到 `name`键。
>
> 这样，当我们在命令行中提供 `--name`参数时，Viper将自动将该值绑定到 `name`键。如果未提供命令行参数，则Viper将在环境变量 中查找值。
