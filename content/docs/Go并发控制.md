# Go并发控制

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
