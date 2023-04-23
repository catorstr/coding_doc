# protoc 简介

> 在linux环境下输入protoc --version 查看protoc版本，如果没有一般会提示你安装。

### proto文件

```properties
//greeter.proto 文件
syntax = "proto3" //指定proto版本

//包命名，确保包名不冲突，导入其他proto文件会用到
package greeter;
//指定生成go程序的位置和包名(不是包文件名)，路径的最后一位是目录名，也是go程序的包名，即packege greeter。(不是go程序文件名，go程序文件名和proto文件同名，greeter.pb.go) 
option go_package="proto/greeter";


service Greeter{
	rpc SayHello (HelloReq) returns(HelloResp){}
}
message HelloReq {
    string name = 1;
}

message HelloResp {
    string message = 1;
} 
```

### 编译命令

```bash
 protoc --proto_path=. --go_out=. proto/greeter/greeter.proto

#--proto_path=PATH 它表示要在哪个路径下搜索.proto文件，
#这个参数也可以用-I 指定。如果不指定该参数，则默认在当前路径下进行搜索；
#另外，该函数可以指定多次，这也意味着可以指定多个路径进行搜索。
#下面三条指令，表达的含义都是一样的
$ protoc --go_out=. proto/greeter/greeter.proto

# 点号表示当前路径，注意-I参数没有等于号
$ protoc -I. --go_out=. proto/greeter/greeter.proto 

$ protoc --proto_path=. --go_out=. proto/greeter/greeter.proto 


```

### 语言插件参数

> *即–cpp_out=，–python_out=等。如果protoc已经内置语言对应的编译插件，则无需再安装。*
>
> *如果没有内置语言，就需要单独安装插件，比如–go_out=，对应的就是protoc-gen-go，安装命令如下：*
>
> ```bash
> # 最新版
> $ go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
>
> # 指定版本
> $ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.3.0 
> ```
>
> **proto文件位置参数**
>
> proto文件位置参数即@`<filename>`参数，指定了proto文件的具体位置，如proto/greeter/greeter.proto。
>
> proto文件有两个易混参数，即package和xx_package，xx是编译语言，比如要编译成Go语言，对应的就是go_package。
>
> ***package***
>
> package参数针对的是protobuf，是proto文件的命名空间，它的作用是为了避免定义的接口或者message出现命名冲突。
>
> 比如有a.proto和b.proto两个文件，内容如下：
>
> ```
>  # a.proto
> message PointInfo {
>     uint64 id = 1;
>     string name = 2;
> }
>
> # b.proto
> message PointInfo {
>     uint64 id = 1;
>     string name = 2;
> }  
> ```
>
> 两个文件都有一个PointInfo的message，如果需要在a文件引用b文件，如果没有指定package，就无法区分是要调a的还是调b的。
>
> **xx_package**
>
> 比如go_package，该参数主要声明go代码的存放位置
>
> .pb.go存放路径一般是在同名proto文件下，如果想把所有.pb.go文件都放在一个特定文件夹 下（pb_go），有两种办法：
>
> 第一种：
>
> ```
>  # 修改 --go_out，go_package 保持不变
> $ protoc --proto_path=. --go_out=./proto/pb_go proto/greeter/greeter.proto
> 这样生成的pb文件在 pb_go/proto/greeter 目录下，pb文件的包名仍然是 greeter  
> ```
>
> 第二种：
>
> ```
>  # 修改 go_package， go_out 保持不变
> option go_package="proto/pb_go";
>
> $ protoc --proto_path=. --go_out=. proto/greeter/greeter2.proto
> 这样生成的pb文件在 pb_go 目录下，pb文件的包名为 pb_go 
> ```

### 来源

> http://www.guoxiaolong.cn/blog/?id=12009
