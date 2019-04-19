# golang 中 github.com/jinzhu/gorm 包学习记录
* 包相关简介
* 包使用命令
* 包使用注意事项

## gorm 包使用相关简介
* 包文档地址
* 产生背景和支持特性
* 包安装命令介绍
* 快速开始代码演示

### 包文档地址
> [地址](http://gorm.io/) 

### 产生背景和支持特性
> The fantastic ORM library for Golang, aims to be developer friendly.

#### Overview
* Full-Featured ORM (almost) 
* Associations (Has One, Has Many, Belongs To, Many To Many, Polymorphism)(关联操作)
* Hooks (Before/After Create/Save/Update/Delete/Find)(支持钩子操作)
* Preloading (eager loading)(预加载)
* Transactions(事物操作)
* Composite Primary Key(复合主键)
* SQL Builder(sql构造)
* Auto Migrations(自动迁移)
* Logger(支持日志)
* Extendable, write Plugins based on GORM callbacks(支持基于回调函数的可扩展的写插件)
* Every feature comes with tests(带测试的特性)
* Developer Friendly(开发人员友好)

### 包安装命令介绍
> go get -u github.com/jinzhu/gorm

### 快速开始代码演示 注意在使用的时候需要引入该类型数据库的驱动包才行 比如 mysql "github.com/go-sql-driver/mysql" 驱动类型
```golang
package main 
import (
	"github.com/jinzhu/gorm"
	_"github.com/jinzhu/gorm/dialects/sqlite"
)

type Product struct {
	gorm.Model
	Code string
	Price uint
}

func main() {
	db, err := gorm.Open("sqlite3", "test.db")
	if err != nil {
		panic("failed to connect database")
	}
	defer db.Close()

	//支持数据库迁移
	db.AutoMigrate(&Product{})

	//插入数据
	dn.Create(&Product{Code:"L1212", Price: 1000})

	//查找读数据
	var product Product
	db.First(&product, 1) // find product with id 1 查找第一条产品中id 为 1的数据
	db.First(&product, "code = ?", "L1212")   //find product with code L1212

	//更新数据操作 update product's price to 2000
	db.Model(&product).Update("Price", 2000)

	//删除数据
	db.Delete(&product)
}
```

## 包使用命令
* 模型定义
* 基础使用(增删改查相关使用)
* 事物操作等特殊操作

* 模型声明 相关代码演示
### 模型定义类型一: 支持数据类型包括普通的golang结构体类型,基础的go类型或者他们的指针 sql.Sacnner和driver.Valuer 等接口类型也是支持的
```golang
type User struct {
  gorm.Model
  Name         string
  Age          sql.NullInt64
  Birthday     *time.Time
  Email        string  `gorm:"type:varchar(100);unique_index"`
  Role         string  `gorm:"size:255"` // set field size to 255
  MemberNumber *string `gorm:"unique;not null"` // set member number to unique and not null
  Num          int     `gorm:"AUTO_INCREMENT"` // set num to auto incrementable
  Address      string  `gorm:"index:addr"` // create index with name `addr` for address
  IgnoreMe     int     `gorm:"-"` // ignore this field
}
```

> 对于上面出现的gorm.Model这个字段 在官方文档中是写: gorm.Model是一个基础 golang的结构体 包括的基本字段有:
ID, CreateAt, UpdateAt, DeleteAt, 这里只是为了方便用户节省时间和代码成本, 如果不需要这里面的某些字段也可以不使用这里的gorm.Model这个字段来生成相应的代码结构, 也可以自己定义相关的
结构体来嵌入这里当做面向对象的复合类型来使用, 下面贴上在gorm中关于这一段代码的结构体的定义, gorm.Model definition
// gorm.Model definition
```golang
type Model struct {
	ID unit `gorm:"primary_key"`
	CreatedAt time.Time
	UpdatedAt time.Time
	DeletedAt *time.Time
}
```

> 这里还有一个特性就是:在指定结构体某一个field作为主键的时候, gorm使用的默认会将结构体中名称为ID 作为主键, 还有也可以在字段后面打一个tag声明其是主键的属性 例如下面的结构体声明
```golang
type User struct {
  ID   string // field named `ID` will be used as primary field by default
  Name string
}

// Set field `AnimalID` as primary field
type Animal struct {
  AnimalID int64 `gorm:"primary_key"`
  Name     string
  Age      int64
}
```

> 还有一点就是关于结构体名称与数据库表名的映射的问题,默认采用的pluralized(复数形式) 即表名是结构体名的小写复数形式 自己手动指定表明, 不一定要按照默认的规则但是也可以自己来指定表明对应映射, 例如下面的代码演示:
```golang
type User struct{}  //default table name is `users`

// Set User's table name to be `profiles`
func (User) TableName() string {
	return "profiles"
}

// 或者这里根据对象的某一个属性来指定要映射的表名
func (u User) TableName() string {
	if u.Role == "admin" {
		return "admin_users"
	} else {
		return "users"
	}
}

//关闭复数表名映射的方式 这样的话 比如User 这样就会映射成小写的 user表
//Disable table name's pluralization, if set to true, `User`'s table name will be `user`
db.SingularTable(true)
```

> 用某一个结构体来创建表的使用情况 代码相关演示如下所示:
```golang
// Create `deleted_users` table with struct User's definition
db.Table("deleted_users").CreateTable(&User{})

var deleted_users []User
db.Table("deleted_users").Find(&deleted_users)
//// SELECT * FROM deleted_users;

db.Table("deleted_users").Where("name = ?", "jinzhu").Delete()
//// DElETE FROM deleted_users WHERE name = 'jinzhu'
```

> 设置统一的表明前缀: Change default tablenames: You can apply any rules on the default table name by defining the DefaultTableNameHandler.
```golang
gorm.DefaultTableNameHandler = func(db *gorm.DB, defaultTableName string) string {
	return "prefix_" + defaultTableName
}
```

> 关于各个表名字段的默认映射规则 默认采用的是小写的蛇形命名规范 类似于这样的命名规范(下划线命名规范) like_this 代码结构体定义如下:
```golang
type User struct {
	ID uint // column name is `id`
	Name string // column name is `name`
	Birthday time.Time // column name is `birthday`
	CreatedAt time.Time // column name is `created_at`
}

// Overriding Column Name
// 重写字段映名称
type Animal struct {
	AnimalId int64 `gorm:"column:beast_id`  // set column name to `beast_id`
	Birthday time.Time `gorm:"column:day_of_the_beast` // set column name to `day_of_the_beasr`
	Age int64 `gorm:"column:age_of_the_beast` // set column name to `age_of_the_beast`
}
```

> 关于gorm中的时间戳跟踪的问题的注意事项和解析
* CreatedAt 大多数模型里面都会有这个CreatedAt这个字段, 它会被设置成record记录最先被创建的时间戳记录
```golang
db.Create(&user)  // will set `CreatedAt` to current time

