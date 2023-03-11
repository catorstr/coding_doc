# Kraots 入门

>安装kratos cli工具
>
>```bash
>$go install github.com/go-kratos/kratos/cmd/kratos/v2@latest
>```

## 初始化项目

```bash
# 使用默认模板创建项目kratos new <project-name>
kratos new helloworld
# 如在国内环境拉取失败, 可 -r 指定源
kratos new helloworld -r https://gitee.com/go-kratos/kratos-layout.git
# 亦可使用自定义的模板
kratos new helloworld -r xxx-layout.git
# 同时也可以通过环境变量指定源
KRATOS_LAYOUT_REPO=xxx-layout.git
kratos new helloworld
#生成的目录结构如下：
  .
├── Dockerfile  
├── LICENSE
├── Makefile  
├── README.md
├── api // 下面维护了微服务使用的proto文件以及根据它们所生成的go文件
│   └── helloworld
│       └── v1
│           ├── error_reason.pb.go
│           ├── error_reason.proto
│           ├── error_reason.swagger.json
│           ├── greeter.pb.go
│           ├── greeter.proto
│           ├── greeter.swagger.json
│           ├── greeter_grpc.pb.go
│           └── greeter_http.pb.go
├── cmd  // 整个项目启动的入口文件
│   └── server
│       ├── main.go
│       ├── wire.go  // 我们使用wire来维护依赖注入
│       └── wire_gen.go
├── configs  // 这里通常维护一些本地调试用的样例配置文件
│   └── config.yaml
├── generate.go
├── go.mod
├── go.sum
├── internal  // 该服务所有不对外暴露的代码，通常的业务逻辑都在这下面，使用internal避免错误引用
│   ├── biz   // 业务逻辑的组装层，类似 DDD 的 domain 层，data 类似 DDD 的 repo，而 repo 接口在这里定义，使用依赖倒置的原则。
│   │   ├── README.md
│   │   ├── biz.go
│   │   └── greeter.go
│   ├── conf  // 内部使用的config的结构定义，使用proto格式生成
│   │   ├── conf.pb.go
│   │   └── conf.proto
│   ├── data  // 业务数据访问，包含 cache、db 等封装，实现了 biz 的 repo 接口。我们可能会把 data 与 dao 混淆在一起，data 偏重业务的含义，它所要做的是将领域对象重新拿出来，我们去掉了 DDD 的 infra层。
│   │   ├── README.md
│   │   ├── data.go
│   │   └── greeter.go
│   ├── server  // http和grpc实例的创建和配置
│   │   ├── grpc.go
│   │   ├── http.go
│   │   └── server.go
│   └── service  // 实现了 api 定义的服务层，类似 DDD 的 application 层，处理 DTO 到 biz 领域实体的转换(DTO -> DO)，同时协同各类 biz 交互，但是不应处理复杂逻辑
│       ├── README.md
│       ├── greeter.go
│       └── service.go
└── third_party  // api 依赖的第三方proto
    ├── README.md
    ├── google
    │   └── api
    │       ├── annotations.proto
    │       ├── http.proto
    │       └── httpbody.proto
    └── validate
        ├── README.md
        └── validate.proto
```

## make

make命令的使用要与makefile文件在同一级目录下：

```bash
# make help 查看makefile 文件中配置了哪些命令
# make init 初始化项目依赖
# maek api 根据proto文件生成对应的api代码
# kratos run 运行项目
```

## 添加新服务

> kratos默认提供了一个helloword的服务，接下来将介绍如何新增一个服务。

### 定义服务

在api目录中新增一个student服务

```bash
cd api/
mkdir student/v1
#添加一个proto的模板
# 生成 proto 模板,根据它提供的模板结合自己的业务自行编写proto文件
kratos proto add api/student/v1/student.proto
# 生成 client 源码 这一步可以直接make api
kratos proto client api/student/v1/student.proto
# 生成 server 源码 使用 -t 指定生成目录
kratos proto server api/student/v1/student.proto -t internal/service
```

```protobuf
syntax = "proto3";

package api.student.v1;
import "google/api/annotations.proto"; //proto文件中主动添加的这一行报红
/*
IDE中的这个提示不会影响项目的正常编译，如果您需要解决这个报错，请将项目中的thrid_party目录加入Protobuf的custom include paths下。
参考：https://go-kratos.dev/docs/intro/faq
*/

option go_package = "helloworld/api/student/v1;v1";
option java_multiple_files = true;
option java_package = "api.student.v1";

service Student {
	//定义rpc服务
	rpc CallStudent (StudentRequest) returns (StudentReply){
		//http请求 提供http服务的接口
		option (google.api.http) = {
			// 定义一个 GET 接口，并且把 name 映射到 HelloRequest
			get: "/v1/student/{name}/{age}",
			// 可以添加附加接口
			additional_bindings {
				// 定义一个 POST 接口，并且把 body 映射到 HelloRequest
				post: "/v1/student",
				body: "*",
			}
		};
	}
	
}

message StudentRequest {
	string name=1;
	string age =2;
}
message StudentReply {
	string message =1;
}
```

