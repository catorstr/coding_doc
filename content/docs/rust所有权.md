# Rust所有权

> 所有权是Rust最独特的特征，对语言的其他部分有着深刻的影响。它使Rust能够在不需要垃圾收集器的情况下保证内存安全，因此了解所有权是如何工作的非常重要。在本章中，我们将讨论所有权以及几个相关特性:借用、切片以及Rust如何在内存中布局数据。

> 所有权规则
> 首先，让我们来看看所有权规则。在我们学习说明这些规则的例子时，请记住这些规则:
>
> 1. Rust中的每个值都有一个所有者
> 2. 一次只能有一个所有者。
> 3. 当所有者超出作用域时，该值将被删除。

### 变量的作用域

> 作用域之间的关系以及变量何时有效与其他编程语言类似。

```rust
{//s在这里是无效的，它还没有被声明
 let s = "hello word";//s是从现在开始有效的。
 /*
变量s引用一个字符串字面值，其中字符串的值被硬编码到程序的文本中。变量从声明它的点到当前作用域的结束都是有效的。
*/
 //todo 在这里对s进行一些其他操作
} //此作用域现在已经结束，s不再有效

```

### String字符串类型

为了说明所有权的规则，我们需要一个比我们在第3章“数据类型”一节中讨论的数据类型更复杂的数据类型。前面介绍的类型具有已知的大小，可以存储在栈中，当它们的作用域超过时，可以从栈中弹出，如果代码的另一部分需要在不同的作用域中使用相同的值，可以快速轻松地复制到一个新的独立实例中。但是我们想看看存储在堆上的数据，并探索Rust如何知道什么时候清理这些数据，String类型就是一个很好的例子。
我们将集中讨论字符串中与所有权有关的部分。这些方面也适用于其他复杂的数据类型，无论它们是标准库提供的还是您创建的。我们将在第8章更深入地讨论字符串
我们已经看到了字符串字面值，其中字符串值硬编码到我们的程序中。字符串字面量是方便的，但它们并不适用于我们可能需要使用文本的每一种情况。一个原因是它们是不可变的。另一个问题是，当我们编写代码时，并不是每个字符串值都是已知的:例如，如果我们想获取用户输入并存储它，该怎么办?对于这些情况，Rust有第二个字符串类型String。这种类型管理在堆上分配的数据，因此能够存储大量在编译时未知的文本。您可以使用from函数从字符串字面量创建String，如下所示

```rust
let s = String::from("hello word");//从字符串字面量创建String
```

双冒号::操作符允许我们将这个特定的from函数命名为String类型，而不是使用string from之类的名称。我们将在第5章的“方法语法”一节和第7章的“引用模块树中的项的路径”中讨论模块的命名空间时进一步讨论这个语法。这类字符串可以发生改变。

对于String类型，为了支持可变的、可增长的文本块，我们需要在堆上分配一定量的内存(在编译时未知)来保存内容。这意味着:

1. 必须在运行时从内存分配器请求内存。
2. 当我们处理完String时，我们需要一种将内存返回给分配器的方法

第一部分由我们完成:当我们调用String::From时，它的实现请求它所需的内存。这在编程语言中几乎是通用的。
然而，第二部分是不同的。在带有垃圾收集器(GC)的语言中，GC跟踪并清理不再使用的内存，我们不需要考虑它。在大多数没有GC的语言中，我们的责任是识别何时不再使用内存，并调用代码显式释放它，就像我们请求它一样。历史上，正确地做到这一点一直是一个困难的编程问题。如果我们忘记了，我们就会浪费记忆。如果我们做得太早，就会有一个无效的变量。如果我们做了两次，那也是一个bug。我们需要将正好一个分配与正好一个空闲配对。Rust作为新兴的编程语言很好的为我们解决了这个问题，我们这需要知道：

1. 如何去申请一个堆的内存
2. 及我们申请的内存何时有效，它的作用范围。

当一个变量超出作用域时，Rust会为我们调用一个特殊的函数。这个函数被称为drop的函数，这个函数会为我们释放申请在堆上的内存空间。

### 关于所有权需要注意的事情

