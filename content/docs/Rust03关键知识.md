# rust的关键知识

### rust中的enum

> enum的所用和大多数语言中的概念和作用是一样的。

```rust
//但在rust中枚举有更高级的用途
pub enum Res<T, E> {
    Thing(T),
    Error(E),
}
fn divide(a: i32, b: i32) -> Res<i32, String> {
    if b == 0 {
        return Res::Error("Cannot Divide by zero".to_string());
    }
    Res::Thing(a / b)
}
fn main() {
    let a = divide(10, 5);
    let b = divide(10, 0);
    //1. match 匹配所有可能的情况
    match a {
        Res::Thing(v) => println!("val= {}", v),
        Res::Error(v) => println!("val = {}", v),
    }
    //2. 当我们只想匹配一种情况时
    // match b{
    //     Res::Thing(v) =>println!("val= {}",v), //只匹配我们需要的情况
    //     _ =>{}, //忽略其他的情况,什么也不做
    // }
    //3. 2的方式有一种更好的写法:
    if let Res::Thing(v) = b {
        //这里代表，当b是Res::Ting(T)类型时，我们将a赋值给v
        println!("val = {}", v);
    }
    println!("-------------------");
    let a = divide2(10, 2);
    let b = divide2(5, 0);
    if let Result::Ok(v) = a {
        println!("val = {}", v)
    }
    println!("a = {:?} ,b ={:?}", a, b);
    println!("-------------------");

    let mut i = 0;
    println!("i = {}",i);
    let a = divide3(10,i);
     //i++; 注意在rust中没有++这个运算符
    i +=1;
    println!("i = {}",i);
    let b  =divide3(10,i);
    println!("a = {:?},b = {:?}",a,b);

  
}

//但其实, 以上的Res的功能，标准库已经帮我们实现了
// pub enum Result<T, E> {
//     Ok(T),
//     Err(E),
// }
//来看一下用标准库的Result如何实现提上divide函数：
fn divide2(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        // return Result::Err("Cannot divide by zero".to_string());
        return Err("Cannot divide by zero".to_string()); //可简写，因为Ok和Err也是标准库的一部分，就行print!系列的宏一样。
    }
    // Result::Ok(a / b)
    Ok(a / b)
}
//同时，标准库还有一个Option枚举，它返回一个选项，要么又要么没有。
// pub enum Option<T> {
//     Some(T),
//     None,
// }

fn divide3(a: i32, b: i32) -> Option<i32> {
    if b == 0 {
        return None; //简写，因为None和Some也是标准库的一部分，就行print!系列的宏一样。
    }
    Some(a/b)
}

/*
/demo# cargo run
   Compiling demo v0.1.0 (/root/workspace/study/rust-study/demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/demo`
val= 2
-------------------
val = 5
a = Ok(5) ,b =Err("Cannot divide by zero")
-------------------
i = 0
i = 1
a = None,b = Some(10)
*/
```

### rust中的循环机制

```rust
//rust中的迭代器

fn main() {
    //rust 中的循环
    let mut i = 0;
    loop{
        i+=1;
        if i>10{
            break;
        }
        println!("loop i = {} item",i);
    }
    i=0;
    while i<10{
        i+=1;
        println!("while i = {} item",i);
    }
    //在rust中for的对象必须是一个迭代器
    for i in 1..10 { //左包含，右不包含  1..=10 左右包含
        println!("for loop {}",i);
    }
}


/*
root@yhpro:~/workspace/study/rust-study/demo# cargo run
   Compiling demo v0.1.0 (/root/workspace/study/rust-study/demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29s
     Running `target/debug/demo`
loop i = 1 item
loop i = 2 item
loop i = 3 item
loop i = 4 item
loop i = 5 item
loop i = 6 item
loop i = 7 item
loop i = 8 item
loop i = 9 item
loop i = 10 item
while i = 1 item
while i = 2 item
while i = 3 item
while i = 4 item
while i = 5 item
while i = 6 item
while i = 7 item
while i = 8 item
while i = 9 item
while i = 10 item
for loop 1
for loop 2
for loop 3
for loop 4
for loop 5
for loop 6
for loop 7
for loop 8
for loop 9
root@yhpro:~/workspace/study/rust-study/demo# 

*/
```

> 运算符 .. 是rust的语法糖，如1..10 表示1到9 的数字。在rust中，for循环遍历的对象必须是一个迭代器对象。

```rust
//创建一个可迭代的对象
pub struct Stepper {
    curr: i32, //当前的位置
    step: i32, //前进的步数
    max: i32,  //所能到的最大位置
}

