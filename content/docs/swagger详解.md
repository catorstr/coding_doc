# 在gin 中如何使用Swagger

## **前言**

在前后端分离的项目开发过程中，如果后端同学能够提供一份清晰明了的接口文档，那么就能极大地提高大家的沟通效率和开发效率。那如何维护接口文档，历来都是令人头痛的，感觉很浪费精力，而且后续接口文档的维护也十分耗费精力。在很多年以前，也流行用word等工具写接口文档，这里面的问题很多，如格式不统一、后端人员消费精力大、文档的时效性也无法保障。

针对这类问题，最好是有一种方案能够既满足我们输出文档的需要又能随代码的变更自动更新，Swagger正是那种能帮我们解决接口文档问题的工具。

## **Swagger介绍**

Swagger是基于标准的 OpenAPI 规范进行设计的，本质是一种用于描述使用json表示的Restful Api的接口描述语言，只要照着这套规范去编写你的注解或通过扫描代码去生成注解，就能生成统一标准的接口文档和一系列 Swagger 工具。Swagger包括自动文档，代码生成和测试用例生成。

## **安装Swagger**

### 安装

```bash
go install github.com/swaggo/swag/cmd/swag
```

### 检测安装

```bash
$ swag -v
swag version v1.8.12
```

**###安装gin-swagger扩展**

```text
$ go get -u -v github.com/swaggo/gin-swagger
$ go get -u -v github.com/swaggo/files
$ go get -u -v github.com/alecthomas/template
```

### 使用

使用gin-swagger为你的代码自动生成接口文档，一般需要下面三个步骤：

1. 按照swagger要求给接口代码添加声明式注释。
2. 使用swag工具扫描代码自动生成api接口文档数据。
3. 使用gin-swagger渲染在线接口文档页面。

### 添加注释

go-swapper注解规范说明：

注：注解详情可参见官网文档 ***[Swagger Documentation](https://link.zhihu.com/?target=https%3A//link.juejin.cn/%3Ftarget%3Dhttps%253A%252F%252Fswagger.io%252Fdocs%252F)***

| 注解     | 描述                                                                                                       |
| -------- | ---------------------------------------------------------------------------------------------------------- |
| @Summary | 摘要                                                                                                       |
| @Produce | API 可以产生的 MIME 类型的列表，MIME 类型你可以简单的理解为响应类型，例如：json、xml、html 等等            |
| @Param   | 参数格式，从左到右分别为：参数名、入参类型(param type)、数据类型(data type)、是否必填(true or fasle)、注释 |
| @Success | 响应成功，从左到右分别为：状态码、参数类型、数据类型、注释                                                 |
| @Failure | 响应失败，从左到右分别为：状态码、参数类型、数据类型、注释                                                 |
| @Router  | 路由，从左到右分别为：路由地址，HTTP 方法                                                                  |

#### Param Type

* object (struct)
* string (string)
* integer (int, uint, uint32, uint64)
* number (float32)
* boolean (bool)
* array

#### Data Type

* string (string)
* integer (int, uint, uint32, uint64)
* number (float32)
* boolean (bool)
* user defined struct

示例demo：

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/swaggo/files"
	ginSwagger "github.com/swaggo/gin-swagger"
	_ "github/mwqnice/swag/docs" // 千万不要忘了导入把你上一步生成的docs
)

type Article struct{
	ID         uint32 `gorm:"primary_key" json:"id"`
	CreatedBy  string `json:"created_by"`
	ModifiedBy string `json:"modified_by"`
	CreatedOn  uint32 `json:"created_on"`
	ModifiedOn uint32 `json:"modified_on"`
	DeletedOn  uint32 `json:"deleted_on"`
	IsDel      uint8  `json:"is_del"`
	Title         string `json:"title"`
	Desc          string `json:"desc"`
	Content       string `json:"content"`
	CoverImageUrl string `json:"cover_image_url"`
	State         uint8  `json:"state"`
}

func NewArticle() Article {
	return Article{}
}

func main()  {
	r := gin.Default()
	r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
	r.Run(":8088")
}

// @Summary 获取单个文章
// @Produce json
// @Param id path int true "文章ID"
// @Success 200 {object} Article "成功"
// @Failure 400 {object} string "请求错误"
// @Failure 500 {object} string "内部错误"
// @Router /api/v1/articles/{id} [get]
func (a Article) Get(c *gin.Context) {

}

// @Summary 获取多个文章
// @Produce json
// @Param name query string false "文章名称"
// @Param tag_id query int false "标签ID"
// @Param state query int false "状态"
// @Param page query int false "页码"
// @Param page_size query int false "每页数量"
// @Success 200 {object} Article "成功"
// @Failure 400 {object} string "请求错误"
// @Failure 500 {object} string "内部错误"
// @Router /api/v1/articles [get]
func (a Article) List(c *gin.Context) {
	return
}

// @Summary 创建文章
// @Produce json
// @Param tag_id body string true "标签ID"
// @Param title body string true "文章标题"
// @Param desc body string false "文章简述"
// @Param cover_image_url body string true "封面图片地址"
// @Param content body string true "文章内容"
// @Param created_by body int true "创建者"
// @Param state body int false "状态"
// @Success 200 {object} Article "成功"
// @Failure 400 {object} string "请求错误"
// @Failure 500 {object} string "内部错误"
// @Router /api/v1/articles [post]
func (a Article) Create(c *gin.Context) {

}

// @Summary 更新文章
// @Produce json
// @Param tag_id body string false "标签ID"
// @Param title body string false "文章标题"
// @Param desc body string false "文章简述"
// @Param cover_image_url body string false "封面图片地址"
// @Param content body string false "文章内容"
// @Param modified_by body string true "修改者"
// @Success 200 {object} Article "成功"
// @Failure 400 {object} string "请求错误"
// @Failure 500 {object} string "内部错误"
// @Router /api/v1/articles/{id} [put]
func (a Article) Update(c *gin.Context) {
	return
}

// @Summary 删除文章
// @Produce  json
// @Param id path int true "文章ID"
// @Success 200 {string} string "成功"
// @Failure 400 {object} string "请求错误"
// @Failure 500 {object} string "内部错误"
// @Router /api/v1/articles/{id} [delete]
func (a Article) Delete(c *gin.Context) {
	return
}
```

## **2、生成接口文档数据**

格式化swag注解

```text
$ swag fmt
```

在项目根目录执行以下命令，使用swag工具生成接口文档数据。

```java
$ swag init
```

执行完上述命令后，如果你写的注释格式没问题，此时你的项目根目录下会多出一个docs文件夹。

./docs

├── docs.go

├── swagger.json

└── swagger.yaml

## **3、引入gin-swagger渲染文档数据**

然后在项目代码中注册路由的地方按如下方式引入gin-swagger相关内容：

```text
import (
	"github.com/gin-gonic/gin"
	"github.com/swaggo/files"
	ginSwagger "github.com/swaggo/gin-swagger"
	_ "github/mwqnice/swag/docs" // 千万不要忘了导入把你上一步生成的docs
)
//添加swagger访问路由
r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```

启动项目，在浏览器中输入地址： ***[http://127.0.0.1:8088/swagger/index.html](https://link.zhihu.com/?target=https%3A//link.juejin.cn/%3Ftarget%3Dhttp%253A%252F%252F127.0.0.1%253A9090%252Fswagger%252Findex.html)***

### 更多

> https://www.liwenzhou.com/posts/Go/gin-swagger/
