# docker 常用操作

> Docker是一个开源的容器化平台，可以帮助开发者快速构建、测试和部署应用程序。在使用Docker时，需要使用各种不同的参数来配置和管理容器。下面是一些常用的Docker参数及其解释。

```bash
-d，--detach
#后台运行容器，不阻塞命令行界面。
-p，--publish
3将容器端口映射到主机端口。
-v，--volume
#将主机文件夹映射到容器文件夹，使得容器内的文件可以被主机读写。
-e，--env
#设置容器的环境变量。
--name
#为容器指定一个名称。
-it
#使用交互式终端运行容器。
--rm
#容器退出后自动删除。
--network
#指定容器使用的网络。
--restart
#指定容器在退出后自动重启的策略。
--link
#将容器连接到其他容器。
--cpu-shares
#设置容器的CPU使用权重。
--memory
#设置容器的内存限制。
--privileged
#启用容器的特权模式。
--user
#指定容器使用的用户。
--entrypoint
#指定容器启动时执行的命令。
```

以上是Docker常用的一些参数及其解释。在实际使用中，还有许多其他的参数可以用来配置和管理容器，具体使用还需参考Docker文档。

### docker 与容器

```bash
    docker pull mysql:latest #拉取mysql最新镜像
    docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql #MYSQL_ROOT_PASSWORD=123456：设置 MySQL 服务 root 用户的密码。
    docker exec -it mysql-test bash #进入容器
    #登录mysql
    mysql -u root -p
    #添加远程登录用户
    CREATE USER 'admin'@'%' IDENTIFIED WITH mysql_native_password BY 'admin123';
    GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%';
    #使用admin用户登入，然后创建数据库
    create database fuxichainpro;
    #redis
    docker pull redis:latest
    docker run -itd --name redis-test -p 6379:6379 redis
    docker exec -it redis-test /bin/bash
    redis-cli #通过 redis-cli 连接测试使用 redis 服务。
    #查看是否设置了密码验证：
    127.0.0.1:6379> CONFIG get requirepass
    1) "requirepass"
    2) ""
    #默认情况下 requirepass 参数是空的，这就意味着无需通过密码验证就可以连接到 redis 服务。
    #可以通过以下命令来修改该参数：
    127.0.0.1:6379> CONFIG set requirepass "fuxichainredis"
    OK
    127.0.0.1:6379> CONFIG get requirepass
    1) "requirepass"
    2) "fuxichainredis"
    #设置密码后，客户端连接 redis 服务就需要密码验证，否则无法执行命令。
    127.0.0.1:6379> AUTH "fuxichainredis"
    OK
    127.0.0.1:6379> SET mykey "Test value"
    OK
    127.0.0.1:6379> GET mykey
    "Test value"

#mongo
docker pull mongo  
docker run -itd --name mongo-test -p 27017:27017 mongo  --auth
#拉取最新的镜像
#-d 后台运行  
#-p 27017:27017 ：映射容器服务的 27017 端口到宿主机的 27017 端口。外部可以直接通过 宿主机 ip:27017 访问到 mongo 的服务。
#--name:为容器指定一个名称。
 #--auth：需要密码才能访问容器服务。
#docker exec -it mongodb mongo  
docker exec -it mongo-test mongosh admin
# 创建一个名为 admin，密码为 123456 的用户。
db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
# 尝试使用上面创建的用户信息进行连接。
db.auth('admin', '123456')
quit;
```

### 自己构建一个docker镜像

在一个空的目录下创建一个相关的Dockerfile，其中包含以下内容：

```
FROM mongo
RUN echo "replSet=rs0" >> /etc/mongod.conf
```

这将从mongo镜像创建一个新的Docker镜像，并将副本集配置添加到mongod.conf文件中。

然后，通过 docker build 命令来构建一个镜像。

```
docker build -t my-mongo-image .
```

* **-t** ：指定要创建的目标镜像名
* **.** ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径

使用docker images 查看创建的镜像已经在列表中存在,镜像ID为493672733c51

```bash
$ docker images
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
my-mongo-image   latest    493672733c51   35 minutes ago   646MB

```

这样旧构建了一个镜像了，更多复杂的操作还需要去学习Docker语法。

### docker 常用命令

```bash
docker start/stop/restart 容器id #启动/关闭/重启一个容器
docker ps -a #查看容器状态
docker images #列出当前所有的镜像
docker rm -f 容器id/名称 #删除一个容器
docker rmi 镜像id #删除一个镜像
```

> 如何将自己使用dockerfile构建的镜像上传到docker hub上作为一个使用镜像?