> 所有权的问题，需要我们在额外的关注，不然在开发中可能会出现一些意想不到的事情。

来看看这个问题：

```rust
fn main(){

  let x =5;
  let y =x;
  println!("x={x},y={y}"); //x=5,y=5
  //目前看来还没有什么问题,继续
  let s1 = String::from("hello");//s2=hello
  let s2 = s1;
  //println!("s1={s1},s2={s2}"); //这就报错，编译不了了
  println!("s2={s2}");
  //是不是很奇怪！？
  //这就和所有权息息相关了

  //对上面的代码做进一步的解释
  /* 
    我们刚开始将一个5绑定到变量x中，然后又将x的值绑定到变量y中，同时打印x和y的值
    正确输出了x=5，y=5
    这是因为，我们定义的x，y它们是整数，是具有已知的、固定大小的简单值，这两个5值被压入栈中。
    但当我们定义一个String类型时，它是在堆上开辟了一块内存空间，当我们把s1赋值给s2时，发生了这样的事情：https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
    当我们将s1赋给S2时(发生了浅拷贝)，字符串数据被复制，这意味着我们只复制了栈上的指针、长度和容量。我们不复制指针所指的堆上的数据。
    前面我们说过，当一个变量超出作用域时，Rust会自动调用drop函数并清理该变量的堆内存。但图4-2显示了两个指向同一位置的数据指针。这是一个问题，当s2和s1超出作用域时，它们都会尝试释放相同的内存。这被称为双自由错误，是我们前面提到的内存安全漏洞之一。两次释放内存可能会导致内存损坏，这可能会导致安全漏洞

    为了确保内存安全，在行lets2=s1;之后，Rust认为s1不再有效。因此，当s1超出范围时，Rust不需要释放任何东西。
  
  */
  //如果恶意继续使用s1那么就会发生这样的错误： value borrowed here after move(在rust中称为move错误)
 // println!("s1={s1},s2={s2}")

 /* 
  此外，这里还隐含了一个设计选择:
  Rust永远不会自动创建数据的“深层”副本。
  因此，就运行时性能而言，任何自动复制都可以认为是廉价的。

    如果我们想深度复制String的堆数据，而不仅仅是堆栈数据，我们可以使用一种称为clone的常用方法。我们将在第5章讨论方法语法。但是因为方法是许多编程语言的共同特性，所以你可能以前见过它们。下面是一个实际应用中的克隆方法的例子:
 */
    let s1= String::from("test clone");
    let s2  =s1.clone(); //可以理解为深拷贝，clone帮我们在堆上分配了一个和s1内容一样的内存空间
    println!("s1={s1},s2={s2}"); //s1=test clone,s2=test clone

    /*
    但是这段代码似乎与我们刚刚学到的内容相矛盾:我们没有调用clone，但是x仍然有效，没有移到y中。
原因是，在编译时大小已知的类型(如整数)完全存储在堆栈中，因此可以快速复制实际值。这意味着我们没有理由在创建变量y后，阻止x有效。换句话说，这里的深拷贝和浅拷贝没有区别。所以调用clone不会做任何与通常的浅层复制不同的事情，我们可以省略它。

Rust有一个称为Copy trait的特殊注释，我们可以将它放在存储在栈中的类型上，比如整数(我们将在第10章中详细讨论trait)。如果一个类型实现了Copy trait，使用它的变量不会move移动，而是被简单地复制，使它们在赋值给另一个变量后仍然有效。
     */

    /*
    那么，什么类型实现了Copy trait呢?您可以检查给定类型的文档以确定，但作为一般规则，任何简单标量值组都可以实现复制，并且任何需要分配的内容或某种形式的资源都不能实现复制。下面是一些实现复制的类型:
    所有整数类型，如u32。
    布尔类型bool，值为true和false。
    所有的浮点类型，比如f64。
    字符类型为char。
    组，如果它们只包含也实现复制的类型。例如， i32，i32) 实现了复制，但 (i32，String)没有。
    */
}
```

### 所有权和函数

