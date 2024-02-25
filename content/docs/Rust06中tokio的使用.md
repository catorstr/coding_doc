# Rust中Tokio的使用

> https://tokio.rs/tokio/tutorial
>
> tokio是rust的异步编程框架，接下里建使用tokio这个库来写一个简单的即时通信系统。

### hello tokio

> ```rsut
> use mini_redis::{client,Result};
>
> #[tokio::main]
> async fn main()->Result<()> {
>     let mut client = client::connect("127.0.0.1:6379").await?;
>     client.set("hello","world".into()).await?;
>     let result = client.get("hello").await?;
>     println!("result = {:?}",result);
>     Ok(())
> }
>
> /*
> 该 #[tokio::main] 函数是一个宏。
> 它将 转换为 async fn main() 同步 fn main()，
> 用于初始化运行时实例并执行异步主函数。
> #[tokio::main]
> async fn main() {
>     println!("hello");
> }
> 被转换为：
> fn main(){
>     let mut rt = tokio::runtime::Runtime::new().unwrap();
>     rt.blok_on(async{
>         println!("hello");
>     })
> }
> */
> ```
>
> create server
>
> ```rust
> use tokio::net::{TcpListener,TcpStream};
> use mini_redis::{Connection,Frame};
>
> #[tokio::main]
> async fn main(){
>     let listener = TcpListener::bind("127.0.0.1:6379").await.unwrap();
>     loop{
>         let (socket,_) = listener.accept().await.unwrap();
>         //process(socket).await; //每一个socket连接进来会在这里发生阻塞
>    
>         //并发处理
>         /*
>         tokio 中的任务非常轻量级。在后台，它们只需要一个分配和 64 字节的内存。
>         应用程序应该可以自由生成数千甚至数百万个任务。
>         */
>         tokio::spawn(async move{
>             process(socket).await;
>         });
>
>         //tokio::spawn块可能有返回值，可以这样获取它的返回值
>         let handle = tokio::spawn(async{
>             //Do some async work...;
>             "retrurn value"
>         });
>         //do some other work...
>         //处理返回值
>         let out = handle.await.unwrap();
>         println!("Got:{}",out)
>         /*
>         等待返回 JoinHandle Result .
>         当任务在执行过程中遇到错误时，将 JoinHandle 返回 Err .
>         当任务出现恐慌，或者运行时关闭强制取消任务时，会发生这种情况。
>         */
>     }
> }
>
> async fn process(socket:TcpStream){
>     let mut connetction = Connection::new(socket);
>
>     if let Some(frame) =connetction.read_frame().await.unwrap(){
>         println!("Got: {:?}",frame);
>     }
>     //Respond with error
>     let response = Frame::Error("error msg".to_string());
>     connetction.write_frame(&response).await.unwrap();
>
> }
> ```
>
> Store values
>
> ```rust
> use mini_redis::{Connection, Frame};
> use tokio::net::{TcpListener, TcpStream};
>
> #[tokio::main]
> async fn main() {
>     let listener = TcpListener::bind("127.0.0.1:6379").await.unwrap();
>     loop {
>         let (socket, _) = listener.accept().await.unwrap();
>         tokio::spawn(async move {
>             process(socket).await;
>         });
>
>     }
> }
>
> async fn process(socket: TcpStream) {
>     use mini_redis::Command;
>     use std::collections::HashMap;
>
>     let mut db = HashMap::new();
>
>     let mut connection = Connection::new(socket);
>     while let Some(frame) = connection.read_frame().await.unwrap() {
>         let response = match Command::from_frame(frame).unwrap() {
>             Command::Set(cmd) => {
>                 db.insert(cmd.key().to_string(), cmd.value().to_vec());
>                 Frame::Simple("OK".to_string()) //这里一定要写OK才行，其他的不行，好奇快
>             }
>             Command::Get(cmd) => {
>                 if let Some(value) = db.get(cmd.key()) {
>                     Frame::Bulk(value.clone().into())
>                 } else {
>                     Frame::Null
>                 }
>             }
>             Command::Publish(_) => todo!(),
>             Command::Subscribe(_) => todo!(),
>             Command::Unsubscribe(_) => todo!(),
>             Command::Unknown(cmd) => panic!("unkown:{:?}", cmd),
>         };
>         connection.write_frame(&response).await.unwrap();
>     }
> }
> /*
> 我们现在可以获取和设置值，但存在一个问题：这些值不在连接之间共享。
> 如果另一个套接字连接并尝试使用 GET hello 密钥，它将找不到任何内容。
> */
>
> ```
>
> shared state（共享状态）

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

### tokio中函数的调用

