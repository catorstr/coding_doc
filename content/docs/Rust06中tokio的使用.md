# Rust中Tokio的使用

> https://tokio.rs/tokio/tutorial
>
> tokio是rust的异步编程框架，接下里建使用tokio这个库来写一个简单的即时通信系统。

### 创建一个监听IP:Port的服务

> ```toml
> #Cargo.toml
> [dependencies]
> tokio = { version = "1.32.0", features = ["full"] }
> ```
>
> ```rust
> use tokio::net::TcpListener;
> use tokio::io::{AsyncReadExt,AsyncWriteExt,BufReader};
>
> #[tokio::main]
> async fn main() -> Result<(), Box<dyn std::error::Error>> {
>     let listener = TcpListener::bind("127.0.0.1:8080").await?;
>
>     loop {
>         let (mut socket, addr) = listener.accept().await?;
>         println!("{} connected",addr)
>         tokio::spawn(async move {
>           
>             let mut msg = BufReader::new();
>         });
>     }
> }
> ```


```rust
use tokio::{io::BufReader,net::TcpListener};
const LOACL_SERVER:&str = "127.0.0.1:8888";

//使用tokio异步运行时来写我们的程序
#[tokio::main]
async fn main() ->Result<(),Box<dyn std::error::Error>>{

    //绑定服务监听地址
    let listener  =TcpListener::bind(LOACL_SERVER).await?;
    loop{ 
        //接收客户端的连接
        let (mut socket,addr) = listener.accept().await?;
        println!("{} connected",addr);
        //这里相当于使用go的携程去处理客户端连接
        tokio::spawn(async move{
            //socket.split()会将socket连接进行读写分离
            let (reader,mut writer) = socket.split();
            //读取客户端的消息
            let mut reader = BufReader::new(reader);
            let mut msg = String::new();
            loop{
                //read_line 按行读取客户端消息到msg中，并返回读取的字节数
                let read_n= reader.read_line(&mut msg).await.unwrap();
                if read_n ==0{
                    break;//读取不到值时直接退出
                }
                println!("recv client msg: {}",msg);
              
                //给客户端回个消息吧
                writer.write_all(msg.as_bytes()).await.unwrap();
                msg.clear();//清理缓存
            }
        });
      
    }
}
```