> 将值传递给函数的机制类似于将值赋给变量的机制。传递一个变量给一个函数会移动或复制，就像赋值一样。
>
> ```rust
> fn main() {
>     let s = String::from("hello");  //s 进入作用域 s comes into scope 
>
>     takes_ownership(s);             //s的所有权权被移到到了函数中， s's value moves into the function...
>                                     //之后s将不再有效 ... and so is no longer valid here
>
>     let x = 5;                      //x进入作用域 x comes into scope
>
>     makes_copy(x);                  //x的所有权被移动到了函数中 x would move into the function,
>                                     //但x的数据类型是i32 是一个Copy类型，所以x仍然有效 but i32 is Copy, so it's okay to still
>                                     //之后x还可以继续使用 use x afterward
>
> } // 这里，x，s退出它的作用域，但因为s的所有权在前边已经发生了移动，所以没有任何变化。Here, x goes out of scope, then s. But because s's value was moved, nothing
>   // special happens.
>
> fn takes_ownership(some_string: String) { // some_string comes into scope
>     println!("{}", some_string);
> } // Here, some_string goes out of scope and `drop` is called. The backing
>   // memory is freed.
>
> fn makes_copy(some_integer: i32) { // some_integer comes into scope
>     println!("{}", some_integer);
> } // Here, some_integer goes out of scope. Nothing special happens.
> //如果我们试图在调用到取得所有权后使用5。Rust会抛出一个编译时错误。这些静态检查可以保护我们不犯错误。
> ```

### 函数的返回值与所有权

> 返回值也可以转移所有权。
>
> ```rust
> fn main() {
>     let s1 = gives_ownership();         // gives_ownership moves its return
>                                         // value into s1
>
>     let s2 = String::from("hello");     // s2 comes into scope
>
>     let s3 = takes_and_gives_back(s2);  // s2 is moved into
>                                         // takes_and_gives_back, which also
>                                         // moves its return value into s3
> } // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
>   // happens. s1 goes out of scope and is dropped.
>
> fn gives_ownership() -> String {             // gives_ownership will move its
>                                              // return value into the function
>                                              // that calls it
>
>     let some_string = String::from("yours"); // some_string comes into scope
>
>     some_string                              // some_string is returned and
>                                              // moves out to the calling
>                                              // function
> }
>
> // This function takes a String and returns one
> fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
>                                                       // scope
>
>     a_string  // a_string is returned and moves out to the calling function
> }
> ```

### 引用和借用Reference and Borrowing

>  变量的所有权每次都遵循相同的模式:将一个值赋给另一个变量会移动它。当包含堆上数据的变量超出作用域时，除非数据的所有权已移动到另一个变量，否则将通过删除来清除该值。

> 虽然这样做是可行的，但是获取所有权，然后返回每个函数的所有权是有点乏味的。如果我们想让一个函数使用一个值，而不使用它的所有权，该怎么办?非常恼人的是，如果我们想要再次使用它，那么传入的任何内容都需要被传回，另外还有我们可能希望返回的函数体产生的任何数据。Rust允许我们使用元组返回多个值：
>
> ```rust
> fn main() {
>     let s1 = String::from("hello");
>
>     let (s2, len) = calculate_length(s1);
>
>     println!("The length of '{}' is {}.", s2, len);
> }
>
> fn calculate_length(s: String) -> (String, usize) { //返回值一是为了s的所有权，但在实际开发中为了归还所有权而增加一个返回参数不是很明智的选择。
>     let length = s.len(); // len() returns the length of a String
>
>     (s, length)
> }
> ```
>
> 幸运的是，Rust有一个在不转移所有权的情况下使用值的特性，称为引用。
>
> 我们可以提供对字符串值的引用,引用类似于指针，因为它是一个地址，我们可以通过它访问存储在该地址中的数据，该数据属于其他一些变量。与指针不同，引用保证在引用的生命周期内指向特定类型的有效值。
>
> 引用借用的行为，如在现时生活中你向某人借一本书，看完了就得必须还回去，但是你并不拥有它。而所有权，一旦获得了，就是属于自己的了，还不还回去得看自己的心情。
>
> 下面是如何定义和使用calculate_length函数，该函数将对象的引用作为参数，而不是获取值的所有权:
>
> ```rust
>
> fn main(){
>   let s1 =String::from("hello");
>   let len = calculate_length(&s1); //注意我们将&s1传递到calculateength中，在它的定义中，我们采用&String而不是string。这些&符号表示引用，它们允许您引用某个值而不拥有它的所有权。
>   println!("s1 len ={len}");
>   //引用的方式s1的所有权不会move转移
>   println!("s1={s1}");
>   //注:与使用&引用相反的是解引用，这是通过解引用运算符*完成的。
> }
>
> fn calculate_length(s :&String)->usize{ //函数参数引用的定义方式，引用不需要归还所有权
>   s.len() //rust的语法特色，也是需要注意的地方，注意这样写的话不要有分号
> }
> ```
>
> ```rust
> fn main(){
>   let s = String::from("hello");
>   change(&s)
> }
> fn change(s :&String){
>  s.push_str(",word"); //Error 当我们试图改变引用的东西时就会报错。
> }
> ```
>
> 正如变量在默认情况下是不可变的一样，引用也是不可变的。Rust不允许修改我们引用的东西。

