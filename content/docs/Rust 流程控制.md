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

但是如果预期不是一个布尔值，编译阶段就会报错。(和go一样)

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
/*
number：3
number：2
number：1
end
*/
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

### 习题一

> ```rust
> use std::io;
> fn main() {
> /*
> 1.在华氏和摄氏之间转换温度
> 华氏度＝32＋摄氏度×1.8
>  */
>   loop {
>     println!("1 华氏传摄氏");
>     println!("2 摄氏传华氏");
>     println!("请选择");
>   let mut choose = String::new();
>   io::stdin()
>     .read_line(&mut choose)
>     .expect("input not is num");
>
>   let choose:u32 =match choose.trim().parse(){
>     Ok(num)=>num,
>     Err(_)=>continue,
>   };
>     if choose ==1{
>       println!("您的选择是： 1 华氏传摄氏");
>       println!("请输入您要抓换的 华氏温度：");
>       let mut temperature1 = String::new();
>       io::stdin().read_line(&mut temperature1).expect("input err");
>       let temperrature:f32 =match temperature1.trim().parse(){
>         Ok(num) =>num,
>         Err(_)=>continue,
>       };
>       //摄氏度 =(华氏度-32)/1.8
>       println!("华氏度{temperature1}对应的摄氏度是{}",(temperrature-32.0)/1.8)
>     }else if choose==2{
>       println!("您的选择是： 2 摄氏传华氏");
>       println!("请输入您要抓换的 摄氏温度：");
>       let mut temperature2 = String::new();
>       io::stdin().read_line(&mut temperature2).expect("input err");
>       let temperrature:f32 =match temperature2.trim().parse(){
>         Ok(num) =>num,
>         Err(_)=>continue,
>       };
>       //华氏度＝32＋摄氏度×1.8
>       println!("摄氏度{temperature2}对应的华氏度是{}",(temperrature+32.0)*1.8)
>     }else{
>       println!("请重新输入");
>       continue;
>     }
>   }
> }
>
> ```

### 习题二

> ```rust
> use std::io;
> fn main(){
>   /*
>   2.生成第n个斐波那契数
> 斐波那契数列（Fibonacci sequence），又称黄金分割数列，因数学家莱昂纳多·斐波那契（Leonardo Fibonacci）以兔子繁殖为例子而引入，故又称为“兔子数列”，指的是这样一个数列：1、1、2、3、5、8、13、21、34、……在数学上，斐波那契数列以如下被以递推的方法定义：F(0)=1，F(1)=1, F(n)=F(n - 1)+F(n - 2)（n ≥ 2，n ∈ N*）
>  */
> loop{
>   println!("请输入您要生成的斐波那契数的个数n:");
>   let mut n= String::new();
>   io::stdin()
>     .read_line(&mut n)
>     .expect("input err");
>   let n:u32 = match n.trim().parse(){
>     Ok(num)=>num,
>     Err(_)=>continue,
>     };
>     //计算斐波那契数列
>     if n==0{
>     println!("0的斐波那契数列是1");
>   }else if n ==1{
>      println!("1的斐波那契数列是1、1");
>   }else {
>       let mut strn =String::from("的斐波那契数列是1、1");
>       let fg = String::from("、");
>       for i in 2..=n {
>         let num= fnf(i);
>         let num = num.to_string();
>         strn.push_str(&fg);
>         strn.push_str(&num);
>        if i==n{
>           println!("{}{}",n,strn);
>         }
>       }
>   }
>  
> }
>
> }
> //计算斐波那契数
> fn fnf(num:u32)-> u64{
>     if num ==0{
>       return  1;
>     }else if num==1{
>       return 1;
>     }
>     return  fnf(num - 1)+fnf(num-2);
> }
> ```
