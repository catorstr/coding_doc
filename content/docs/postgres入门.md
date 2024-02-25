# Postgres 入门

### 安装

> postgres在ubuntu上的安装很简单：
>
> ```bash
> sudo apt-get update
> sudo apt-get install postgresql postgresql-client
> ```
>
> 安装完毕后，系统会创建一个数据库超级用户 postgres，密码为空。
>
> ```
> sudo -i -u postgres #登入
> psql --version #查看安装的版本
> psql  #进入命令行
> \q #退出命令行
> ```
>
> 安装后的配置：
>
> 1. 配置远程连接，默认只允许本地访问：
>    ```
>
>    vim /etc/postgr
>    esql/12/main/postgresql.conf #打开其配置文件1
>    #在配置文件中找到listen_addresses并将其值修改为*，表示允许来自任何主机的连接，(为了安全性也可以指定ip的连接)
>    vim /etc/postgresql/12/main/pg_hba.conf #修改配置文件2
>    #将 #IPv4 local connetctions:下一行的127.0.0.1/32  改为0.0.0.0/0
>
>    #重启
>    sudo service postgresql restart /systemctl restart postgresql
>    systemctl enable postgresql #设置开机自启
>    systemctl status postgresql #查看服务状态
>    ```
> 2. 为超级用户postgres添加密码：
>    ```
>    sudo -i -u postgres
>    psql
>    \passwd   #回车输入密码。
>    #若修改不成功，使用这个：
>    alter user your_username with password 'new_password';#提示 ALTER ROLE 表示成功。
>
>    # 或者创建新的用户
>    create role your_username with login password 'your_password';
>    # 创建数据库
>    create database  db_name;
>    #授予数据库访问权限：
>    grant all privileges on database db_name to your_username;#这将对授予用户对数据库db_name的所有权限，包过创建表，插入，更新，删除等权限。
>    #移除指定账户指定数据库的所有权限
>    revoke all privileges on database db_name from your_username;
>    #删除用户
>    drop user your_username;
>
>    ## 在某数据库中，将该数据库授权给其他用户
>    grant all privileges on database db_name to your_username;
>    #此时用户还没有读写权限，需要继续授权
>    grant all privileges on all tables in schema public to your_username;#在pg中用户与角色木有明显的区别，即用户和角色是一样的
>    #创建角色
>    create role role_name;#注意角色不带有登入的属性，即创建角色，角色是不能登入的。
>
>    ```
>
>    3. 安装sqlx-cli
>
>       > Sqlx-cli是一个命令行工具，它可以用于执行sqlx命令，例如创建表、插入数据、查询数据等。安装: ``cargo install sqlx-cli --features postgres``这将安装SQLx-CLI和PostgreSQL数据库支持。
>       >
>
>       `sqlx migrate add init -r` 是 Rust 中的一个命令，用于向数据库中添加一个名为 `init` 的迁移脚本。该命令的选项包括：
>
>       * `-r`：表示递归地创建迁移脚本。如果没有指定该选项，则只会创建当前目录下的迁移脚本。
>
>       当你运行 `sqlx migrate add init -r` 命令时，SQLx 会在当前目录下创建一个名为 `init.sql` 的迁移脚本文件。该文件包含一个名为 `up` 的 SQL 语句，用于创建数据库表或执行其他必要的操作。
>
>       例如，如果你要创建一个名为 `users` 的表，可以在 `init.sql` 文件中添加以下 SQL 语句：
>
>       <pre><div class="relative"><div></div><pre><code class="language-sql"><span><span class="token">CREATE</span><span></span><span class="token">TABLE</span><span> users </span><span class="token">(</span><span>
>       </span></span><span><span>    id </span><span class="token">SERIAL</span><span></span><span class="token">PRIMARY</span><span></span><span class="token">KEY</span><span class="token">,</span><span>
>       </span></span><span><span>    name </span><span class="token">VARCHAR</span><span class="token">(</span><span class="token">255</span><span class="token">)</span><span></span><span class="token">NOT</span><span></span><span class="token">NULL</span><span class="token">,</span><span>
>       </span></span><span><span>    email </span><span class="token">VARCHAR</span><span class="token">(</span><span class="token">255</span><span class="token">)</span><span></span><span class="token">NOT</span><span></span><span class="token">NULL</span><span></span><span class="token">UNIQUE</span><span class="token">,</span><span>
>       </span></span><span><span>    password </span><span class="token">VARCHAR</span><span class="token">(</span><span class="token">255</span><span class="token">)</span><span></span><span class="token">NOT</span><span></span><span class="token">NULL</span><span>
>       </span></span><span><span></span><span class="token">)</span><span class="token">;</span><span>
>       </span></span><span></span></code></pre><div></div><button class="absolute right-12 top-0 mt-2 mr-2 hover:text-blue-500"><svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 512 512" class="w-5 h-5" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><rect width="336" height="336" x="128" y="128" fill="none" stroke-linejoin="round" stroke-width="32" rx="57" ry="57"></rect><path fill="none" stroke-linecap="round" stroke-linejoin="round" stroke-width="32" d="M383.5 128l.5-24a56.16 56.16 0 00-56-56H112a64.19 64.19 0 00-64 64v216a56.16 56.16 0 0056 56h24"></path></svg></button><button class="absolute right-6 top-0 mt-2 mr-2 hover:text-blue-500"><svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 512 512" class="w-5 h-5" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><path fill="none" stroke-linecap="round" stroke-linejoin="round" stroke-width="32" d="M160 368L32 256l128-112m192 224l128-112-128-112"></path></svg></button><button class="absolute right-0 top-0 mt-2 mr-2 hover:text-blue-500"><svg stroke="currentColor" fill="currentColor" stroke-width="0" viewBox="0 0 512 512" class="w-5 h-5" height="1em" width="1em" xmlns="http://www.w3.org/2000/svg"><path fill="none" stroke-linecap="round" stroke-linejoin="round" stroke-width="48" d="M268 112l144 144-144 144m124-144H100"></path></svg></button></div></pre>
>
>       然后，你可以运行 `sqlx migrate run` 命令来执行该迁移脚本，从而创建 `users` 表。
>
>       需要注意的是，`sqlx migrate add init -r`命令只是向数据库中添加了一个迁移脚本，它并不会执行该脚本。如果你想要执行该脚本，需要运行 `sqlx migrate run` 命令。
>    3. ...

