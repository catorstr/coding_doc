# Rust 学习之旅

## 安装Rust

> #https://www.rust-lang.org/zh-CN/learn/get-started
>
> 我使用的是windows的Linux子系统(WSL),所以：
>
> ```bash
> curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
> ```

Rust 的升级非常频繁。如果您安装 Rustup 后已有一段时间，那么很可能您的 Rust 版本已经过时了。运行 `rustup update` 获取最新版本的 Rust。

### Cargo：Rust 的构建工具和包管理器

您在安装 Rustup 时，也会安装 Rust 构建工具和包管理器的最新稳定版，即 Cargo。Cargo 可以做很多事情：

* `cargo build` 可以构建项目，会在 target/debug/ 目录下生成编译好的文件
* `cargo run` 可以运行项目
* `cargo test` 可以测试项目
* `cargo doc` 可以为项目构建文档
* `cargo publish` 可以将库发布到 [crates.io](https://crates.io/)。
* `cargo check`. 为项目进行语法检查

要检查您是否安装了 Rust 和 Cargo，可以在终端中运行：

`cargo --version`

## 创建新项目

我们将在新的 Rust 开发环境中编写一个小应用。首先用 Cargo 创建一个新项目。在您的终端中执行：

`cargo new hello-rust`

这会生成一个名为 `hello-rust` 的新目录，其中包含以下文件：

```
hello-rust
|- Cargo.toml
|- src
  |- main.rs
```

`Cargo.toml` 为 Rust 的清单文件。其中包含了项目的元数据和依赖库。

`src/main.rs` 为编写应用代码的地方。

---

`cargo new` 会生成一个新的“Hello, world!”项目！我们可以进入新创建的目录中，执行下面的命令来运行此程序：

`cargo run`

您应该会在终端中看到如下内容：

```
$ cargo run
   Compiling hello-rust v0.1.0 (/Users/ag_dubs/rust/hello-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 1.34s
     Running `target/debug/hello-rust`
Hello, world!
```

但是这里在真实运行时报错了：error: linker `cc` not found

解决方案：

```bash
sudo apt update
sudo apt install build-essential
```

`cargo run`

```bash
root@DESKTOP-INJPABC:~/worksapce/dev/rustStudy/helo-rust# cargo run
   Compiling helo-rust v0.1.0 (/root/worksapce/dev/rustStudy/helo-rust)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/helo-rust`
Hello, world!
root@DESKTOP-INJPABC:~/worksapce/dev/rustStudy/helo-rust#
```

在创建的hello-rust项目下用vscode打开，安装rust的vscode插件，方便后续开发：[rust-analyzer](vscode-file://vscode-app/e:/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html "command:extension.open?%5B%22rust-lang.rust-analyzer%22%5D")

## 扩展阅读

> cargo 是正式开发中使用的，它帮助我们管理我们的代码，让我们更方便的开发，但是对于一些简单的程序，我们可以使用rustc 来编译我们的代码。

我们可以手动创建一个目录，然后编写和运行一个简单的Rust程序：

```bash
mkdir ~/rustStudy
cd ~/riustStudy
mkdir hello_word
cd hello_word
vim main.rs #rust程序的源文件是以.rs为扩展名的，rust推荐的文件命名风格是：单词+下划线+单词。如hello_word.rs。

```

在我们的main.rs编写我们的rust代码：

```rust
fn main() {
    println!("hello word!");
}
```

hello word 程序是每一门编程语言首先要学的程序，rust也不例外。就这样我们rust语言的hello word程序已经编写完成了，接下来就是保存源文件，然后回到终端中编译我们的rust程序：使用rustc + 源文件 就可以编译成可执行文件了。

```bash
rustc main.rs
```

运行rust程序：

```bash
./main # or .\main.exe on Windows
```

注意：在windowns下会多出来一个 main.pdb的文件，它包含了程序的调试信息，包含调试信息的文件扩展名为pdb。在这里，您可以运行main或main.exe文件，它会将hello word 打印到终端中。

### rust的编译和运行是单独的步骤

Rust和go一样是静待类型的语言。如果您更熟悉动态语言，比如Ruby、Python或JavaScript，您可能不习惯将编译和运行程序作为单独的步骤。Rust是一种超前的编译语言，这意味着你可以编译一个程序，然后把可执行文件交给其他人，他们甚至可以在没有安装Rust的情况下运行它。如果您给某人一个rb、py或js文件，他们需要安装Ruby、Python或lavScript实现(分别安装)。但是在Rust语言中，你只需要一个命令来编译和运行你的程序。一切都是语言设计中的取舍。
使用rustc编译对于简单的程序来说是很好的，但是随着项目的增长，您将希望管理所有的选项并使共享代码变得容易。接下来，我们将向您介绍Cargo工具，它将帮助您编写真实世界的Rust程序。

### Cargo

Cargo是Rust的构建系统和软件包管理器。大多数Rustaceans使用这个工具来管理他们的Rust项目，因Cargo会为您处理很多任务，比如构建代码、下载代码所依赖的库以及构建这些库。 (我们称你的代码需要依赖的库。)
最简单的Rust程序，就像我们目前所写的一样，没有任何依赖关系。如果我们使用Cargo建立了“你好世界!”项目，它将只使用Cargo中处理构建代码的部分。当您编写更复杂的Rust程序时，您将添加依赖项，如果您使用Cargo启动一个项目，添加依赖项将容易得多。
因为绝大多数Rust项目都使用Cargo，所以本书的其余部分假设您也在使用Cargo。如果您使用“安装”节中讨论的官方安装程序，Cargo会随Rust一起安装。如果您通过其他方式安装了Rust，请在终端中输入以下内容检查是否安装了Cargo:

```bash
cargo --version
```

如果你看到一个版本号，你就有了!如果您看到错误，例如找不到命令，请查看您的安装方法的文档，以确定如何单独安装Cargo。

#### cargo 创建一个新的项目的细节

> 随便在一个你喜欢的目录下执行：
>
> ```bash
> cargo new hello_cargo
> cd hello_cargo
> ```
>
> 第一个命令创建一个名为hellocargo的新目录和项目。我们将项目命名为hello_cargo，Cargo将在同名目录中创建其文件。
> 进入hellocargo目录并列出文件。您将看到Cargo为我们生成了两个文件和一个目录:一个Cargo.toml的文件和一个src目录，其中包含一个main.rs文件。
> 它还初始化了一个新的Git仓库和一个gitignore文件。如果在现有的Git仓库中运行cargo new，则不会生成Git文件，你可以使用cargo new--vcs=git来覆盖这一行为。
>
> 这个文件在TOML中(*Tom’s*显而易见的，最小的语言)格式，这是Cargo的配置格式。
>
> ```toml
> [package]
> name = "hello_cargo"
> version = "0.1.0"
> edition = "2021"
>
> # See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
>
> [dependencies]
> ```
>
> 第一行[package]是一个小节标题，指出下面的语句正在配置一个包。当我们向该文件添加更多信息时，我们将添加其他部分。
> 接下来的三行设置Cargo编译程序所需的配置信息:要使用的Rust的名称、版本和版本密钥edition。我们将在附录E中讨论版本密钥。
> 最后一行[dependencies] 是一个部分的开始，用于列出项目的任何依赖项。在Rust中，代码包被称为板条箱。我们将不需要任何其他板条箱为这个项目，但我们会在第二章的第一个项目。所以我们将使用这个依赖项部分。
>
>> 现在打开src/main.rs，看一看
>>
>> ```rust
>> fn main() {
>>     println!("Hello, world!");
>> }
>> ```
>>
>> Cargo已经为您生成了一个“Helloworld!”程序，就像我们在清单1-1中编写的程序一样!到目前为止，我们的项目和Cargo生成的项目之间的区别在于，Cargo将代码放在src目录中，并且在我们的顶级目录中有一个Cargo.toml的配置文件。
>> Cargo希望您的源文件位于src目录中。顶层项目目录仅用于存放README文件、许可证信息、配置文件以及其他与代码无关的文件。帮助您组织项目。每件事都有自己的位置，每件事都在自己的位置。
>> 如果您启动了一个不使用Cargo的项目，就像我们使用“Hello，world!”项目，您可以将其转换为确实使用Cargo的项目。将项目代码移动到src目录中，并创建一个适当的Cargo.toml文件。
>>

#### 构建和运行一个Cargo项目

让我们来看使用cargo构建的程序有什么不同，在hello_cargo目录下使用cargo build来构建你的项目。这个命令在target/debug/hello_cargo(在Windows上是target/debug/hello_cargo.exe)中创建一个可执行文件，而不是在当前目录中。由于默认版本是调试版本，Cargo将二进制文件放在名为debug的目录中。
可以使用：

```bash
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

如果一切顺利，Hello, world! 应该打印到终端。第一次运行cargobuild也会导致Cargo在顶层创建一个新文件:Cargo.lock。此文件跟踪项目中依赖项的确切版本。这个项目没有依赖关系，所以文件有点稀疏。您永远不需要手动更改此文件;Cargo会为您管理其内容。

可以使用cargo run编译运行代码。
使用cargo run比必须记住运行cargobuild然后使用二进制文件的整个路径要方便得多，所以大多数开发人员使用cargo run.
注意，使用caogo run 时Cargo发现文件没有改变，所以它没有重建，只是运行了二进制文件。如果您修改了源代码，Cargo会在运行之前重新构建项目，您会看到以下输出:

```bash
 cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo还提供了一个名为cargo check 的命令。这个命令可以快速检查你的代码，以确保它可以编译，但不会产生一个可
执行文件:

```bash
 cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

通常，cargo check比cargo build快得多，因为它跳过了生成可执行文件的步骤。如果你在写代码的时候不断地检查你的工作，使用cargo check会加快让你知道你的项目是否还在编译的过程!因此，许多Rustaceans运行cargo check检查，因为他们写他们的程序，以确保它编译。然后当他们准备好使用可执行文件时运行货物构建
让我们回顾一下到目前为止我们学到的关于Cargo的知识:

1. 我们可以使用cargo new创建一个项目。
2. 我们可以构建一个项目使用cargo build。
3. 我们可以使用cargorun一步构建和运行项目。
4. 我们可以构建一个项目，而不需要生成二进制文件来使用cargocheck检查错误
5. Cargo没有将构建结果保存在代码所在的目录中，而是将其存储在target/debug目录中。

使用Cargo的另一个优点是，无论您使用哪种操作系统，命令都是相同的。

#### 构建为发布程序

当您的项目最终准备好发布时，您可以使用cargo build --release通过优化来编译它。该命令将在target/release而不是target/debug中创建可执行文件。这些优化使您的Rust代码运行得更快，但启用它们会延长程序编译所需的时间。

这就是为什么有两个不同的配置文件:

一个用于开发，当你想快速和频繁地重建，另一个用于构建最终的程序，你会给一个用户，不会重复重建，并将尽可能快地运行。

如果要对代码的运行时间进行基准测试，请确保运行cargo build

--release并对target/release中的可执行文件进行基准测试。

实际上，要在任何现有项目上工作，你可以使用以下命令在Git中检出代码，切换到该项目的目录，然后构建:

```bash
git clone example.org/someproject
cd someproject
cargo build

```

## 总结

你已经有了一个伟大的开始，在你的锈之旅!在本章中，你已经学习了如何:

1. 使用 rustup 安装最新的稳定版本的Rust.

   1. rustup update 更新到较新的Rust版本
   2. rustup self uninstall 卸载Rust
   3. curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh 该命令下载一个脚本并开始安装rustup工具，该工具将安装最新稳定版本的Rust。
2. 打开本地安装的文档 rustup doc
3. 编写并运行一个“你好，世界!”直接使用rustc编程
4. 使用Cargo的约定创建并运行新项目

   这是一个伟大的时间来建立一个更实质性的程序，以适应阅读和编写Rust代码。
