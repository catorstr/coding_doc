# gorm Gen模式

> gorm 的gen模式就是根据与数据库的连接，将数据库中的表及字段编程成对应的模型代码

```go
//Build/main.go
package main

import (
	"fmt"
	"gorm.io/driver/mysql"
	"gorm.io/gen"
	"gorm.io/gorm"
)

const dsn = "root:123456@tcp(localhost:3306)/chigo?charset=utf8mb4&parseTime=True&loc=Local"

func main() {
	db, err := gorm.Open(mysql.Open(dsn))
	if err != nil {
		panic(fmt.Errorf("cannot establish db connection: %w", err))
	}
	// 生成实例
	g := gen.NewGenerator(gen.Config{
		OutPath:      "../app/orm/dal",
		ModelPkgPath: "../app/orm/model",
		Mode:         gen.WithDefaultQuery | gen.WithoutContext,
	})
	// 设置目标 db
	g.UseDB(db)
	g.ApplyBasic(g.GenerateAllTable()...) //渲染所有表
	g.ApplyBasic(g.GenerateModel("accounts", gen.FieldType("level", "Level"))) //渲染accounts表的level字段为我们自己定义的Level类型
	g.Execute()
}
```

```go
//App/app.go
package main

import (
	"fmt"
	"github.com/langwan/chigo/Gorm/Gen/App/orm/dal"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
)

const dsn = "root:123456@tcp(localhost:3306)/chigo?charset=utf8mb4&parseTime=True&loc=Local"

func main() {
	db, err := gorm.Open(mysql.Open(dsn))
	if err != nil {
		panic(fmt.Errorf("cannot establish db connection: %w", err))
	}
	dal.SetDefault(db)
	//查询id=1的数据
	//acc, err := dal.Account.Where(dal.Account.ID.Eq(1)).First()
	//if err != nil {
	//	return
	//}
	//
	//fmt.Println(acc)

	//acc, err := dal.Account.Where(dal.Account.Level.Eq(model.LevelManager)).First()
	//查询id为1，2,3,4并且amount>1的的数据
	////dal.Account.Where(dal.Account.ID.In(1, 2, 3, 4), dal.Account.Amount.Gt(1)).Find()
	//
	//fmt.Println(acc, err)
	//
	//_, err = dal.Message.Unscoped().Where(dal.Message.Name.Like("%aa%")).Delete()
	//if err != nil {
	//	return
	//}
	//_, err = dal.Message.Unscoped().Where(dal.Message.ID.In(1, 2, 3, 4, 5)).Delete()
	//if err != nil {
	//	return
	//}
    
	//单字段更新
	dal.Account.Where(dal.Account.ID.Eq(1)).Update(dal.Account.Name, "langwan")
	//多字段更新
	acc, _ := dal.Account.Where(dal.Account.ID.Eq(1)).First()
	acc.Password = "000000"
	acc.Name = "chihuo"
	dal.Account.Save(acc)
	////acc, err := dal.Account.Where(dal.Account.ID.Eq(1)).First()
	////fmt.Println(acc)
	//
	//acc1, err := dal.Account.Where(dal.Account.Money.Eq(1)).First()
	//dal.Account.Where(dal.Account.Type.Eq(model.AccountTypeLevel1)).Count()
	//
	//dal.Account.Where(dal.Account.ID.Eq(1)).Update(dal.Account.Nickanme, "langwan")
	//acc, err := dal.Account.Where(dal.Account.ID.Eq(1)).First()
	//acc.Nickanme = "langwan"
	//acc.Money = 9999999999
	//dal.Account.Save(acc)

	//fmt.Println(acc1, err)

}
```

