# Go读取Excel文档

> 输入Excel文件的路径，同时兼容xls和xlsx两种格式，输出一个数组,包括文件所有内容（仅读取第一行工作表）其中第一行的内容就当作列名:

```go
package main
import (
"fmt"
"time"
"github.com/360EntSecGroup-Skylar/excelize"
)
//读取Excel文件，返回行列数组
func ReadExcel(filename string) []map[string]string {
  ret := []map[string]string{}
  f, err := excelize.OpenFile(filename)
if err != nil {
    fmt.Println("读取Excel文件出错", err.Error())
return ret
  }
  sheets := f.GetSheetMap()
  fmt.Println(sheets)
  sheet1 := sheets[1]
  fmt.Println("第一个工作表:", sheet1)
  rows := f.GetRows(sheet1)
  cols := []string{}
for i, row := range rows {
if i == 0 {
for _, colCell := range row {
        cols = append(cols, colCell)
      }
      fmt.Println("列信息", cols)
    } else {
      theRow := map[string]string{}
for j, colCell := range row {
//log.Println(fmt.Sprintf("%d.%d=%s", i, j, colCell))
        k := cols[j]
        theRow[k] = colCell
      }
      ret = append(ret, theRow)
    }
  }
return ret
}
func main() {
//记录开始执行时间
  start := time.Now()
//读取一个1万行数据的excel文件
  arr := ReadExcel("demo10k.xlsx")
//计算总用时
  elapsed := time.Since(start)
//输出信息
  fmt.Println("记录数：", len(arr))
  fmt.Println("用时：", elapsed)
}
```

