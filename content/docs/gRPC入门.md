# grpc入门操作

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
