
#### 创建连接

```
// 参考 https://github.com/go-sql-driver/mysql#dsn-data-source-name 获取详情
dsn := "root:root@tcp(127.0.0.1:3306)/gormtest?charset=utf8mb4&parseTime=True&loc=Local"
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
    //NamingStrategy: schema.NamingStrategy{
    //	TablePrefix:   "gorm_",
    //	SingularTable: true,
    //	NameReplacer:  nil,
    //	NoLowerCase:   true,
    //},
    Logger: log,
})
if err != nil {
    panic(err.Error())
}
```
- 自动创建表
> Db.AutoMigrate(&Student{})