### 注册服务

> 接下来将我们定义好的服务注册到项目中去

```bash
#在internal/biz目录下新建student.go文件
cd internal/biz
touch student.go
```

```go
//internal/biz/student.go

package biz

import (
	"context"

	"github.com/go-kratos/kratos/v2/log"
)

// 定义一个接收参数的struct
type Student struct {
	//字段可以安装参数来，也可自行定义多个字段
	Name string
	Age  int64
}

// 定义一个interface 面向对象编程
type StudentRepo interface {
	Save(context.Context, *Student) (*Student, error)
}

// 定义实例
type StudentUsecase struct {
	repo StudentRepo
	log  *log.Helper
}

func NewStudentUsecase(repo StudentRepo, logger log.Logger) *StudentUsecase {
	return &StudentUsecase{
		repo: repo,
		log:  log.NewHelper(logger),
	}
}
```

然后在biz/biz.go中注册

```go
//biz.go

package biz

import "github.com/google/wire"

// ProviderSet is biz providers.
var ProviderSet = wire.NewSet(NewGreeterUsecase, NewStudentUsecasee)
```

```bash
#在internal/data目录下新建student.go文件
cd internal/data
touch student.go #照猫画虎就好
```

```go
//internal/data/student.go
package data

import (
	"context"
	"helloworld/internal/biz"

	"github.com/go-kratos/kratos/v2/log"
)

//工厂模式

type studentRepo struct {
	data *Data
	log  *log.Helper
}

func NewStudentRepo(data *Data, logger log.Logger) biz.StudentRepo {
	return &studentRepo{
		data: data,
		log:  log.NewHelper(logger),
	}
}

func (s *studentRepo) Save(ctx context.Context, g *biz.Student) (*biz.Student, error) {
	return g, nil
}

```

然后在data/data.go中注册

```go
package data

import (
	"helloworld/internal/conf"

	"github.com/go-kratos/kratos/v2/log"
	"github.com/google/wire"
)

// ProviderSet is data providers.
var ProviderSet = wire.NewSet(NewData, NewGreeterRepo, NewStudentRepo)

// Data .
type Data struct {
	// TODO wrapped database client
}

// NewData .
func NewData(c *conf.Data, logger log.Logger) (*Data, func(), error) {
	cleanup := func() {
		log.NewHelper(logger).Info("closing the data resources")
	}
	return &Data{}, cleanup, nil
}

```

```go
//编写internal/service目录下新建student.go文件 此文件已经由上一步的kratos proto server ... 命令生成
package service

import (
	"context"
	"fmt"

	pb "helloworld/api/student/v1"
	"helloworld/internal/biz"
)

type StudentService struct {
	pb.UnimplementedStudentServer
	uc *biz.StudentUsecase //新增
}
//新增
func NewStudentService(uc *biz.StudentUsecase) *StudentService {
	return &StudentService{
		uc: uc,
	}
}

func (s *StudentService) CallStudent(ctx context.Context, req *pb.StudentRequest) (*pb.StudentReply, error) {
	//service 层相当于控制层
	//这里示例就简单写一点业务层的代码。业务代码一般建议在biz层编写 让service引用biz层的一些方法
	smg := fmt.Sprintf("%s--%v", req.Name, req.Age)
	return &pb.StudentReply{
		Message: smg,
	}, nil
}

```

然后在service/service.go中注册

```go
package service

import "github.com/google/wire"

// ProviderSet is service providers.
var ProviderSet = wire.NewSet(NewGreeterService, NewStudentService)

```