```rust
#[tokio::main]
async fn main() {
    //1. tokio中使用普通fn
    tokio::spawn(async {
        hello();
    });
    //2. 有时普通函数可能发生阻塞，这时我们希望程序在此期间执行其他任务时
    let rt = tokio::task::spawn_blocking(blocking_call);
    //do some other ...
    println!("world");
    let res = rt.await.unwrap();
    println!("res = {res}");

    //3.异步阻塞函数
    let mut async_handles = Vec::new();
    for num in 0..10 {
        async_handles.push( tokio::spawn(async_call(num)));
    }
    println!("rust..");
    //处理返回的结果
    for handle in async_handles{
        handle.await.unwrap();
    }
}

//普通函数
fn hello() {
    println!("hello")
}
//普通阻塞函数
use std::{thread, time};
fn blocking_call() -> String {
    thread::sleep(time::Duration::from_secs(2));
    "blocking call end ...".to_string()
}
//异步阻塞函数
use tokio::time::{sleep, Duration};
async fn async_call(num: i32) {
    sleep(Duration::from_secs(5)).await;
    println!("async call num = {num}");
}

/*
world
hello
res = blocking call end ...
rust..
async call num = 6
async call num = 1
async call num = 4
async call num = 8
async call num = 9
async call num = 2
async call num = 0
async call num = 3
async call num = 5
async call num = 7

*/
```

### tokio中互斥锁的使用

```rust
use std::sync::Arc;
use tokio::sync::Mutex;
#[tokio::main]
async fn main() {
    let t_num = 10;
    let remote = Mutex::new(t_num);
    let remote_arc = Arc::new(remote); //使之变成线程安全的

    let mut async_handles = Vec::new();
    for num in 0..10 {
        async_handles.push(tokio::spawn(process(num,remote_arc.clone())));
    }
    for handle in async_handles {
        handle.await.unwrap();
    }
}

use tokio::time::{sleep, Duration};
async fn process(num: i32, t_num: Arc<Mutex<i32>>) {
     //加锁，返回其对象(锁对象)的引用，在这里被锁的对象是t_num是一个i32类型
    let mut rx = t_num.lock().await;
    sleep(Duration::from_secs(1)).await;
    *rx += 1; //解引用并修改其(锁对象)值
    println!("num = {num},rx = {rx}");
}

/*
num = 0,rx = 11
num = 2,rx = 12
num = 3,rx = 13
num = 5,rx = 14
num = 6,rx = 15
num = 7,rx = 16
num = 4,rx = 17
num = 9,rx = 18
num = 8,rx = 19
num = 1,rx = 20
*/
```

### tokio中的读写锁

```rust
use std::sync::Arc;

use tokio::sync::RwLock;
use tokio::time::{sleep,Duration};

async fn read_from_document(id:i32,document:Arc<RwLock<String>>){
    let reader = document.read().await;
    println!("reader_{id}:{reader}");
}
async fn write_to_document(document:Arc<RwLock<String>>,new_string:&str){

    let mut writer = document.write().await;
    writer.push_str(new_string);
    writer.push_str(" ");
}
#[tokio::main]
async fn main(){

    let some_string = "some string a b c d e f g h i j k l q t y i o p s w v x w  m n z";

    let document =Arc::new(RwLock::new("".to_string()));
    let mut handhles = Vec::new();
    for new_string in some_string.split_whitespace(){
        handhles.push(tokio::spawn(read_from_document(1, document.clone())));
        handhles.push(tokio::spawn(write_to_document(document.clone(), new_string)));
        sleep(Duration::from_millis(1)).await;
        handhles.push(tokio::spawn(read_from_document(2, document.clone())));
        handhles.push(tokio::spawn(read_from_document(3, document.clone())));
    }
    for handle in handhles{
        handle.await.unwrap();
    }
}    
```

###  tokio中信号量的使用

> 假设，银行有四个窗口在为用户办理业务，四个窗口各有一张票充当信号量，只有拿到票的用户才有资格办理业务，当它办理完业务后，这张票会被转移给另一个人。

```rust
use std::sync::Arc;
use tokio::sync::Semaphore;
use tokio::time::{sleep, Duration};

//办理业务的人
async fn person(name: String, semaphore: Arc<Semaphore>) {
    //假设现在窗口都有人，而此时name这个人正在排队等待
    println!("{} is waiting line ...", name);
    //此时，他接收到了一个信号(没错排到他了)
    teller(name, semaphore).await;
}

async fn teller(customer: String, semaphore: Arc<Semaphore>) {
    //获取许可证
    let permit = semaphore.acquire().await.unwrap();
    //到柜台处理业务...
    sleep(Duration::from_secs(2)).await;
    println!("{} is being served by teller", customer);
    sleep(Duration::from_secs(1)).await;
    println!("{}ids now leaving the teller", customer);
    drop(permit);
}

#[tokio::main]
async fn main() {
    //定义柜台数量
    let num_of_teller = 4;
    let semaphore = Semaphore::new(num_of_teller);
    let semaphore_arc = Arc::new(semaphore);

    let mut people_handles = Vec::new();
    //假设现在有10个人
    for num in 0..10 {
        people_handles.push(tokio::spawn(person(
            format!("perple_{num}"),
            semaphore_arc.clone(),
        )));
    }
    //主线程等待任务结束
    for handle in people_handles{
        handle.await.unwrap();
    }
}
/*
perple_0 is waiting line ...
perple_1 is waiting line ...
perple_2 is waiting line ...
perple_6 is waiting line ...
perple_7 is waiting line ...
perple_8 is waiting line ...
perple_4 is waiting line ...
perple_9 is waiting line ...
perple_5 is waiting line ...
perple_3 is waiting line ...
perple_6 is being served by teller
perple_1 is being served by teller
perple_0 is being served by teller
perple_2 is being served by teller
perple_2ids now leaving the teller
perple_1ids now leaving the teller
perple_6ids now leaving the teller
perple_0ids now leaving the teller
perple_4 is being served by teller
perple_7 is being served by teller
perple_8 is being served by teller
perple_9 is being served by teller
perple_8ids now leaving the teller
perple_4ids now leaving the teller
perple_7ids now leaving the teller
perple_9ids now leaving the teller
perple_3 is being served by teller
perple_5 is being served by teller
perple_5ids now leaving the teller
perple_3ids now leaving the teller 
*/


```

