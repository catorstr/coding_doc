# Rust 结构体/函数/方法/枚举

### 函数

在 Rust 代码中函数随处可见，例如我们使用的 main 函数，关于函数的几个特点总结如下：

* 使用 fn 关键字声明。
* 函数参数必须定义类型。
* 箭头 `->` 后声明返回类型，默认情况下返回最后一个表达式，注意不要有分号 `;`
* 也可使用 return 返回，这里要加分号 `;`。

```go
fn main() {
    let res1 = multiply(2, 3);
    let res2 = add(2, 3);
    print!("multiply {}, add {} \n", res1, res2);
}
fn multiply(x: i32, y: i32) -> i32 {
    x * y
}
fn add(x: i32, y: i32) -> i32 {
    return x + y;
}
```

### 结构体

结构体是一种自定义数据类型，它由一系列属性组成（这个属性拥有自己的属性和值），结构体是数据的集合，就像面向对象编程语言中一个无方法的轻量级类，因为 Rust 本身不是一门面向对象的语言，合理的使用结构体可以使我们的程序更加的结构化。

#### 定义一个结构体

使用 struct 关键字定义一个结构体，创建一个结构体实例也很简单，如下例所示：

```rust
struct User {
    username: String,
    age: i32
}
fn main() {
    // 创建结构体实例 user1
    let user1 = User {
        username: String::from("五月君"),
        age: 18
    };
    print!("我是: {}, 永远 {}\n", user1.username, user1.age); // 我是: 五月君, 永远 18
}

```

### 方法

方法与函数类似，使用 fn 关键字声明，拥有参数和返回值，不同的是 **方法在结构体的上下文中定义，方法的第一个参数始终为 `self` 表示调用该方法的结构体实例** 。

改写上面的结构体示例，在结构体 User 上定义一个方法 info 打印信息，这里用到一个关键字 `impl`，它是 implementation 的缩写。

```rust
fn main() {
    //定义结构体实例
    let user1 = User{
        username :String::from("lhc"),
        age:23,
    };
    //使用结构体的方法
    user1.info()
    /*我是lhc,年龄23 */
}

struct User {
    username :String, //定义结构以字段需要,隔开
    age :i32
}

//为结构体定义方法
impl User{
    fn info(self){
        print!("我是{},年龄{}",self.username,self.age);
    }
}
```

### 枚举
