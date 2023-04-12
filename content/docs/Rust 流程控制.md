# Rust 流程控制语句

### if 表达式

Rust 中的 if 语句必须接收一个布尔值，与go一样，还可以省略括号。

```go
fn main() {
    let number = 1;
    if number < 2 {
        println!("true"); // true
    } else {
        println!("false");
    }
}
```

但是如果预期不是一个布尔值，编译阶段就会报错。(go 没有这个限制)

```go
fn main() {
    let number = 1;
    if number { //不是一个bool值会报错
        println!("true");
    }
}

```

运行之后，报错如下：

```bash
error[E0308]: mismatched types
 --> main.rs:3:8
  |
3 |     if number { 
  |        ^^^^^^ expected `bool`, found integer

error: aborting due to previous error

For more information about this error, try `rustc --explain E0308`.
```

在 let 中使用 if 表达式，注意 if else 分支的数据类型要一致，因为 Rust 是静态类型，需要在编译期间确定所有的类型。

```go
fn main() {
    let condition = true;
    let num = if condition { 1 } else { 0 };
    println!("num: {}", num); // 1
}
```



### loop 循环

loop 表达式会无限的循环执行代码块，如果想终止循环，可配合 break 语句使用。

```go
fn main() {
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };

    println!("result: {}", result); // 20
}
```

### while 循环

使用 while 可以加上条件判断决定是否还要循环多少次，如果条件为 true 继续循环，条件为 false 则退出循环。

```go
fn main() {
    let mut counter = 3;
    while counter != 0 {
        println!("counter: {}", counter);
        counter -= 1;
    }
    println!("end");
}
```

### for 循环

使用 for 循环遍历集合元素，例如在访问一个数组时，增加了程序的安全性不会出现超出数组大小或读取长度不足的情况。

```go
fn main() {
    let arr = ['a', 'b', 'c'];
    for element in arr.iter() {
        println!("element: {}", element);
    }
    println!("end");
}
```

在 Rust 中使用 for 循环的另一种方式。

```go
fn main() {
    for number in (1..4).rev() {
        println!("number：{}", number);
    }
    println!("end");
}
```

```rust
fn main() {
    let arr:[i32;5] = [1,2,3,4,5];
    for i in arr.iter(){
        println!("i = {}",i);
    }
    println!("------------");
    for i in (1..4).rev(){
        println!("i ={}",i);
    }
/*
i = 1
i = 2
i = 3
i = 4
i = 5
------------
i =3
i =2
i =1
 */
}

```
