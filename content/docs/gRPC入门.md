# grpc入门操作

> https://grpc.io/docs/what-is-grpc/introduction
>
> https://grpc.io/docs/languages/go
>
> 关于grpc的概念及它的作用，这里就不说了，直接上代码。

### 服务定义

```properties
//productInfo.proto
syntax = "proto3";
package product;
// './pb':指定生成文件的路径，也可在命令行参数中指定；'product':指定生成的go文件的包名，
option go_package="./pb;product";
service ProductInfo{
    rpc addProduct(Product)returns(ProductId);
    rpc getProduct(ProductId)returns(Product){};
}

message Product{
    string id=1;
    string name =2;
    string description=3;
}

message ProductId{
    string id =1;
}
```

### 编译

> 在.proto 文件所在的目录打开终端。
>
> ```bash
> protoc --proto_path=. --go-grpc_out=. --go_out=:. ./productInfo.proto
> ```

### gRPC服务器端

### gRPC-Gateway

> 由谷歌开发的一个protoc插件，该项目旨在为您的GRPC服务提供HTTP+JSON接口。在您的服务中只需要少量的信任来附加HTTP语义，就可以用这个库生成一个反向代理。
>
> 参考：
>
> https://github.com/grpc-ecosystem/grpc-gateway
>
> https://grpc-ecosystem.github.io/grpc-gateway
>
> https://grpc-ecosystem.github.io/grpc-gateway/docs/tutorials/introduction