//实现rust中的迭代器的接口
impl Iterator for Stepper {
    type Item = i32;
    fn next(&mut self) -> Option<Self::Item> {
        if self.curr >= self.max {
            return None;
        }
        let res = self.curr;
        self.curr += self.step;
        Some(res) //return Some(res);
    }
}
fn main() {
    let mut st = Stepper{
        curr:0,
        step:2,
        max:10,
    };
    loop{
        match st.next() {
            Some(v) =>println!("loop {} item",v),
            None =>break,
        }
    }
    let mut st2 = Stepper{
        curr:0,
        step:3,
        max:10,
    };
    while let Some(n) =st2.next(){
        println!("while n = {} item",n);
    }
    let  st3 = Stepper{
        curr:0,
        step:5,
        max:50,
    };
    for i in st3{
        println!("for i = {} item",i);
    }
}


/*
root@yhpro:~/workspace/study/rust-study/demo# cargo run
   Compiling demo v0.1.0 (/root/workspace/study/rust-study/demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22s
     Running `target/debug/demo`
loop 0 item
loop 2 item
loop 4 item
loop 6 item
loop 8 item
while n = 0 item
while n = 3 item
while n = 6 item
while n = 9 item
for i = 0 item
for i = 5 item
for i = 10 item
for i = 15 item
for i = 20 item
for i = 25 item
for i = 30 item
for i = 35 item
for i = 40 item
for i = 45 item
root@yhpro:~/workspace/study/rust-study/demo# 
*/
```

### rust中的数据拷贝

> rust中基本数据类型都默认实现了Colne这个taint，当我们自己定义一个类型时，一旦把它赋值给另一个值时就发生了所有权的转移了，原来的变量就不能用了，当我们想要将一个自己自定义的数据类型赋值给另一个变量，而希望原来的变量也能使用时，我们就必须实现Colne这个taint。

```rust
#[derive(Debug,Clone)] 
pub struct Person{
    name:String,
    age:i32,
}
fn main(){
    let x = 5;
    let y = x;
    println!("x= {},y= {}",x,y);

    let p1 = Person{
        name:"alex".to_string(),
        age:22,
    };
    // let p1 = p2; //这里所有权发生转移，不能在使用p了
    //println!("p1 = {:?},p2 ={:?}",p1,p2) 
    //当为其实现Colne这个taint之后
    let mut p2 = p1.clone(); //深拷贝
    p2.name.push_str("2");//push_str() String类型自带的方法
    println!("p1 = {:?},p2 ={:?}",p1,p2); //p1 = Person { name: "alex", age: 22 },p2 =Person { name: "alex2", age: 22 }
    //要实现跟基本变量一样，我们还需要实现Copy这个taint,在实现Copy这个taint之前，我们必须实现Clone这个taint
    let pn1 = Point::new(1,2);
    let mut pn2 = pn1;
    pn2.x +=10;//Copy这个taint也是深拷贝，废话事先实现了Colne这个taint的
    println!("pn1={:?},pn2={:?}",pn1,pn2);//pn1=Point { x: 1, y: 2 },pn2=Point { x: 11, y: 2 }

}
#[derive(Debug,Clone,Copy)]
pub struct Point{
    x:i32,
    y:i32,
}
impl Point{
    fn new(_x:i32,y:i32)-> Self{
        Point{
            x:_x,
            y, //倘若参数名于变量名一致，可省略写
        }
    }
}
```

### rust冒泡排序

```rust
//冒泡排序
pub fn bubble_sort<T:PartialOrd+std::fmt::Debug>(arr:&mut [T]){ //PartialOrd 是一个比较运算符
    let mut  count=0;
    let mut swap= 0;
    for i in 0..arr.len(){
        //算法优化
        let mut ok = true;
        for j in 0..(arr.len()-1)-i{
            //统计一下比较的次数
            count +=1;
            println!("arr={:?}",arr);
            if arr[j]>arr[j+1]{
                //统计一下交换的次数
                arr.swap(j,j+1);
                swap+=1;
                ok = false;
            }
        }

        if ok{
            println!("arr.len() = {},count={},swap={}",arr.len(),count,swap);
            return;
        }
    }
    println!("arr.len() = {},count={},swap={}",arr.len(),count,swap);
}

#[cfg(test)]
mod tests{
    use super::*;
    #[test]
    fn it_works(){
        let mut arr = vec![1,13,3,4,85,12,5];
        bubble_sort(&mut arr);
        assert_eq!(arr,vec![1,3,4,5,12,13,85])
        //arr.len() = 7,count=21,swap=7
        //arr.len() = 7,count=18,swap=7  很明显，优化了之后，效率高了不少
    }
}

```

### rust归并排序
