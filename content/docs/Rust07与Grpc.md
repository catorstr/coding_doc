# Rust 与Grpc

```bash
# 新建一个rust项目
cargo new rust-grpc-demo --lib
#用vscode打开新建的项目
code rust-grpc-demo
#在Cargo.toml文件中导入需要的库

```

### 导入必要的包

```toml
# Cargo.toml

[package]
name = "rust-grpc-demo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
tonic = "0.5" #grpc服务所依赖的包
prost = "0.8" # .proto 序列化与反序列化
#grpc异步运行时
tokio = {version="1.0",features = ["macros","rt-multi-thread"]}

[build-dependencies]
# .proto 转rust grpc服务
tonic-build = "0.5"


```

### 定义./proto文件

> 定义一个say_hello服务。

```bash
vim proto/hello.proto
```

```bash
//cat proto/hello.proto
syntax="proto3";

package hello;

message HelloRequest{
    string name=1;
}

message HelloResponse{
    string greeting=1;
}

service Greeter{
    rpc SayHello(HelloRequest)returns(HelloResponse);
}
```

### 构建 .proto 文件

> 在项目的根目录下新建build.rs文件,编写一下内容，然后cargo build
>
> build.rs文件是cargo 的一个规则，在运行main.rs之间会先运行build.rs。

```rust
use std::path::PathBuf;

fn main(){
    //.proto 文件转rust grpc服务代码的输出路径
    let out_dir = PathBuf::from("src");
    //编译.proto文件
    tonic_build::configure()
        .out_dir(out_dir)
        .compile(&["proto/hello.proto"], &["proto"])
        .unwrap()
}
```

现在项目应该是这样子：

```bash
├── build.rs
├── Cargo.lock
├── Cargo.toml
├── proto
│   └── hello.proto
├── src
│   ├── hello.rs
│   └── lib.rs
└── target
```

> 为了使用"src/hello.rs",需要在lib.rs文件中声明：
>
> ```rust
> // src/lib.rs
> pub mod hello;
> ```

### 编写grpc  server端代码：

```rust
//server.rs
use std::net::SocketAddr;

use tonic::transport::Server;
use tonic::Request;
use tonic::Response;
use tonic::Status;


use grpc_demo::hello::greeter_server::Greeter;
use grpc_demo::hello::greeter_server::GreeterServer;
use grpc_demo::hello::HelloRequest;
use grpc_demo::hello::HelloResponse;

use tokio::runtime::Builder;


struct MyGreeter{}

#[tonic::async_trait]
impl Greeter for MyGreeter{
    async fn say_hello(&self,req:Request<HelloRequest>)->Result<Response<HelloResponse>,Status>{
        Ok(Response::new(HelloResponse{
            greeting:format!("Hello {}!",req.get_ref().name)
        }))
    }
}

fn main(){

    let rt = Builder::new_multi_thread().enable_all().build().unwrap();
    rt.block_on(async{
        //设置socket监听的 ip and port
        let addr:SocketAddr = "127.0.0.1:5001".parse().unwrap();
        //服务实例
        let greeter =MyGreeter{};
        Server::builder()
            .add_service(GreeterServer::new(greeter))
            .serve(addr)
            .await
            .unwrap();
    });
}
```

### 编写grpc  client端代码：

```rust
//client.rs
use grpc_demo::hello::greeter_client::GreeterClient;
use grpc_demo::hello::HelloRequest;


use tonic::Request;

use tokio::runtime::Builder;

fn main(){
    let rt = Builder::new_multi_thread().enable_all().build().unwrap();
    rt.block_on(async{
        let mut cli = GreeterClient::connect("http://127.0.0.1:5001")
            .await
            .unwrap();
      
        let req = Request::new(HelloRequest{
            name:"Alex".into(),
        });

        let res = cli.say_hello(req).await.unwrap();
        let hello_res = res.get_ref();
        println!("{:?}",hello_res)
    })
}

```

### 调用

```bash
//server
cargo run --bin server
   Compiling grpc-demo v0.1.0 (/root/workspace/rust-study/grpc-demo)
    Finished dev [unoptimized + debuginfo] target(s) in 12.18s
     Running `target/debug/server`
```

```bash
//client
cargo run --bin client
   Compiling grpc-demo v0.1.0 (/root/workspace/rust-study/grpc-demo)
    Finished dev [unoptimized + debuginfo] target(s) in 10.52s
     Running `target/debug/client`
HelloResponse { greeting: "Hello Alex!" }

```