```go
//编写internal/server 下的代码
//http.go
package server

import (
	v1 "helloworld/api/helloworld/v1"
	studentV1 "helloworld/api/student/v1"
	"helloworld/internal/conf"
	"helloworld/internal/service"

	"github.com/go-kratos/kratos/v2/log"
	"github.com/go-kratos/kratos/v2/middleware/recovery"
	"github.com/go-kratos/kratos/v2/transport/http"
)

// NewHTTPServer new an HTTP server. //新增参数student *service.StudentService 新增一个服务 需要将这个服务的server当作参数加进去
func NewHTTPServer(c *conf.Server, greeter *service.GreeterService, student *service.StudentService, logger log.Logger) *http.Server {
	var opts = []http.ServerOption{
		http.Middleware(
			recovery.Recovery(),
		),
	}
	if c.Http.Network != "" {
		opts = append(opts, http.Network(c.Http.Network))
	}
	if c.Http.Addr != "" {
		opts = append(opts, http.Address(c.Http.Addr))
	}
	if c.Http.Timeout != nil {
		opts = append(opts, http.Timeout(c.Http.Timeout.AsDuration()))
	}
	srv := http.NewServer(opts...)
	v1.RegisterGreeterHTTPServer(srv, greeter)
	//新增
	studentV1.RegisterStudentHTTPServer(srv, student)
	return srv
}


//grpc.go
package server

import (
	v1 "helloworld/api/helloworld/v1"
	studentV1 "helloworld/api/student/v1"
	"helloworld/internal/conf"
	"helloworld/internal/service"

	"github.com/go-kratos/kratos/v2/log"
	"github.com/go-kratos/kratos/v2/middleware/recovery"
	"github.com/go-kratos/kratos/v2/transport/grpc"
)

// NewGRPCServer new a gRPC server. //新增student *service.StudentService 新增一个服务 需要将这个服务的server当作参数加进去
func NewGRPCServer(c *conf.Server, greeter *service.GreeterService, student *service.StudentService, logger log.Logger) *grpc.Server {
	var opts = []grpc.ServerOption{
		grpc.Middleware(
			recovery.Recovery(),
		),
	}
	if c.Grpc.Network != "" {
		opts = append(opts, grpc.Network(c.Grpc.Network))
	}
	if c.Grpc.Addr != "" {
		opts = append(opts, grpc.Address(c.Grpc.Addr))
	}
	if c.Grpc.Timeout != nil {
		opts = append(opts, grpc.Timeout(c.Grpc.Timeout.AsDuration()))
	}
	srv := grpc.NewServer(opts...)
	v1.RegisterGreeterServer(srv, greeter)
	studentV1.RegisterStudentServer(srv, student)
	return srv
}

```

这个时候cmd/helloword目录下的wire_gen.go报错了，接下来使用wire依赖注入：

> cd到cmd的main同级目录中执行wire命令重新生成wire_gen.go的代码
>
> *#wire 命令安装*
> go install github.com/google/wire/cmd/wire
>
> 在我们代码中的data、service、biz这些层我们只需要将各种New方法注入到ProviderSet中，然后在cmd下面服务的wire.go中加上对应包中的ProviderSet，在cmd下执行wire命令就可以直接生成wire_gen.go文件了。
>
> 为什么使用依赖注入；
>
> 1. 优雅地解决在main.go中写很多new mysql、redis等等由server、service、biz、data依赖的对象；
>
> kratos用wire默认做了什么事情？
>
> 1. kratos在某服务的internal\server、service、biz、data等包里定义了ProviderSet；
> 2. 进入某服务的cmd/wire.go，调用了wire.Build()：
> 3. 细看wire.go，会发现wire抓server、service、biz、data的ProviderSet所对应的方法；
> 4. 其实这是kratos为我们写好，默认就好了；
> 5. 所以，在这个目录下打开bash，执行wire，就能生成wire_gen.go文件；
> 6. 在main.go中调用wire_gen.go的wireApp；
> 7. 所以我们写代码时，在依赖方面，只需把依赖写在构造方法的参数中就可以了；

## 新增服务大功告成

```bash
kratos run #测试接口吧
```

## kratos项目配置文件的加载以及data层的使用介绍

> kratos项目的配置文件默认在configs/config.yaml文件中，配置文件以yaml文件的格式书写，根据我们的项目具体情况自行编写。另外在kratos初始项目中的internal/conf/conf.proto和conf.pb.go(根据kratos项目的makefile文件生成的，指令make config)

### internal/conf/conf.proto

> conf.proto 是与configs/config.yaml相关联的

```protobuf
//conf.proto
syntax = "proto3";
package kratos.api;

option go_package = "helloworld/internal/conf;conf";

import "google/protobuf/duration.proto";


message Bootstrap { //Bootstrap可以理解为配置文件的根
  Server server = 1; //Server 对应config.yaml文件的一个根
  Data data = 2; // Data 同Server一样
}

message Server { //对应config.yaml文件的一个根下面的二级根
  message HTTP {
    string network = 1;
    string addr = 2;
    google.protobuf.Duration timeout = 3;
  }
  message GRPC {//对应config.yaml文件的一个根下面的三级根字段
    string network = 1;
    string addr = 2;
    google.protobuf.Duration timeout = 3;
  }
  HTTP http = 1;//对应config.yaml文件的一个根下面的二级根字段
  GRPC grpc = 2;
}

message Data {
  message Database {
    string driver = 1;
    string source = 2;
  }
  message Redis {
    string network = 1;
    string addr = 2;
    google.protobuf.Duration read_timeout = 3;
    google.protobuf.Duration write_timeout = 4;
  }
  Database database = 1;
  Redis redis = 2;
}
```