// To change its value, you could use 'Update'
db.Model(&user).Update("CreatedAt", time.Now())
```
* UpdatedAt 大多数模型也都会有一个UpdatedAt这个字段, 当该条记录修改的时候会被设置成当时修改的时间
```golang
db.Save(&user)  // will set `UpdatedAt` to current time
db.Model(&user).Update("name", "jinzhu") // will set `UpdatedAt` to current time
```
* DeletedAt 大多数的模型也都会有DeletedAt这个字段,当这条记录被删除的时候会被设置成删除当时的时间,并不会真的从数据库里面删除掉, 但是会在这个实例上面调用删除函数的时候会设置这个字段删除的时间值, 是一种软删除. 其原文文档如下所示:

> For models with a DeletedAt field, when Delete is called on that instance, it won’t truly be deleted from database, but will set its DeletedAt field to the current time. Refer to Soft Delete

### soft delete 文档相关解释 软删除
> If a model has a DeletedAt field, it will get a soft delete ability automatically! When calling Delete, the record will not be permanently removed from the database; rather, the DeletedAt‘s value will be set to the current time
```golang
db.Delete(&user)
//// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;

// Batch Delete 批量删除
db.Where("age = ?", 20).Delete(&User{})
//// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;

// Soft deleted records will be ignored when query them  //这里的一点需要特别注意 当query这条记录的时候,软删除这种操作会被忽略掉
db.Where("age = 20").Find(&user)
//// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;

// Find soft deleted records with Unscoped
db.Unscoped().Where("age = 20").Find(&users)
//// SELECT * FROM users WHERE age = 20;
```

### Delete record permanently 永久删除数据库记录的方法使用 代码演示如下: Unscoped应该是指软删除的情况
```golang
// Delete record permanently with Unscoped
db.Unscoped().Delete(&order)
//// DELETE FROM orders WHERE id=10;
```

### gorm包基础命令中查找的相关操作具体看文档 很仔细 这里主要是记录一些注意事项的问题 具体的操作或其他相关问题具体请查看文档

> 主要是一些非零值的设置的问题 在官方文档里面 我们可以知道在创建数据和查找数据的时候:  all fields having a zero value, like 0, '', false or other zero values, won’t be saved into the database but will use its default value. If you want to avoid this, consider using a pointer type or scanner/valuer, e.g: 即是所有的字段都可以有零值, 比如0, '', false, 或者其他的zero values但是有这些零值的字段不会被保存进数据库里面但是会使用它设置的默认值保存进数据库里面, 如果你想要避免这个问题,你可以考虑使用指针类型scanner或者valuer
```golang
// Use pointer value
type User struct {
	gorm.Model
	Name stringAge *int `gorm:"default:18"`
}

// Use scanner/valuer
type User struct {
	gorm.Model
	Name string
	Age sql.NullInt64 `gorm:"default:18"`
}
```

> 还有在查找的时候也需要注意相关空值的问题 就是在设置查找条件的时候 如果设置了某个字段的查找条件为零值的时候 查找条件会忽略该字段匹配
NOTE When query with struct, GORM will only query with those fields has non-zero value, that means if your field’s value is 0, '', false or other zero values, it won’t be used to build query conditions, for example:
```golang
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
//// SELECT * FROM users WHERE name = "jinzhu";
```
> you could consider to use pointer type or scanner/valuer to avoid this. 可以考虑使用指针类型或者scanner/valuer去避免这个情况
```golang
// Use pointer value
type User struct {
  gorm.Model
  Name string
  Age  *int
}

// Use scanner/valuer
type User struct {
  gorm.Model
  Name string
  Age  sql.NullInt64
}
```