>
> 要将自己使用Dockerfile构建的镜像上传到Docker Hub上，并将其用作镜像，您需要遵循以下步骤：
>
> 1. 登录到Docker Hub并创建一个名为`<your-repository-name>`的仓库。
> 2. 在终端中运行以下命令以登录到Docker Hub：
>
>    ```
>    docker login
>    ```
>
>    您将被要求提供Docker Hub的用户名和密码。
> 3. 在终端中运行以下命令以构建镜像：
>
>    ```
>    docker build -t <your-dockerhub-username>/<your-repository-name>:<tag> .
>    ```
>
>    其中，`<your-dockerhub-username>`是您的Docker Hub用户名，`<your-repository-name>`是您的仓库名称，`<tag>`是您的镜像标记。例如，您可以将镜像标记设置为“latest”。
> 4. 运行以下命令以将您的镜像上传到Docker Hub：
>
>    ```
>    docker push <your-dockerhub-username>/<your-repository-name>:<tag>
>    ```
>
>    这将上传您的镜像并将其存储在Docker Hub中。
> 5. 在另一个主机上，使用以下命令拉取您的镜像：
>
>    ```
>    docker pull <your-dockerhub-username>/<your-repository-name>:<tag>
>    ```
>
>    这将从Docker Hub中下载并存储您的镜像。
> 6. 在主机上运行以下命令以启动容器：
>
>    ```
>    docker run -d -p <host-port>:<container-port> <your-dockerhub-username>/<your-repository-name>:<tag>
>    ```
>
>    其中，`<host-port>`是您要将容器映射到的主机端口，`<container-port>`是容器内部正在运行的端口。例如，如果您的容器正在侦听端口80，则`<container-port>`应为80。
>
>    请注意，此命令将在后台启动容器，并将容器输出连接到主机终端。
>
> 现在，您已经成功将自己使用Dockerfile构建的镜像上传到Docker Hub上，并将其用作镜像启动了一个容器。

> 那如何拉取使用这个镜像呢?

>
> 要拉取使用Docker Hub上的镜像，您需要遵循以下步骤：
>
> 1. 在终端中运行以下命令，以拉取镜像：
>
>    ```
>    docker pull <your-dockerhub-username>/<your-repository-name>:<tag>
>    ```
>
>    其中，`<your-dockerhub-username>`是您的Docker Hub用户名，`<your-repository-name>`是您的仓库名称，`<tag>`是您的镜像标记。例如，您可以将镜像标记设置为“latest”。
> 2. 等待Docker从Docker Hub中下载镜像。该过程可能需要几分钟，具体取决于您的网络连接和镜像的大小。
> 3. 运行以下命令以启动容器：
>
>    ```
>    docker run -d -p <host-port>:<container-port> <your-dockerhub-username>/<your-repository-name>:<tag>
>    ```
>
>    其中，`<host-port>`是您要将容器映射到的主机端口，`<container-port>`是容器内部正在运行的端口。例如，如果您的容器正在侦听端口80，则`<container-port>`应为80。
>
>    请注意，此命令将在后台启动容器，并将容器输出连接到主机终端。
>
> 现在，您已经成功拉取使用Docker Hub上的镜像启动了一个容器。

> 如何将自己使用dockerfile构建的镜像上传到gitlab作为一个使用镜像?

>
> 要将自己使用Dockerfile构建的镜像上传到GitLab并将其用作镜像，您需要遵循以下步骤：
>
> 1. 登录到GitLab并创建一个新项目。
> 2. 在终端中运行以下命令以登录到GitLab：
>
>    ```
>    docker login registry.gitlab.com
>    ```
>    您将被要求提供GitLab的用户名和密码。
> 3. 在终端中运行以下命令以构建镜像：
>
>    ```
>    docker build -t registry.gitlab.com/<your-gitlab-username>/<your-project-name>:<tag> .
>    ```
>    其中，`<your-gitlab-username>`是您的GitLab用户名，`<your-project-name>`是您的项目名称，`<tag>`是您的镜像标记。例如，您可以将镜像标记设置为“latest”。
> 4. 运行以下命令以将您的镜像上传到GitLab：
>
>    ```
>    docker push registry.gitlab.com/<your-gitlab-username>/<your-project-name>:<tag>
>    ```
>    这将上传您的镜像并将其存储在GitLab的容器仓库中。
> 5. 在另一个主机上，使用以下命令拉取您的镜像：
>
>    ```
>    docker login registry.gitlab.com
>    docker pull registry.gitlab.com/<your-gitlab-username>/<your-project-name>:<tag>
>    ```
>    这将从GitLab中下载并存储您的镜像。
> 6. 在主机上运行以下命令以启动容器：
>
>    ```
>    docker run -d -p <host-port>:<container-port> registry.gitlab.com/<your-gitlab-username>/<your-project-name>:<tag>
>    ```
>    其中，`<host-port>`是您要将容器映射到的主机端口，`<container-port>`是容器内部正在运行的端口。例如，如果您的容器正在侦听端口80，则`<container-port>`应为80。
>
>    请注意，此命令将在后台启动容器，并将容器输出连接到主机终端。
>
> 现在，您已经成功将自己使用Dockerfile构建的镜像上传到GitLab上，并将其用作镜像启动了一个容器。
