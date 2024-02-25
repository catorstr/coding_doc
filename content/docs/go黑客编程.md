# Go 黑客编程

> 基于汤姆斯蒂尔编写的《Go黑帽子渗透测试编程之道》的学习，博客中的一切代码仅用于个人学习，若有人恶意利用，后果自负。

### http请求

> 在访问服务器的时候，经常有ip访问次数限制或访问权重等问题，设置有效的请求策略是一个不错的idea.
>
> 例如：服务器ip限制每秒100次的访问。
>
> 1. 使用令牌桶算法来控制每秒访问次数的 Go 示例代码：
>    ```go
>    package main
>
>    import (
>    	"fmt"
>    	"sync"
>    	"time"
>    )
>
>    type TokenBucket struct {
>    	tokens        int
>    	capacity      int
>    	refillAmount  int
>    	refillInterval time.Duration
>    	mu            sync.Mutex
>    }
>
>    func NewTokenBucket(capacity, refillAmount int, refillInterval time.Duration) *TokenBucket {
>    	return &TokenBucket{
>    		tokens:        capacity,
>    		capacity:      capacity,
>    		refillAmount:  refillAmount,
>    		refillInterval: refillInterval,
>    	}
>    }
>
>    func (tb *TokenBucket) refill() {
>    	for {
>    		time.Sleep(tb.refillInterval)
>    		tb.mu.Lock()
>    		tb.tokens = tb.capacity
>    		tb.mu.Unlock()
>    	}
>    }
>
>    func (tb *TokenBucket) Allow() bool {
>    	tb.mu.Lock()
>    	defer tb.mu.Unlock()
>
>    	if tb.tokens > 0 {
>    		tb.tokens--
>    		return true
>    	}
>
>    	return false
>    }
>
>    func main() {
>    	tb := NewTokenBucket(100, 100, time.Second)
>    	go tb.refill()
>
>    	for i := 1; i <= 200; i++ {
>    		if tb.Allow() {
>    			fmt.Printf("Request %d allowed\n", i)
>    		} else {
>    			fmt.Printf("Request %d denied\n", i)
>    		}
>    		time.Sleep(10 * time.Millisecond) // 模拟请求间隔
>    	}
>    }
>
>    ```
> 2. 使用滑动窗口算法来控制每秒访问次数的 Go 示例代码：
>    ```go
>    package main
>
>    import (
>    	"fmt"
>    	"sync"
>    	"time"
>    )
>
>    type SlidingWindow struct {
>    	windowSize time.Duration
>    	limit      int
>    	window     []time.Time
>    	mu         sync.Mutex
>    }
>
>    func NewSlidingWindow(windowSize time.Duration, limit int) *SlidingWindow {
>    	return &SlidingWindow{
>    		windowSize: windowSize,
>    		limit:      limit,
>    		window:     make([]time.Time, 0),
>    	}
>    }
>
>    func (sw *SlidingWindow) Allow() bool {
>    	sw.mu.Lock()
>    	defer sw.mu.Unlock()
>
>    	now := time.Now()
>
>    	// 清理过期的时间窗口
>    	for len(sw.window) > 0 && now.Sub(sw.window[0]) > sw.windowSize {
>    		sw.window = sw.window[1:]
>    	}
>
>    	// 检查当前窗口内的请求数是否超过限制
>    	if len(sw.window) < sw.limit {
>    		sw.window = append(sw.window, now)
>    		return true
>    	}
>
>    	return false
>    }
>
>    func main() {
>    	sw := NewSlidingWindow(time.Second, 100)
>
>    	for i := 1; i <= 200; i++ {
>    		if sw.Allow() {
>    			fmt.Printf("Request %d allowed\n", i)
>    		} else {
>    			fmt.Printf("Request %d denied\n", i)
>    		}
>    		time.Sleep(10 * time.Millisecond) // 模拟请求间隔
>    	}
>    }
>
>    ```
> 3. 使用漏桶算法来控制每秒访问次数的 Go 示例代码：
>    ```go
>    package main
>
>    import (
>    	"fmt"
>    	"sync"
>    	"time"
>    )
>
>    type LeakyBucket struct {
>    	capacity     int
>    	leaksPerSec  int
>    	leaks        int
>    	lastLeakTime time.Time
>    	mu           sync.Mutex
>    }
>
>    func NewLeakyBucket(capacity, leaksPerSec int) *LeakyBucket {
>    	return &LeakyBucket{
>    		capacity:     capacity,
>    		leaksPerSec:  leaksPerSec,
>    		leaks:        0,
>    		lastLeakTime: time.Now(),
>    	}
>    }
>
>    func (lb *LeakyBucket) Allow() bool {
>    	lb.mu.Lock()
>    	defer lb.mu.Unlock()
>
>    	now := time.Now()
>
>    	// 计算时间间隔内漏水数量
>    	elapsed := now.Sub(lb.lastLeakTime)
>    	leakAmount := int(elapsed.Seconds() * float64(lb.leaksPerSec))
>
>    	// 更新漏桶状态
>    	lb.leaks -= leakAmount
>    	if lb.leaks < 0 {
>    		lb.leaks = 0
>    	}
>    	lb.lastLeakTime = now
>
>    	// 检查漏桶是否溢出
>    	if lb.leaks < lb.capacity {
>    		lb.leaks++
>    		return true
>    	}
>
>    	return false
>    }
>
>    func main() {
>    	lb := NewLeakyBucket(100, 100)
>
>    	for i := 1; i <= 200; i++ {
>    		if lb.Allow() {
>    			fmt.Printf("Request %d allowed\n", i)
>    		} else {
>    			fmt.Printf("Request %d denied\n", i)
>    		}
>    		time.Sleep(10 * time.Millisecond) // 模拟请求间隔
>    	}
>    }
>
>    ```
> 4. orther...
