# Rust 猜数游戏

```go
use std::io;
use rand::Rng;
use std::cmp::Ordering;
fn main(){
    println!("猜一个数");
    /*
        我希望我的学生不只是一个会敲代码的工具人，而是一个有血有肉有创造力有思想的人。
     */
    /*
    Cargo的另一个巧妙特性是运行cargo doc --open命令将在本地构建所有依赖项提供的文档，并在浏览器中打开它。例如，如果您对rand板条箱中的其他功能感兴趣，可以运行cargo doc --open并单击左侧工具栏中的rand。
     */
    let secret_number=rand::thread_rng().gen_range(1..=100);
    println!("随机生成的数字是 secret_number = {:?}",secret_number);
    println!("请输入你的猜测:");
  
   
      loop{
        let mut guess = String::new();
     /*
     您现在知道let mut guess将引入一个名为guess的可变变量。 = 告诉Rust我们现在要绑定一些东西到变量。等号右边是guess所绑定的值，这是调用String:new的结果，该函数返回字符串的新实例。String是标准库提供的一种字符串类型，是一种可增长的UTF-8编码的文本位。
    new行中的::语法指示new是string类型的关联函数。关联函数是在类型上实现的函数，在本例中是String。这个新函数创建一个新的空字符串。你会在许多类型上发现一个新的函数，因为它是一个函数的通用名称，它产生了某种新的值。
    总之,let mut guess=String::new 0;行创建了一个可变变量，该变量当前绑定到String的一个新的空实例
      */

     io::stdin()
        .read_line(& mut guess) //读取一行输入，并将其追加到指定的缓存区
        .expect("failed to read line");

    /* 
    如果我们没有在程序开始时使用std::io;导入io库，我们仍然可以通过编写std::io:stdin函数调用来使用这个函数。
    stdin函数返回std:io::Stdin的一个实例，它是一个代表终端标准输入句柄的类型。

接下来行read_line(&mut guess)调用标准输入句柄上的read_line方法，从用户那里获取输入。我们还将&mut guess作为参数传递给read_line，告诉它在哪个字符串中存储用户输入。read_line的全部工作是将用户输入到标准输入中的任何内容附加到字符串中(不覆盖其内容)，因此我们将该字符串作为参数传递。字符串参数需要是可变的，这样方法就可以改变字符串的内容。

&表示这个参数是一个引用，它为您提供了一种方法，让代码的多个部分访一段数据，而不需要将该数据多次复制到内存中。引用是一个复杂的功能，Rust的主要优点之一就是使用引用是多么的安全和容易。你不需要知道很多这些细节来完成这个程序。现在，您只需要知道，就像变量一样，引用在默认情况下是不可变的。因此，您需要编写&mut guess而不是&guess，以使其可变。
    */

    /*
    以结果处理潜在故障
我们还在研究这行代码。我们现在讨论的是第三行文本，但请注意，它仍然是单一逻辑代码行的一部分接下来的部分就是这个方法
.expect("failed to read line")
我们可以把这段代码写成:
io::stdin().read_line(&mut guess).expect("Failed to read line");
然而，一条长长的线很难读，所以最好把它分开。在使用.method_name()语法调用方法时，引入换行符和其他空格通常是明智的。现在我们来讨论一下这条线的作用。
如前所述，read_line将用户输入的任何内容放入我们传递给它的字符串中，但它也返回一个Result值。result是一个枚举，通常称为enum，它是一种可以处于多种可能状态之一的类型。我们把每一个可能的状态称为一个变体。
第六章将更详细地介绍枚举。这些Result类型的目的是编码错误处理信息。
结果的变体是Ok and Err。Ok变量表示操作成功，Ok中是成功生成的值。Err变量表示操作失败，Err包含有关操作失败的方式或原因的信息。
Result类型的值和任何类型的值一样，都有在其上定义的方法。Result实例有一个可以调用的expect方法。如果此Result实例是Err值，expect将导致程序崩溃，并显示您作为参数传递给expect的消息。如果read_line方法返回Err，那么它很可能是来自底层操作系统的错误的结果如果Result的这个实例是一个Ok值，expect将接受Ok保存的返回值，并将该值返回给您，以便您可以使用它。在这种情况下。该值是用户输入的字节数。
     */

        //let guess :u32 = guess.trim().parse().expect("请输入一个数字");
    //为了进一步完善游戏的行为，我们让游戏忽略非数字，这样用户就可以继续猜测，而不是在用户输入非数字时使程序崩溃。
    let guess :u32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) =>continue, //下划线是一个catchall值;在这个例子中，我们希望匹配所有的Err值，不管它们里面有什么信息。因此，程序将执行第二个arm的代码continue，它告诉程序转到循环的下一个迭代，并请求另一个猜测。因此，实际上，程序忽略了解析可能遇到的所有错误!
  
    };
    println!("you guessed: {guess} ");


    /*
        首先，我们添加另一个use语句，将一个名为std;:cmp::Ordering的类型从标准库引入范围。Ordering类型是另一个枚举类型，具有变体Less、Greater和Equal。这是比较两个值时可能出现的三种结果。
然后我们在底部添加五行使用排序类型的新行。cmp方法比较两个值，可以对任何可以比较的值调用该方法。它引用任何你想要比较的东西:这里是比较guess和secret number。然后它返回排序枚举的一个变体，我们使用use语句将其带入作用域。我们使用一个匹配表达式，根据调用cmp时返回的顺序变量guess和secret_number中的值来决定下一步该做什么。
match匹配表达式由arms组成。一个arm由一个要匹配的模式和一段代码组成，如果要匹配的值符合该arm的模式则应该运行该代码。Rust接受给定的匹配值，并依次查看每个手臂的图案。模式和match结构是强大的Rust特性:它们允许您表达代码可能遇到的各种情况，并确保您能够处理所有这些情况。些特性将分别在第6章和第18章中详细介绍。
让我们用这里使用的match表达式来浏览一个例子。假设用户已经猜出了50和这次随机生成的密码是38。
当代码比较50和38时cmp方法将返回Ordering;:Greater,因为50大于38。match表达式获得排序::更大的值，并开始检查每个臂的模式。它查看第一个臂的模式ordering::Less，发现orderingGreater的值与Ordering;:Less不匹配，因此它忽略该警中的代码并移动到下一个臂。下一个手臂的模式是Ordering:Greater，它与Ordering;:Greater匹配!该警中的相关代码将执行并打印太大!到屏幕上。匹配表达式在第一次成功匹配后结束，因此它不会查看本场景中的最后一个臂
     */

        //loop关键字创建一个无限循环。我们将添加一个循环，让用户有更多的机会猜测数字:
        println!("请输入一个数字.");
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("too small"),
            Ordering::Greater => println!("too greater"),
            Ordering::Equal => {
                println!("you win!");
                break;
            }
        }
      }
}
```