> kartos项目在运行时基于初始化模板，会自己在cmd目录下的main.go中自动加载了。配置加载完了之后就看如何去使用了。

### 在data层使用加载的配置

> 在kratos初始项目中的internal/data/data.go文件中，在NewData的时候就已经使用参数的形式传进去了(依赖注入)
>
> ```go
> //internal/data/data.go
> package data
> 
> import (
> 	"github.com/go-redis/redis/extra/redisotel/v8"
> 	"github.com/go-redis/redis/v8"
> 	"go.opentelemetry.io/otel/attribute"
> 	"helloworld/internal/conf"
> 	"time"
> 
> 	"github.com/dtm-labs/rockscache"
> 	"github.com/duke-git/lancet/convertor"
> 	"github.com/go-kratos/kratos/v2/log"
> 	"github.com/google/wire"
> 	"gorm.io/driver/mysql"
> 	"gorm.io/gorm"
> )
> 
> // ProviderSet is data providers.
> // Notice data层需要初始化的Mysql、Redis等资源直接放在这里即可，wire的时候会生成初始化代码
> var ProviderSet = wire.NewSet(NewData, NewMysql, NewRedis, NewRocksCache, NewGreeterRepo, NewStudentRepo)
> 
> // Data .
> type Data struct {
> 	// wrapped database client
> 	Mysql      *gorm.DB
> 	RedisCli   *redis.Client
> 	RocksCache *rockscache.Client // 缓存一致性的库
> }
> 
> // NewData .
> func NewData(c *conf.Data, logger log.Logger, mysqlDB *gorm.DB, redisCli *redis.Client, rocksCli *rockscache.Client) (*Data, func(), error) {
> 	cleanup := func() {
> 		log.NewHelper(logger).Info("closing the data resources")
> 	}
> 	// Notice 初始化Data后封装了Mysql与Redis客户端
> 	return &Data{
> 		Mysql:      mysqlDB,
> 		RedisCli:   redisCli,
> 		RocksCache: rocksCli,
> 	}, cleanup, nil
> }
> 
> func NewMysql(c *conf.Data, logger log.Logger) *gorm.DB {
> 	// recover
> 	defer func() {
> 		if err := recover(); err != nil {
> 			log.NewHelper(logger).Errorw("kind", "mysql", "error", err)
> 		}
> 	}()
> 	// Notice pkg
> 	// mysql数据库连接
> 	db, err := gorm.Open(mysql.Open(c.Database.Source)) //c.Database.Source使用配置
> 	if err != nil {
> 		panic(err)
> 	}
> 
> 	sqlDB, err := db.DB()
> 	if err != nil {
> 		panic(err)
> 	}
> 
> 	// 连接池参数设置
> 	sqlDB.SetMaxIdleConns(int(c.Database.MinIdleConns))
> 	sqlDB.SetMaxOpenConns(int(c.Database.MaxOpenConns))
> 	sqlDB.SetConnMaxLifetime(time.Hour * time.Duration(c.Database.ConMaxLeftTime))
> 
> 	return db
> }
> 
> func NewRedis(c *conf.Data, logger log.Logger) *redis.Client {
> 	rdb := redis.NewClient(&redis.Options{
> 		Addr: c.Redis.Addr,
> 		//Password:     c.Redis.Password,
> 		DB:           int(c.Redis.Db),
> 		PoolSize:     int(c.Redis.PoolSize),     // 连接池数量
> 		MinIdleConns: int(c.Redis.MinIdleConns), //好比最小连接数
> 		MaxRetries:   int(c.Redis.MaxRetries),   // 命令执行失败时，最多重试多少次，默认为0即不重试
> 	})
> 	rdb.AddHook(redisotel.NewTracingHook(
> 		redisotel.WithAttributes(
> 			attribute.String("db.type", "redis"),
> 			attribute.String("db.ip", c.Redis.Addr),
> 			attribute.String("db.instance", c.Redis.Addr+"/"+convertor.ToString(c.Redis.Db)),
> 		),
> 	))
> 	log.NewHelper(logger).Infow("kind", "redis", "status", "enable")
> 	return rdb
> }
> 
> func NewRocksCache(c *conf.Data, logger log.Logger, rdb *redis.Client) *rockscache.Client {
> 	var dc = rockscache.NewClient(rdb, rockscache.NewDefaultOptions())
> 	log.NewHelper(logger).Infow("kind", "rocksCache", "status", "enable")
> 	return dc
> }
> 
> ```