#### 可变的引用

> 如果我们想要修改一个借来的值，就可以使用一个可变的引用，只需要做一些小的调整：
>
> ```rust
> fn main(){
>   let mut s = String::from("hello");
>   change(&mut s);
>   println!("s ={s}"); //s =hello,word //这样我们旧修改了我们引用的东西了
> }
>
> fn change(s :&mut String){
>  s.push_str(",word");
> }
> ```
>
> 可变引用有一个很大的限制:如果你有一个值的可变引用，你就不能有这个值的其他引用。尝试创建两个对s的可变引用的代码将失败:
>
> ```rust
> fn main(){
>   let mut s = String::from("hello");
>
>   let r1 =&mut s; //r1 对s进行了一个可变的引用
>  // let r2 = &mut s; //r2 又对s进行了可变引用的操作，这里旧报错了
>   println!("r1 ={r1}");
>  // println!("r2 = {r2}");
>  /*
>  有这个限制的好处是，RUST可以在编译时防止数据竞争。数据竞争类似于竞争条件，当发生这三种行为时会发生:
>     1. 两个或多个指针同时访问相同的数据
>     2. 至少有一个指针正在用于写入数据
>     3. 没有机制用于同步对数据的访问。
> 数据竞争会导致未定义的行为，当您试图在运行时跟踪它们时，可能很难诊断和修复;Rust通过拒绝编译带有数据竞争的代码来防止这个问题!
> 和往常一样，我们可以使用花括号来创建一个新的作用域，允许多个可变的引用，只是不能同时引用:
>   */
>   let mut s = String::from("hello");
>   let r1 =&mut s;
>   {
>     let r2 = &mut s; //这不会报错，r2的作用域和r1不同
>   }
>
>   //不可变的引用不存在这样的问题
>   let mut s = String::from("hello");
>
>     let r1 = &s; // no problem
>     let r2 = &s; // no problem
>    // let r3 = &mut s; // BIG PROBLEM 但是同时对s进行可变的引用和不可变的引用时会报错。
>     {
>       let r3 = &mut s; //当作用域不同时是可以的
>     }
>     //println!("{}, {}, and {}", r1, r2, r3);
>
>     //请注意，引用的作用域从引入它的位置开始，一直持续到最后一次使用该引用。例如，这段代码可以编译，因为不可变引用println!的最后一次使用发生在引入可变引用之前:
>       let mut s = String::from("hello");
>
>     let r1 = &s; // no problem
>     let r2 = &s; // no problem
>     println!("{} and {}", r1, r2);
>     // variables r1 and r2 will not be used after this point
>
>     let r3 = &mut s; // no problem
>     println!("{}", r3);
>     /* 
>     不可变引用ri和r2的作用域在println!最后一次使用它们的位置，也就是在创建可变引用r3之前。这两个作用域不重叠，所以这段代码是允许的:编译器可以知道在作用域结束之前的某个点上引用不再被使用。
>     */
> }
> ```
