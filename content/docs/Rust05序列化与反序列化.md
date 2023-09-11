# Rust序列化与反序列化

> rust中的序列化与反序列化操作

```toml
# Cargo.toml
[dependencies]
serde = {version = "1",features = ["derive"]}
serde_json = "1.0"
serde_yaml = "1.0"

```

### 代码

```rust
use serde::{Deserialize, Serialize};

//使用宏的方式定义
#[derive(Serialize, Deserialize, Debug)]
struct Pay {
    account: String,
    tax_percent: f32,
}
#[derive(Serialize, Deserialize, Debug)]
struct Person {
    name: String,
    age: u8,
    pays: Vec<Pay>,
}

//这些宏做了些什么？
// use serde::ser::{Serialize, SerializeStruct};
// impl Serialize for Person {
//     fn serialize<S>(&self, serialize: S) -> Result<S::Ok, S::Error>
//     where
//         S: serde::Serializer,//这个接口也要实现
//     {
//         let mut s = serialize.serialize_struct("person",3)?;
//         s.serialize_field("name",&self.name)?;
//         s.serialize_field("age",&self.age)?;
//         s.serialize_field("pays",&self.pays)?;
//         s.send();
//     }
// }

fn main() {
    let ps = Person {
        name: "yh".to_string(),
        age: 100,
        pays: vec![
            Pay {
                account: "12301".to_string(),
                tax_percent: 0.2,
            },
            Pay {
                account: "12302".to_string(),
                tax_percent: 0.3,
            },
        ],
    };
    //序列化
    let json_str = serde_json::to_string_pretty(&ps).unwrap();
    let yaml_str = serde_yaml::to_string(&ps).unwrap();
    println!("json:{}", json_str);
    println!("yaml:{}", yaml_str);
    //反序列化
    // let ps_json:Vec<Person> = serde_json::from_str(json_str.as_str()).unwrap();
    //let ps_yaml:Vec<Person> = serde_yaml::from_str(yaml_str.as_str()).unwrap();
    // println!("{:?}",ps_json);
    // println!("----------");
    // println!("{:?}",ps_yaml);
    //对序列化的内容进行操作
    let mut data: serde_json::Value = serde_json::from_str(json_str.as_str()).unwrap();
    println!("data:{}", data.to_string());
    //data:{"age":100,"name":"yh","pays":[{"account":"12301","tax_percent":0.2},{"account":"12302","tax_percent":0.3}]}
    //1. 加字段
    data["tell"] = serde_json::Value::String("15977862012".to_string());
    println!("data:{}", data.to_string());
    //data:{"age":100,"name":"yh","pays":[{"account":"12301","tax_percent":0.2},{"account":"12302","tax_percent":0.3}],"tell":"15977862012"}

    //2. 加json == "car":{"color":"bule","year":"2023"}
    let mut map_value = serde_json::Map::new();
    map_value.insert(
        "color".to_string(),                           //key
        serde_json::Value::String("bule".to_string()), //value
    );
    map_value.insert(
        "year".to_string(),
        serde_json::Value::String("2023".to_string()),
    );
    data["car"] = serde_json::Value::Object(map_value);
    println!("data:{}", data.to_string());
    //data:{"age":100,"car":{"color":"bule","year":"2023"},"name":"yh","pays":[{"account":"12301","tax_percent":0.2},{"account":"12302","tax_percent":0.3}],"tell":"15977862012"}

    //3. 改  ==>改为空值时就相当于删除了
    data["car"]["color"] = serde_json::Value::String("red".to_string());
    println!("data:{}", data.to_string());
    //data:{"age":100,"car":{"color":"red","year":"2023"},"name":"yh","pays":[{"account":"12301","tax_percent":0.2},{"account":"12302","tax_percent":0.3}],"tell":"15977862012"}

    //4. 查
    println!("pays:{}",data["pays"].to_string())
    //pays:[{"account":"12301","tax_percent":0.2},{"account":"12302","tax_percent":0.3}]
  
    // let json_data = std::fs::read_to_string("./test.json").unwrap();
    // //对序列化的内容进行操作
    // let mut data: serde_json::Value = serde_json::from_str(json_data.as_str()).unwrap();
    // println!("data:{}", data.to_string());
    //
    // data["tell"] = serde_json::Value::String("15977862012".to_string());
    // println!("data:{}", data.to_string());
}

```
