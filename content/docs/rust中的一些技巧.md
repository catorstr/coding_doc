# Rust中的一些编程技巧

## go中控制并发的方式

> go的一大优点是它的高并发，轻量级的协程为我们提供了很多的方便，但是有时候协程并不是开的越多越好，而这时需要我们去控制协程的并发数量。

通过channel去控制协程的数量，如果然后再通过使用WaitGroup来等待协程处理完任务。

```go
func worker(id int, jobs <-chan struct{}, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("test%v\n", id)
	time.Sleep(1 * time.Second) //工作,你要执行任务的函数
	<-jobs
}

func TestWWorker(t *testing.T) {
	workers := 50
	xcTotal := 3
	//使用channel来控制协程的数量
	//创建一个缓冲区大小为n的channel
	//当执行一个新的协程时，先向channel中发送一个元素，如果channel已满，则会阻塞，
	//直到有协程执行完毕并取走一个元素，才会继续执行新的协程。
	//主线程使用sync.WaitGroup来等待协程处理完任务
	jobs := make(chan struct{}, xcTotal)
	var wg sync.WaitGroup
	//创建协程
	for i := 0; i < workers; i++ {
		jobs <- struct{}{}
		wg.Add(1)
		go worker(i, jobs, &wg)
	}
	wg.Wait()
	fmt.Println("等待。。。")
}

```

#### 好用的第三方库

```
"github.com/sourcegraph/conc"
```

详细教程：https://gitee.com/huoyingwhw/go-conc-share

## rust中对并发量的控制

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

> 在这段代码中，在main函数中定义了数量为4的信号量,然后其了十个协程去处理(person函数)，在person函数中调用了teller函数，teller函数需要一个接收信号permit才会继续往下执行，因为我们只有四个信号量，虽然在外边同时有十个协程在调用person函数，但是在执行到teller函数时，最多只能有四个协程在同时执行。这段代码展示了如何使用信号量来控制并发访问，模拟了人员排队等待服务的场景。

## rust中协程间的相互等待

> 在Go语言中，可以使用channel来实现多个协程之间的相互通信与相互等待。如：
>
> ```go
> package main
>
> import (
> 	"fmt"
> 	"time"
> )
>
> func worker1(ch chan string) {
> 	time.Sleep(2 * time.Second)
> 	ch <- "worker1: task done"
> }
>
> func worker2(ch chan string) {
> 	time.Sleep(3 * time.Second)
> 	ch <- "worker2: task done"
> }
>
> func main() {
> 	ch1 := make(chan string)
> 	ch2 := make(chan string)
>
> 	go worker1(ch1)
> 	go worker2(ch2)
>
> 	// 等待worker1和worker2完成任务
> 	select {
> 	case result1 := <-ch1:
> 		fmt.Println(result1)
> 	case result2 := <-ch2:
> 		fmt.Println(result2)
> 	}
> }
>
> ```

### rust中线程间通信的一种方式:

```rust
use std::sync::Arc;

use tokio::sync::Notify;
use tokio::time::{sleep,Duration};

async fn order_pacakge(packge_delivered:Arc<Notify>){
    sleep(Duration::from_secs(2)).await;
    println!("find a package in warehouse");
    sleep(Duration::from_secs(2)).await;
    println!("ship package");
    sleep(Duration::from_secs(2)).await;
    println!("package has been delivered");
    packge_delivered.notify_one();
    // packge_delivered.notify_waiters();
}

async fn grap_package(package_delivered:Arc<Notify>){
    package_delivered.notified().await;
    println!("look outside house for package");
    sleep(Duration::from_secs(2)).await;
    println!("grab package");
}

#[tokio::main]
async fn main(){
    let package_delivered =Notify::new();
    let package_delivered_arc = Arc::new(package_delivered);

    let order_pacakage_handle = tokio::spawn(
        order_pacakge(package_delivered_arc.clone())
    );
    let grap_package_handle = tokio::spawn(
        grap_package(package_delivered_arc.clone())
    );
  
    order_pacakage_handle.await.unwrap();
    grap_package_handle.await.unwrap();
}
```

### rust并发中使用屏障

模拟多个任务协同工作的情况，通过 `Barrier` 和 `Notify` 实现任务之间的同步与协调。

```rust
use std::{sync::Arc, time::Duration};

use tokio::{sync::{Barrier, BarrierWaitResult, Notify}, time::sleep};

async fn barrier_example(barrier:Arc<Barrier>,notify:Arc<Notify>)->BarrierWaitResult{
    println!("waiting at barrier");
    let wait_result  = barrier.wait().await;
    println!("passed through the barrier");
    if wait_result.is_leader(){
        notify.notify_one()
    }
    wait_result
}

#[tokio::main]
async fn main(){
    let total_cans_needed = 12;
    let barrier = Arc::new(Barrier::new(total_cans_needed));
    let notify = Arc::new(Notify::new());
    let mut task_handle = Vec::new();
    notify.notify_one();
    for can_count in 0..60{
        if can_count%12==0{
            notify.notified().await;
            sleep(Duration::from_secs(1)).await;
        }
        task_handle.push(tokio::spawn(
            barrier_example(barrier.clone(), notify.clone())
        ))
    }
    let mut num_of_leaders = 0;
    for handle in task_handle  {
        let wait_result = handle.await.unwrap();
        if wait_result.is_leader(){
            num_of_leaders+=1;
        }
    }
    println!("total num of lerders:{}",num_of_leaders);
}
```

> 在 Go 中，可以使用 `sync` 包中的 `WaitGroup` 和 `Cond` 来实现类似的多任务协同工作模型。下面是一个类似的示例代码:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func barrierExample(wg *sync.WaitGroup, cond *sync.Cond, isLeader *bool) {
	defer wg.Done()

	fmt.Println("waiting at barrier")
	cond.L.Lock()
	cond.Wait()
	cond.L.Unlock()
	fmt.Println("passed through the barrier")

	if *isLeader {
		cond.Signal()
	}
}

func main() {
	totalCansNeeded := 12
	var wg sync.WaitGroup
	var mu sync.Mutex
	cond := sync.NewCond(&mu)
	isLeader := false

	wg.Add(60)
	for canCount := 0; canCount < 60; canCount++ {
		if canCount%12 == 0 {
			mu.Lock()
			isLeader = true
			cond.Broadcast()
			mu.Unlock()
			time.Sleep(time.Second)
		}
		go barrierExample(&wg, cond, &isLeader)
	}

	wg.Wait()
	fmt.Println("all tasks have passed through the barrier")
}

```