### 使用rust创建客户端连接池并操作数据库。

> 前提：
>
> ```
> sudo -i -u postgres
> psql
>  create database rustdb; #也就是先创建好数据库。
> ```

Cargo.toml

```toml
[package]
name = "postgres"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
sqlx = { version = "0.7", features = [ "runtime-tokio","postgres" ] }
tokio = { version = "1.35.0", features = ["full"] }
```

main.rs

```rust
use sqlx::postgres::PgPoolOptions;
#[tokio::main]
async fn main() ->Result<(),sqlx::Error>{
    let url = "postgres://rustcon:root@127.0.0.1/runoobdb";
    //连接数据库
    let db_client = sqlx::postgres::PgPoolOptions::new().max_connections(20).connect(&url).await?;
    //  let db_client: sqlx::Pool<_> = sqlx::pool::PoolOptions::new().max_connections(10).connect(url).await?;
    println!("db_client:{:?}",db_client);

    //执行建表sql语句
    let sql ="create table if not exists email(
            id INT PRIMARY KEY,
            name VARCHAR(20) NOT NULL,
            email VARCHAR(25) NOT NULL)";
    let res = sqlx::query(sql)
    .execute(&db_client)
    .await?;
    //检查执行结果(根据需要而定)
    if res.rows_affected()>0{
        println!("table created successfully");
    }else{
        println!("table create error");
    }
    Ok(())
}


#[derive(sqlx::FromRow)]
struct User{
    id:i32,
    name:String,
    email:String,
}
/*
#[derive(sqlx::FromRow)] 是一个宏属性，用于为结构体自动生成实现 sqlx::FromRow trait 的代码。
FromRow trait 是 SQLx 提供的一个 trait，用于将数据库查询结果转换为 Rust 结构体。
通过使用 #[derive(sqlx::FromRow)]，您可以自动实现 FromRow trait，
从而使得您可以方便地将数据库查询结果映射到 Rust 结构体中的字段。
这样做的好处是，当您执行查询并获取结果时，可以直接使用 fetch 或 fetch_all 方法将结果转换为定义的结构体类型，
而无需手动解析和映射每个字段。
如果您不定义 #[derive(sqlx::FromRow)]，则需要手动编写代码来解析查询结果并将其映射到结构体中的字段。
因此，定义 #[derive(sqlx::FromRow)] 是为了方便地将查询结果映射到结构体中，提供了更简洁和易用的方式。
但如果您不想使用这种自动映射的功能，可以选择手动解析查询结果并将其映射到结构体中的字段。
*/
```
