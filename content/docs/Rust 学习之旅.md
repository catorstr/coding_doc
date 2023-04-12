# Rust 学习之旅

## 安装Rust

>  #https://www.rust-lang.org/zh-CN/learn/get-started
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

sudo apt update
sudo apt install build-essential

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
