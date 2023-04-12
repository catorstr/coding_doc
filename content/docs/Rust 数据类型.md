# 基本数据类型

Rust 是一个静态数据类型的语言，这意味着在编译时它就要知道变量的类型。

Rust 包含四种基本数据类型分别为：整形、浮点型、布尔型、字符型。

#### 整型

Rust 里的整型又分为带符号的整型（signed）和非带符号整型（unsigned），即有符号整型与无符号整型，两者之间的区别是数字是否是负数。带符号整型的安全存储范围为-(2^(n-1)n)到2^(n-1)-1，n 就是下面的长度。

非带符号整型的安全存储范围为 0 到2^(n-1)-1。isize 和 usize 是根据系统架构决定的，例如带符号整型，如果系统是 64 位，类型为 i64，如果系统是 32 位，类型为 i32。

| 长度     | 带符号整型 | 非带符号整型 |
| -------- | ---------- | ------------ |
| 8-bit    | i8         | u8           |
| 16-bit   | i16        | u16          |
| 32-bit   | i32        | u32          |
| 64-bit   | i64        | u64          |
| 128-bit  | i128       | u128         |
| 系统架构 | isize      | usize        |

```rust
fn main() {
    let num = 1;//声明了一个不可变的变量，并将它赋值为1，由于在vscode安装了rust插件，语法被自动补全了(自动类型推断)，我写的其实是：let num =1；
    //num =2 //这里报错了，说明我们用let 声明的num是不可变的变量，只能对它进行一次赋值，
    /* 
变量
Rust 使用 let 声明一个变量，通常来说变量将是可变的，但是在 Rust 中默认设置的变量是预设不可变的，这也是 Rust 推动你能充分利用其提供的安全性来写程序的方式之一，Rust 中鼓励你多多使用不可变的，当然如果你明确知道该变量是可变得，也是可以的。
rust和go一样声明的变量必须使用，不然会报错，这也许是新型语言的一个特点吧
    */
    println!("不可变的变量num = {}",num); //注意，println! 后面加上了符号 ! 并不是一个函数，而是一个宏。{}可以理解为一个格式化方式
    let mut num2 = 2; //在变量名称前加上 mut 关键字，表明该变量是可变的。
    num2 = 3; //这时候对变量进行赋值，就不会报错了
    println!("可变的变量num2 = {}",num2);
/*
常量
常量使用 const 声明，之后是不可变的，在声明时必须指定变量类型，这也是与 let 的不同，还需注意的是常量名称一定要大写，否则编译阶段也是会报错的。
 */
    const NUM:i8 = 1;
    println!("常量NUM:={}",NUM);
/*
作用域
一个变量只有在其作用域内是生效的。下例，变量 y 在花括号内即它的块级作用域内是有效的，当离开花括号如果想在外部打印，会报 cannot find value y in this scope 错误。
 */

    let x = 1;
    {
        let y = 2; // y 在此处开始有效
        println!("y {}", y);
    } // 此作用域结束，y 不再有效
   // println!("x {} y {}", x, y);//报错
    println!("x {}", x);

}


```


#### 浮点型

Rust 的浮点型提供了两种数据类型 f32、f64，分别表示为 32 位与 64 位，默认情况下是 64 位。

```go
fn main() {
    let x = 2.0; // f64
    let y: f32 = 3.0; // f32
    println!("x: {}, y: {}", x, y); // x: 2, y: 3
}
```

#### 布尔型

和大多数编程语言一样，Rust 中的布尔型包含两个值：true 和 false。

```go
fn main() {
    let x = true; // bool
    let y: bool = false; // bool
}
```

#### 字符型

Rust 中的字符型为一个 Unicode 码，大小为 4 bytes，char 类型使用单引号包括。

```go

fn main() {
    let x = 'x';
    let y = '????';
    println!("x: {}, y: {}", x, y); // x: x, y: ????
}
```

### 复合类型

复合类型可以组合多个数值为一个类别，复合类型包含两种：元组（tuples）和数组（arrays）。


#### 元组

元组是将多个不同数值组合为一个复合类型的常见方法，元组拥有固定长度，一旦声明无法更改。

我们通过**解构**的方式，分别从声明的元组中取出数据，如下例所示：

```go
fn main() {
    let tup: (i32, f64, char) = (1, 1.01, '????');
    let (x, y, z) = tup;
    println!("x: {}, y: {}, z: {}", x, y, z); // x: 1, y: 1.01, z: ????
}
```

除此之外我们还可通过数值的索引来访问元组中的数据。

```go
fn main() {
    let tup: (i32, f64, char) = (1, 1.01, '????');
    let x = tup.0;
    let y = tup.1;
    let z = tup.2;
    println!("x: {}, y: {}, z: {}", x, y, z); // x: 1, y: 1.01, z: ????
}
```

#### 数组

与元组不同的是数组中的所有元素类型必须一致，Rust 中的 Array 与其它语言不太一样，因为其 Array 的长度是固定的和元组一样。

```go
fn main() {
    let a: [i32; 5] = [1, 2, 3, 4, 5];
    println!("a[0]: {}, a[4]: {}", a[0], a[1]); // a[0]: 1, a[4]: 2
}
```

```rust
fn main() {
//声明一个元组，并初始化它
let tup:(i32,char,f64) = (1,'H',1.25); //char 字符类型用''
//通过下标访问元组
println!("tup.0 = {},tup.1 ={},tup.2 ={}",tup.0,tup.1,tup.2);
let (i,c,f) =tup;//解构，即拆分元组的数据，然后分别赋值到变量i，c,f中
println!("i = {},c ={},f= {}",i,c,f);

//声明一个数据类型为i32，长度为5的数组，并赋值
let arr:[i32;5] = [1,2,3,4,5];
//通过下标访问数组
println!("arr[0]={},arr[4]={}",arr[0],arr[4]);
/*
tup.0 = 1,tup.1 =H,tup.2 =1.25
i = 1,c =H,f= 1.25
arr[0]=1,arr[4]=5
 */
}

```

### 扩展

```bash
#将源代码编译成可执行文件
root@DESKTOP-INJPABC:~/worksapce/dev/rustStudy/helo-rust/src# rustc main.rs
 #运行编译生成的可执行文件
root@DESKTOP-INJPABC:~/worksapce/dev/rustStudy/helo-rust/src# ./main 
tup.0 = 1,tup.1 =H,tup.2 =1.25
i = 1,c =H,f= 1.25
arr[0]=1,arr[4]=5
root@DESKTOP-INJPABC:~/worksapce/dev/rustStudy/helo-rust/src# 


```