### tokio中Notify的使用

> 假设在商场系统中，用户下单后，其购买的商品会被送到用户家门口，当商品到达用户家门时会通知他去领取。

```rust
use std::sync::Arc;

use tokio::sync::Notify;
use tokio::time::{sleep,Duration};

async fn order_package(package_delivered:Arc<Notify>){
    sleep(Duration::from_secs(2)).await;
    println!("find package in warehouse");
    sleep(Duration::from_secs(3)).await;
    println!("ship package");
    sleep(Duration::from_secs(1)).await;
    println!("package has been delivered");
    package_delivered.notify_one();//发出通知
}
async fn grab_package(package_delivered:Arc<Notify>){
    package_delivered.notified().await; //接收到通知
    println!("look putside house for package");
    sleep(Duration::from_secs(1)).await;
    println!("grab package");

}

#[tokio::main]
async fn main(){

    let package_delivered = Notify::new();
    let package_delivered_arc = Arc::new(package_delivered);
    let order_package_handle =  tokio::spawn(
        order_package(package_delivered_arc.clone())
    );
    let grab_package_handle = tokio::spawn(grab_package(package_delivered_arc.clone()));
  
    order_package_handle.await.unwrap();//这两个并不分先后
    grab_package_handle.await.unwrap(); 
}
/*
find package in warehouse
ship package
package has been delivered
look putside house for package
grab package

*/

```

### 什么是屏障？

> 屏障是一种异步机制，可确保一定数量的任务，在继续执行之前到达集合点。假设，我开了一家制造汽水的公司，现在有一个任务是生产一瓶汽水，另一个任务是当生产的汽水数量达到12瓶时将它们打包带走，为了使这两个异步任务协同工作，需要设置一个屏障(障碍物)，一旦12瓶汽水到达屏障，将会打开屏障，让这12个瓶子被打包带走，然后关闭屏障，重复这个过程。
>
> 在这个过程中，为了每次打包都是12瓶汽水而不是13瓶或是更多，需要在屏障打开的这个时间点，暂定汽水生产的任务，这样该什么设计出我们的程序呢？

```rust
use std::sync::Arc;
use  tokio::sync::{Barrier,BarrierWaitResult,Notify};
use tokio::time::{sleep,Duration};


#[tokio::main]
async fn main(){

    //设置屏障，即所达到要求的汽水总数
    let total_cans_needed =12;
    let barrier = Arc::new(Barrier::new(total_cans_needed));

    let notify = Arc::new(Notify::new());

    let mut task_handles = Vec::new();
    //假设，我们这里要生产60瓶汽水
    //所以生产汽水任务会源源不断的产生汽水
    notify.notify_one();
    for can_count in 0..60{
        //如果汽水数量到达12个
        if can_count%12==0{//因为0%12==0 所以在此之前要发送一个通知
            notify.notified().await;//阻塞等待信号的到来

            //give the barrier some time to close
            sleep(Duration::from_millis(1)).await;
        }
        //开启异步任务
        task_handles.push(tokio::spawn(
            barrier_example(
                barrier.clone(),
                notify.clone(),
                )
        ));
    }

    //生产60瓶汽水，应该会有5个领导者
    let mut num_of_leaders = 0;
    for handle in task_handles{
        let wait_result = handle.await.unwrap();
        if wait_result.is_leader(){
            num_of_leaders+=1;
        }
    }
    println!("leaders = {num_of_leaders}")

}
async fn barrier_example(barrier:Arc<Barrier>,notify:Arc<Notify>)->BarrierWaitResult{
    println!("waiting at barrier");
    let wait_result = barrier.wait().await;
    println!("passed though the barrier");
    //每批汽水中有一个领导者，我们使用领导者作为信号，来给下一批汽水发送通知。
    if wait_result.is_leader(){//如果是领导者
        notify.notify_one();//发送通知给另外正在等待信号的任务
    };
    wait_result
}

/*
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
waiting at barrier     
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
waiting at barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
passed though the barrier
leaders = 5
*/
```
