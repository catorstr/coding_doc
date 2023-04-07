# 中文排序问题

>     中文排序的规则一般是按照汉字的笔画数和拼音的字母顺序进行排列。具体来说，一般先按照第一笔的笔画数排序，如果第一笔一样，则按照第二笔的笔画数排序，以此类推。如果汉字的笔画数都相同，则按照拼音的字母顺序排序。需要注意的是，在某些情况下，还需要考虑汉字的部首和偏旁部首的排序。


在Go语言中实现中文排序，可以使用sort包中的SortStrings函数配合collate包实现。具体步骤如下：

1. 引入sort和collate包：

```go
import (
 "sort"
 "golang.org/x/text/collate"
 "golang.org/x/text/language"
)
```

2. 初始化collate包中的Collator对象，用于比较字符串：

```go
c := collate.New(language.Chinese)
```

3. 准备需要排序的字符串切片slice：

```go
slice := []string{"张三", "李四", "王五", "赵六"}
```

4. 自定义Less函数，用于比较字符串：

```go
less := func(i, j int) bool {
 return c.CompareString(slice[i], slice[j]) < 0
}
```

5. 调用sort.SortStrings函数进行排序：

```go
sort.Slice(slice, less)
```

完整的示例代码如下：

```go
package main

import (
 "fmt"
 "sort"
 "golang.org/x/text/collate"
 "golang.org/x/text/language"
)

func main() {
 c := collate.New(language.Chinese)
 slice := []string{"张三", "李四", "王五", "赵六"}
 less := func(i, j int) bool {
 return c.CompareString(slice[i], slice[j]) < 0
 }
 sort.Slice(slice, less)
 fmt.Println(slice)
}
```

注意，该示例代码需要使用golang.org/x/text包，需要先使用go get命令安装：

```
go get golang.org/x/text
```
