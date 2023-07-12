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

```go
/*
Copyright © 2023 NAME HERE <EMAIL ADDRESS>
*/
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

// emailCmd represents the email command
var emailCmd = &cobra.Command{
	Use:   "email",
	Short: "gsp notice consumer email server",
	PreRun: func(cmd *cobra.Command, args []string) {
		// filehive_client_uri2 := viper.GetString("filehive_client_uri")//硬读的话，当环境变量没有值时读取到的就是空值
		if filehive_client_uri == "" { //当命令行参数为空时才读取
			filehive_client_uri = viper.GetString("filehive_client_uri")
		}

	},
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("email called")
		fmt.Printf("s: %v\n", filehive_client_uri)
	},
}

var (
	filehive_client_uri string
	// filehive_client_token   string
	// rabbit_mq_host_and_port string
	// rabbit_mq_username      string
	// rabbit_mq_password      string
)

func init() {
	rootCmd.AddCommand(emailCmd) //将其挂载到根命令 app email -f "123"
	emailCmd.PersistentFlags().StringVarP(&filehive_client_uri, "filehive-client-uri", "f", "", "filehive client uri -- email attachment down uri")
	// emailCmd.PersistentFlags().StringVarP(&filehive_client_token, "filehive-client-token", "t", "", "filehive token")
	// emailCmd.PersistentFlags().StringVarP(&rabbit_mq_host_and_port, "rabbitmq-host-port", "r", "", "rabbitmq host and port")
	// emailCmd.PersistentFlags().StringVarP(&rabbit_mq_username, "rabbitmq-username", "u", "", "rabbitmq username")
	// emailCmd.PersistentFlags().StringVarP(&rabbit_mq_password, "rabbitmq-password", "p", "", "rabbitmq password")
	//使用viper设置环境变量的前缀
	viper.SetEnvPrefix("email")
	//读取环境变量
	viper.AutomaticEnv()
	//将命令行参数f的值 与 viper的key=filehive-client-uri绑定到一起，就可以使用viper.GetString("filehive_client_uri")读取命令行参数、环境变量、配置文件的值。
	viper.BindPFlag("filehive-client-uri", emailCmd.PersistentFlags().Lookup("f"))
	//viper.BindPFlag("filehive-client-token", emailCmd.Flags().Lookup("filehive-client-token"))
	// viper.BindPFlag("rabbitmq-host-port", emailCmd.Flags().Lookup("rabbitmq-host-port"))
	// viper.BindPFlag("rabbitmq-username", emailCmd.Flags().Lookup("rabbitmq-username"))
	// viper.BindPFlag("rabbitmq-password", emailCmd.Flags().Lookup("rabbitmq-password"))
}

Copyright © 2023 NAME HERE <EMAIL ADDRESS>
*/
package cmd

import (
	"fmt"

	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

// emailCmd represents the email command
var emailCmd = &cobra.Command{
	Use:   "email",
	Short: "gsp notice consumer email server",
	PreRun: func(cmd *cobra.Command, args []string) {
		// filehive_client_uri2 := viper.GetString("filehive_client_uri")//硬读的话，当环境变量没有值时读取到的就是空值
		if filehive_client_uri == "" { //当命令行参数为空时才读取
			filehive_client_uri = viper.GetString("filehive_client_uri")
		}

	},
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("email called")
		fmt.Printf("s: %v\n", filehive_client_uri)
	},
}

var (
	filehive_client_uri string
	// filehive_client_token   string
	// rabbit_mq_host_and_port string
	// rabbit_mq_username      string
	// rabbit_mq_password      string
)

func init() {
	rootCmd.AddCommand(emailCmd) //将其挂载到根命令 app email -f "123"
	emailCmd.PersistentFlags().StringVarP(&filehive_client_uri, "filehive-client-uri", "f", "", "filehive client uri -- email attachment down uri")
	// emailCmd.PersistentFlags().StringVarP(&filehive_client_token, "filehive-client-token", "t", "", "filehive token")
	// emailCmd.PersistentFlags().StringVarP(&rabbit_mq_host_and_port, "rabbitmq-host-port", "r", "", "rabbitmq host and port")
	// emailCmd.PersistentFlags().StringVarP(&rabbit_mq_username, "rabbitmq-username", "u", "", "rabbitmq username")
	// emailCmd.PersistentFlags().StringVarP(&rabbit_mq_password, "rabbitmq-password", "p", "", "rabbitmq password")
	//使用viper设置环境变量的前缀
	viper.SetEnvPrefix("email")
	//读取环境变量
	viper.AutomaticEnv()
	//将命令行参数f的值 与 viper的key=filehive-client-uri绑定到一起，就可以使用viper.GetString("filehive_client_uri")读取命令行参数、环境变量、配置文件的值。
	viper.BindPFlag("filehive-client-uri", emailCmd.PersistentFlags().Lookup("f"))
	//viper.BindPFlag("filehive-client-token", emailCmd.Flags().Lookup("filehive-client-token"))
	// viper.BindPFlag("rabbitmq-host-port", emailCmd.Flags().Lookup("rabbitmq-host-port"))
	// viper.BindPFlag("rabbitmq-username", emailCmd.Flags().Lookup("rabbitmq-username"))
	// viper.BindPFlag("rabbitmq-password", emailCmd.Flags().Lookup("rabbitmq-password"))
}

```
