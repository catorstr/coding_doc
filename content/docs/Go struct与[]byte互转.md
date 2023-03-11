# Go struct与[]byte互转 

```go
package main

import (
  "bytes"
  "encoding/gob"
  "fmt"
)


type complexData struct {
  N int
  S string
  M map[string]int
  P []byte
  C *complexData
  E Addr
}


type Addr struct {
  Comment string
}


func main() {


  testStruct := complexData{
    N: 23,
    S: "我是字符串",
    M: map[string]int{"one": 1, "two": 2, "three": 3},
    P: []byte("我是byte"),
    C: &complexData{
      N: 256,
      S: "有故事的人!",
      M: map[string]int{"01": 1, "10": 2, "11": 3},
      E: Addr{
        Comment: "内容正文",
      },
    },
    E: Addr{
      Comment: "Test123123123",
    },
  }


  fmt.Println("Outer struct: ", testStruct)
  fmt.Println("Inner struct: ", testStruct.C)
  fmt.Println("Inner struct: ", testStruct.E)
  fmt.Println("===========================")


  var b bytes.Buffer
  enc := gob.NewEncoder(&b)
  err := enc.Encode(testStruct)
  if err != nil {
    fmt.Println(err)
  }


  dec := gob.NewDecoder(&b)
  var data complexData
  err = dec.Decode(&data)
  if err != nil {
    fmt.Println("Error decoding GOB data:", err)
    return
  }


  fmt.Println("Outer struct: ", data)
  fmt.Println("Inner struct: ", data.C)
  fmt.Println("Inner struct: ", testStruct.E)


}
```

