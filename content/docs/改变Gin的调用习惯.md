# 改变Gin的调用习惯

```go
package main
import (
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/langwan/chigo/Gin/ReturnFunction/api"
	helper_code "github.com/langwan/langgo/helpers/code"
	"io"
)

func main() {
	g := gin.Default()
	g.Any("api/*uri", ApiHandler()) //接管gin的二级路由 之后就不用加路由了，访问方式后面介绍
	g.Any("hello", HelloHandler) //默认模式
	g.Run(":9010")
}

//改变api的模式
func ApiHandler() gin.HandlerFunc {
	a := api.Api{}
	return func(c *gin.Context) {
		methodName := c.Param("uri")[1:]

		body, err := io.ReadAll(c.Request.Body)
		if err != nil {
			return
		}

		if err != nil {
			c.AbortWithStatus(500)
			return
		}

		response, code, err := helper_code.Call(c, &a, methodName, string(body))

		if err != nil {
			c.AbortWithError(500, err)
		} else if code != 0 {
			a.SendBad(c, code, err.Error(), nil)
		} else {
			a.SendOk(c, response)
		}
	}
}

//gin默认模式的api函数定义
func HelloHandler(c *gin.Context) {
	if true {
		c.AbortWithStatus(403)
		return //这样 错误处理比较麻烦 测试的时候也不好测试
	}
	fmt.Println("end")
}
```

```go
package helper_code

import (
	"context"
	jsoniter "github.com/json-iterator/go"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"reflect"
)

func Call(ctx context.Context, service interface{}, methodName string, request string) (response interface{}, code int, err error) {
	tp := reflect.TypeOf(service)
	method, ok := tp.MethodByName(methodName)
	if !ok {
		return "", int(codes.NotFound), status.Errorf(codes.NotFound, "%s not find", methodName)
	}

	method.Type.NumIn()

	parameter := method.Type.In(2)
	req := reflect.New(parameter.Elem()).Interface()
	jsoniter.ConfigCompatibleWithStandardLibrary.Unmarshal([]byte(request), req)

	in := make([]reflect.Value, 0)
	in = append(in, reflect.ValueOf(ctx))
	in = append(in, reflect.ValueOf(req))
	call := reflect.ValueOf(service).MethodByName(methodName).Call(in)
	if call[1].Interface() != nil {
		e := call[1].Interface().(error)
		st, _ := status.FromError(e)
		return "", int(st.Code()), e
	}

	return call[0].Interface(), 0, nil
}
```

## api的定义

> 上述使用gin的模式，对应的api的定义

```go
//api/api.go
package api

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

type ApiResponse struct {
	Code    int         `json:"code"`
	Message string      `json:"message"`
	Body    interface{} `json:"body" `
}

type Api struct { //Api结构体可以当上下文使用，
    //自定义时段，可以在下文中调用
}

func (api Api) SendOk(c *gin.Context, body any) {
	resp := ApiResponse{}
	resp.Code = 0
	resp.Message = ""
	resp.Body = body
	c.JSON(http.StatusOK, resp)
}

func (api Api) SendBad(c *gin.Context, code int, message string, body any) {
	resp := ApiResponse{}
	resp.Code = code
	resp.Message = message
	resp.Body = body
	c.AbortWithStatusJSON(http.StatusOK, resp)
}
```

```go
//api/Login.go
package api

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

type LoginRequest struct {
	Name string `json:"name"`
}

type LoginResponse struct {
	Message string `json:"message"`
}

//明确的指定输入输出及错误
func (api Api) Login(c *gin.Context, request *LoginRequest) (*LoginResponse, error) { //输入输出错误模式，相对于gin的原始模式更明确
	message := fmt.Sprintf("hi %s, login ok", request.Name)
	return &LoginResponse{Message: message}, nil
}
```

```go
//api/Reg.go
package api

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

type RegRequest struct {
	Name string `json:"name"`
}

type RegResponse struct {
	Message string `json:"message"`
}

func (api Api) Reg(c *gin.Context, request *RegRequest) (*RegResponse, error) {
	message := fmt.Sprintf("hi %s, reg ok", request.Name)
	return &RegResponse{Message: message}, nil
}
```

> 访问上述两个业务的方式：
>
> http://127.0.0.1::9010/api/Login (没错Login是大写开头)
>
> http://127.0.0.1::9010/api/Reg