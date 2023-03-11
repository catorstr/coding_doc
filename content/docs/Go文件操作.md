**Go文件操作**

**设备文件**

>屏幕(标准输出) 如fmt.Println() 往标准输出设备写内容
>
>键盘(标准输入) 如fmt.Scan(）从标准输入设备读取内容



**磁盘文件**

>放在存储设备上的文件，文本文件，二进制文件。
>
>为什么需要文件？内存是掉电丢失的，程序结束，内存中的内容消失，文件放在磁盘，程序结束，文件还存在。

**文件操作常用的接口**

**文件的建立与打开**

```go
func Create(name string) (file *File,err Error)
//功能：根据提供的文件名，创建新的文件，返回一个文件对象权限默认是0666即具有读写权限的的文件，内部调用了OpenFile
func Open(name string)(file *File,err Error)
//功能：打开一个名为name的文件，是只读的方式，内部调用了OpenFile
func OpenFile(name string,flag int,perm uint 32)(file *File,err Error)
//参数：name要打开/创建的文件名，flag打开的方式(只读，只写，读写，创建等),perm是权限。
```

**写文件**

```go
func (file *File)Write(b []byte)(n int,err Error) 
//功能：将byte类型的数据写入文件中 
func (file *File) WriteAt(b []byte,off int64)(n int,err Error) 
//功能：在指定的off位置写入byte类型的数据 
func (file *File)WriteString(s string)(ret int,err Error) //功能:将string类型的数据写入文件中，返回写进去的长度
```

**读文件**

```go
func (file *File)Read(b []byte)(n int,err Error) 
//功能：将文件中的内容读取到b中 
func (file *File)ReadAt(b []byte,off int64)(n int,err Error) 
//功能：从文件中指定的位置读物数据保存到b中
```

**删除文件**

```go
func Remove(name string) Error 
//功能：参数文件名为name的文件
```

**设备文件的使用**

```go
package main  
import (    
    "fmt"    
    "os" 
)  
func main() {    
    //标准设备文件(os.Stdout 和os.Stdin)默认都是打开的。    
    /*        
    os.Stdout 输出到控制台        
    os.Stdin 从控制台输入    
    */    
    //往标准输出设备写数据    
    //fmt.Print 系列默认打印到控制台    
    var str string = "令狐楚"    
    fmt.Println("小王子")    
    fmt.Print(str)    
    fmt.Printf("\nstr: %v\n", str)    
    //fmt.Fprint 系列可以指定你把数据写到哪里去
    fmt.Fprint(os.Stdout, str) 
    //os.Stdout表示写到控制台去    
    //fmt.Sprint 系列可用数据的动态组装    
    for i := 0; i < 10; i++ {        
        str1 := fmt.Sprintf("李信%d", i)        
        fmt.Printf("str1: %v\n", str1)    
    }    
    //fmt.Scan系列默认从标准输入读取数据    
    fmt.Scan(&str)    
    fmt.Println(str)    
    //fmt.Fscan系列指定从哪里读io.Reader
    fmt.Fscan(os.Stdin, &str)    
    fmt.Println(str) 
} 
```

**文件的读写**

**写方式一**

```go
func main() {    
    //创建一个文件    
    f, err := os.Create("test.txt")    
    if err != nil {        
        fmt.Printf("err: %v\n", err)        
        return    
    }    
    //记得关闭文件，描述符   
    defer f.Close()        
    for i := 0; i < 10; i++ {        
        buf := fmt.Sprintf("这是第%d行\n", i)        
        //写入内容，方式一        
        n, err := f.WriteString(buf)        
        if err != nil {            
            fmt.Printf("err: %v\n", err)            
            return        
        }        
        fmt.Printf("写入了%v个字节\n", n)    
    } 
}
```

**写方式二**

```go
package main  
import (    
    "bufio"    
    "fmt"    
    "os" 
) 
func main() {    
    //打开文件    
    f, err := os.OpenFile("test.txt", os.O_APPEND, 0666)
    // os.O_RDWR会从文件头部开始写，并且写多少个字节覆盖原来的多少个字节会覆盖掉    
    //os.O_APPEND 追加    
    if err != nil {       
        fmt.Printf("err: %v\n", err)        
        return    
    }    
    defer f.Close()    
    w := bufio.NewWriter(f)    
    /*Write将p的内容写入缓冲区。 它返回写入的字节数nn。    
    如果nn < len(p)，它还返回一个错误，解释为什么写很短。
    WriteByte写入单个字节。    
    WriteRune写入单个Unicode码位，返回写入的字节数和所有错误。
    WriteString编写字符串。它返回写入的字节数。    
    如果计数小于len(s),则返回一个错误,解释为什么写入是短的。
    */    
    p := []byte("令狐楚")    
    nn, err := w.Write(p)    
    if err != nil {        
        fmt.Printf("err: %v\n", err)    
    }    
    fmt.Printf("nn: %v\n", nn)    
    w.Flush()  
}
```

**读方式一**

```go
func main() {    
    //打开文件    
    f, err := os.Open("test.txt")    
    if err != nil {        
        fmt.Printf("err: %v\n", err)        
        return    
    }    
    defer f.Close()    
    //读取，    
    buf := make([]byte, 1024)    
    //Read从文件中最多读取len(b)个字节。    
    //它返回读取的字节数和遇到的任何错误。 在文件末尾，Read返回0,io.EOF。    
    n, err := f.Read(buf)    
    //想要读取全部的数据buffer要足够大    
    if err != nil && err != io.EOF {
        fmt.Printf("err: %v\n", err)        
        return    
    }    
    fmt.Printf("n: %v\n", n)    
    fmt.Printf("buf: %v\n", string(buf)) 
}
```

**读方式二**

```go
func main() {    
    //打开文件    
    f, err := os.Open("test.txt")    
    if err != nil {        
        fmt.Printf("err: %v\n", err)        
        return    
    }    
    //读取，方式二    
    /*        
    ReadAll从r中读取数据，直到出现错误或EOF，并返回读取的数据。
    一个成功的调用返回err == nil，而不是err == EOF。        
    因为ReadAll被定义为从src读取直到EOF，所以它不会将从read读取的EOF视为要报告的错误。    
    */    
    b, err := io.ReadAll(f)    
    //b, err :=ioutil.ReadAll(f) 
    //功能一模一样    
    if err != nil {        
        fmt.Printf("err: %v\n", err)        
        return    
    }    
    fmt.Printf("b: %v\n", string(b)) 
}
```

**读方式三**

```go
func main() {    
    //打开文件    
    f, err := os.Open("test.txt")    
    if err != nil {        
        fmt.Printf("err: %v\n", err)        
        return    
    }    //读取，方式三    
    //NewReader返回一个新的Reader，其缓冲区大小为默认值。    
    r := bufio.NewReader(f)    
    for {        
        /*            
        把文件中的内容先读取到一个缓冲区中            
        ReadBytes读取直到输入中第一次出现delim为止，
        返回一个包含数据直到并包括分隔符的切片。            
        如果ReadBytes在找到分隔符之前遇到错误，            
        它将返回在错误之前读取的数据和错误本身(通常是io.EOF)。
        当且仅当返回的数据不是以delim结束时，ReadBytes返回err != nil。            
        对于简单的使用，Scanner可能更方便。        
        */        
        buf, err := r.ReadBytes('\n') 
        //每次读取遇到'\n'返回读取结果        
        if err != nil {
            //'\n'也会读取到buf中
            fmt.Printf("err: %v\n", err)            
            return        
        }        
        fmt.Printf("buf: %v\n", string(buf))    
    } 
}
```

**Go生成json文本信息**

**编码json** 

>使用json.Marshal()函数可以对一组数据进行json格式的编码
>
>func Marshal(v interface{})([]byte,err)

**Go通过结构体生成json及json转换为结构体**

```go
package main  
import (    
    "encoding/json"    
    "fmt" 
) 
type Data struct {    
    Name     string   `json:"name"` //成员字段首字母要大写，不然UnMarshal时会不成功。    
    Subjects []string `json:"subject"`    
    Isok     bool     `json:"is_ok"`    
    Age      int      `json:"age"`    
    Score    float64  `json:"score"` 
}  
func main() {     
    data := Data{
        "令狐楚", 
        []string{"计算机", "信息安全"},
        true, 
        22, 
        99.99
    }    
    //结构体转json    
    b, err := json.Marshal(data)    
    if err != nil {        
        fmt.Printf("err: %v\n", err)        
        return    
    }    
    fmt.Println(string(b))    
    //{"name":"令狐楚","subject":["计算机","信息安全"],"is_ok":true,"age":22,"score":99.99}        
    //json转结构体    
    var data2 Data    
    buf := `{"name":"令狐楚","subject":["计算机","信息安全"],"is_ok":true,"age":22,"score":99.99}`    
    if err = json.Unmarshal([]byte(buf), &data2); err != nil {        
        fmt.Println(err)        
        return    
    }    
    fmt.Printf("data2: %v\n", data2)    
    //data2: {令狐楚 [计算机 信息安全] true 22 99.99
}  
} 
```

**json转换为map**

> 太麻烦了，一般用不到，转结构体是更好的方案

```go
func main() {    
    //json转map 缺点读取数据太麻烦    
    //5 对应json的字段，如下面的json字段有name,subject,is_ok,age,score    
    m := make(map[string]interface{}, 5)    
    buf := `{"name":"令狐楚","subject":["计算机","信息安全"],"is_ok":true,"age":22,"score":99.99}`    
    if err := json.Unmarshal([]byte(buf), &m); err != nil{ 
        fmt.Println(err)        
        return    
    }    
    fmt.Printf("m: %v\n", m)    
    //m: map[age:22 is_ok:true name:令狐楚 score:99.99 subject:[计算机 信息安全]]    
    //读取    
    for key, value := range m {        
        //先看map里边每一个字段的类型 key肯定是string
        fmt.Printf("key:%v------value:%T--%v\n", key, value, value)        
        /*            
        key:name------value:string--令狐楚
        key:subject------value:[]interface {}--[计算机 信息安全]            
        key:is_ok------value:bool--true
        key:age------value:float64--22
        key:score------value:float64--99.99
        */        
        var str string        
        var temp []interface{}        
        var flag bool        
        //var num, num2 float64        
        ////类型断言,取数据        
        switch date := value.(type) {        
            case string:            
            	str = date            
            	fmt.Printf("str: %v\n", str)        
            case bool:            
            	flag = date            
            	fmt.Printf("flag: %v\n", flag)        
            case []interface{}:            
            	temp = date            
           	 fmt.Printf("temp: %v\n", temp)        
            case float64:            
            	fmt.Printf("map[%s]的值得类型为float64，value = %v\n", key, date)         
        }    
    }  
}
```

