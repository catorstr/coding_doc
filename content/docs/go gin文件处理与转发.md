# gin 文件上传

### 单个文件

文件上传前端页面代码：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>上传文件示例</title>
</head>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="f1">
    <input type="submit" value="上传">
</form>
</body>
</html>
```

后端gin框架部分代码：

```go
func main() {
	router := gin.Default()
	// 处理multipart forms提交文件时默认的内存限制是32 MiB
	// 可以通过下面的方式修改
	// router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// 单个文件
		file, err := c.FormFile("f1")
		if err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{
				"message": err.Error(),
			})
			return
		}

		log.Println(file.Filename)
		dst := fmt.Sprintf("C:/tmp/%s", file.Filename)
		// 上传文件到指定的目录
		c.SaveUploadedFile(file, dst)
		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("'%s' uploaded!", file.Filename),
		})
	})
	router.Run()
}
```

### 多个文件上传

```go
func main() {
	router := gin.Default()
	// 处理multipart forms提交文件时默认的内存限制是32 MiB
	// 可以通过下面的方式修改
	// router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// Multipart form
		form, _ := c.MultipartForm()
		files := form.File["file"]

		for index, file := range files {
			log.Println(file.Filename)
			dst := fmt.Sprintf("C:/tmp/%s_%d", file.Filename, index)
			// 上传文件到指定的目录
			c.SaveUploadedFile(file, dst)
		}
		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("%d files uploaded!", len(files)),
		})
	})
	router.Run()
}
```

### gin 文件转发

> 在开发中，有这样的一种场景，就比如在前边我们已经写好了一个gin的服务器1，用来接收客户端上传的文件。但是现在我们不能让客户直接访问服务器1，我们给他访问另一个服务器2，但是服务2没有处理文件的业务，因为这时服务器1已经部署上线了，我们也没有必要在服务器2另写一个处理文件的业务。此时，文件转发就体现了他的一个作用。以下是一个文件转发的例子：

```go
//为了方便，我们把当前服务器(用户可以直接俄访问的服务器)叫做服务器1，用户不能访问的服务器叫做服务器2.其中服务器2使用的是gin的`FormFile`函数接收文件：
// 首先，创建一个multipart/form-data类型的文件上传请求
formData := &bytes.Buffer{}
writer := multipart.NewWriter(formData)

// 将上传的文件写入请求中
fileWriter, err := writer.CreateFormFile("file", file.Filename)
if err != nil {
    // 处理错误
}
_, err = io.Copy(fileWriter, file)
if err != nil {
    // 处理错误
}

// 关闭请求的Writer
err = writer.Close()
if err != nil {
    // 处理错误
}

// 创建一个新的http请求，将请求数据发送给服务器2
req, err := http.NewRequest("POST", "http://server2.com/upload", formData)
if err != nil {
    // 处理错误
}
req.Header.Set("Content-Type", writer.FormDataContentType())

// 发送请求
client := &http.Client{}
resp, err := client.Do(req)
if err != nil {
    // 处理错误
}
defer resp.Body.Close()

// 处理响应
```

在上面的代码中，我们首先使用 `multipart.Writer`创建一个新的multipart/form-data类型的请求，然后将上传的文件写入请求中。我们还设置了正确的请求头，以便服务器2可以正确地解析文件。最后，我们使用 `http.Client`发送请求，并处理响应。请注意，上述代码仅供参考，并且可能需要根据您的具体用例进行修改。特别是，需要将URL和请求头字段替换为您的服务器2的实际值。

### gin 处理既有文件又有其他参数的请求

在处理即有文件又有其他参数的请求时，我们可以使用 `MultipartForm()`方法解析请求体，并使用 `form.File`和 `form.Value`分别获取文件和其他参数(当然更高效的方法当然是用 `ShouldBind或ShouldBindJSON来接收了`)。因为文件是可选的，所以需要检查 `form.File`是否为空来确定是否上传了文件。如果上传了文件，`form.File["file"]`将返回一个包含 `*multipart.FileHeader`对象的slice。我们可以通过循环来处理每个文件。

以下是一个处理这种请求的示例：

```go
 func uploadHandler(c *gin.Context) {
    // 获取表单数据
    form, err := c.MultipartForm()
    if err != nil {
        c.String(http.StatusBadRequest, fmt.Sprintf("get form err: %s", err.Error()))
        return
    }

    // 获取文件
    files := form.File["file"]
    if len(files) == 1 {
        // 处理单个文件
        file := files[0]
        err = c.SaveUploadedFile(file, file.Filename)
        if err != nil {
            c.String(http.StatusBadRequest, fmt.Sprintf("upload file err: %s", err.Error()))
            return
        }
        // 处理其他参数
        name := form.Value["name"][0]
        // ...
    } else if len(files) > 1 {
        // 处理多个文件
        for _, file := range files { //直接使用for range 遍历而不加if判断，好像不太行，会报错，一时找不出原因，就这样嵌套一层写了。
            err = c.SaveUploadedFile(file, file.Filename)
            if err != nil {
                c.String(http.StatusBadRequest, fmt.Sprintf("upload file err: %s", err.Error()))
                return
            }
        }
        // 处理其他参数
        name := form.Value["name"][0]
        // ...
    } else {
        // 没有文件上传
        // 处理其他参数
        name := form.Value["name"][0]
        // ...
    }
    c.String(http.StatusOK, fmt.Sprintf("Upload file ok"))
}
```

在这个示例中，我们首先使用 `MultipartForm()`方法获取表单数据，然后使用 `form.File`和 `form.Value`分别获取文件和其他参数。我们通过检查 `len(files)`来确定是否上传了文件，如果上传了一个文件，我们可以直接处理文件和其他参数；如果上传了多个文件，我们需要使用循环来处理每个文件；如果没有上传文件，则可以直接处理其他参数。最后，我们可以使用 `String`方法返回响应。

这里，当编写了完整的业务代码后，使用postAPI调试接口时报错 "no multipart boundary param in Content-Type"，这个错误通常是由于请求头中的Content-Type不正确引起的。当使用multipart/form-data格式发送POST请求时，需要在请求头中设置Content-Type，并在Content-Type中包含一个分界线参数，用于区分请求体中不同的部分。

要解决这个问题，您需要确保使用正确的Content-Type并设置正确的分界线参数。您可以使用以下代码设置请求头：

```html
headers = {'Content-Type': 'multipart/form-data; boundary=分界线参数'}
```

请注意，分界线参数是任意选择的字符串，只需要确保它在整个请求体中是唯一的。

如果您使用的是第三方库（例如requests），那么通常会自动设置正确的Content-Type和分界线参数。但是，如果您手动构建请求，请确保正确设置请求头并包含正确的分界线参数。
