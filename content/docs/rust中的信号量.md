# Rust中信号量的使用

```rust
use std::sync::Arc;
use tokio::sync::Semaphore;
use tokio::time::{sleep,Duration};

//rust中信号量的使用
pub async fn person(semaphore:Arc<Semaphore>,name:String){
    println!("{} is waiting in line",name);
    teller(semaphore,name).await;
}

async fn teller(semaphore:Arc<Semaphore>,customer:String){
    let permit = semaphore.acquire().await.unwrap();
    sleep(Duration::from_secs(2)).await;
    println!("\n{} is being served by the teller",customer);
    sleep(Duration::from_secs(5)).await;
    println!("{} is now leaving the teller",customer);
    drop(permit);
}

#[tokio::main]
async fn main(){
        let num_of_tellers = 4;
        let semaphore = Semaphore::new(num_of_tellers);
        let semaphore_arc = Arc::new(semaphore);
        let mut people_handles = Vec::new();
        for num in 0..10{
            people_handles.push(
                tokio::spawn(
                    datarust::semaphore::person(semaphore_arc.clone(),  format!("Proson_{num}"))
            ));
        }
        for handle in people_handles {
            handle.await.unwrap();
        }
}
```

> 在这段代码中，在main函数中定义了数量为4的信号量,然后其了十个协程去处理(person函数)，在person函数中调用了teller函数，teller函数需要一个接收信号permit才会继续往下执行，因为我们只有四个信号量，虽然在外边同时有十个携程在调用person函数，但是在执行到teller函数时，最多只能有四个协程在同时执行。这段代码展示了如何使用信号量来控制并发访问，模拟了人员排队等待服务的场景。
