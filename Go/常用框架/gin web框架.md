

#### 参数获取
- url中的参数获取
> context.GetQuery("name")
- 获取url的路径参数
> 路径申明：/userTest/:userId  
> 获取参数context.Param("userId")

- 获取get 请求url后的?name=ss参数
> context.Query("name")  

- 获取body里面参数
> postForm（“key”）  可以解析 form-data， 和 www-form-urlencoded两种type
- 对接json格式的 body    
> context.shouldBindJson (&obj)

#### 文件操作
- file上传
```
fileHeader, err := context.FormFile("file")
if err != nil {
    context.String(500, err.Error())
    return
}
err = context.SaveUploadedFile(fileHeader, "ss.jpg")
```
- file 下载
> 	context.File(fileName)


#### 多处理函数
- 多handler 方式
```
默认依次执行
context.next() 表示在当前handler中 执行下一个hander 函数，
context.abort() 表示在当前hanler中执行完，不继续执行后面的hander
abort（）要在next（）之前才有效
```  
#### 中间件
- 全局中间件
> router.Use(logMiddlewire)  logMiddlewire 里面也能使用 next() abort()的逻辑
-  可以 传值给后面的函数使用。
```
 设置： context.Set("middName", "log")
 获取： middName := c.GetString("middName")
```

####  路由分组
- 按照功能模块统一模块前缀
```
    //file
	fileGroup := router.Group("file")
	fileGroup.POST("/upload", _fileUpload)   //访问路径： file/upload
	fileGroup.GET("/download", _fileDownload) //访问路径： file/download
```


```
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"io"
	"net/http"
	"os"
)

func main() {

	router := gin.Default()
	router.Use(logMiddlewire)

	router.GET("/_get", _get)
	router.GET("/userTest/:userId", param)
	router.POST("/addUser", postFrom)

	//restful
	router.POST("/user/", _addUser)
	router.POST("/userJson/", _addUserJson)
	router.PUT("/user/:userId", _editUser)
	router.GET("/user/:userId", _getDetail)
	router.GET("/users", _getList)
	router.DELETE("/user/:userId", _deleteUser)

	//file
	fileGroup := router.Group("file")
	fileGroup.POST("/fileUpload", _fileUpload)
	fileGroup.GET("/fileDownload", _fileDownload)

	err := router.Run("0.0.0.0:8090")
	if err != nil {
		panic("err:" + err.Error())
	}

	http.ListenAndServe(":8090", router)
}

// 日志中间件
func logMiddlewire(context *gin.Context) {
	fmt.Println("log:   path为", context.Request.RequestURI)
	//context.Next()
	context.Set("middName", "log")
	fmt.Println("log:   httpCode为", context.Writer.Status())
}

func _fileDownload(context *gin.Context) {
	fileName := context.PostForm("fileName")
	//file, err := os.Open(fileName)
	//if err != nil {
	//	context.String(500, err.Error())
	//	return
	//}
	//defer file.Close()
	//_, err = io.Copy(context.Writer, file)
	//if err != nil {
	//	context.String(500, err.Error())
	//	return
	//}
	context.File(fileName)

}

func _fileUpload(context *gin.Context) {
	fileHeader, err := context.FormFile("file")
	if err != nil {
		context.String(500, err.Error())
		return
	}
	//err = context.SaveUploadedFile(fileHeader, "ss.jpg")
	file, err := fileHeader.Open()
	if err != nil {
		context.String(500, err.Error())
		return
	}
	fileName := context.PostForm("fileName")
	fmt.Println("fileName:", fileName)
	openFile, err := os.OpenFile(fileName, os.O_CREATE|os.O_WRONLY, 644)

	if err != nil {
		context.String(500, err.Error())
		return
	}
	_, err = io.Copy(openFile, file)
	if err != nil {
		context.String(500, err.Error())
		return
	}
	openFile.Close()
}

func _get(context *gin.Context) {
	//context.String(200, "hello world!")
	fmt.Println(context.GetQuery("name"))
	context.Next()
	fmt.Println(context.Query("age"))
	fmt.Println(context.QueryMap("map"))
	context.JSON(200, User{
		Name: "Yds",
		Age:  20,
	})
}

// 动态参数
func param(context *gin.Context) {
	fmt.Println(context.Param("userId"))
}

// 表单类型：form-data  form-urlencoded
func postFrom(context *gin.Context) {
	fmt.Println(context.PostForm("name"))
}

type User struct {
	Id   string
	Name string
	Age  int
}

//result ful 类型

func _getList(c *gin.Context) {
	userList := []User{{
		Id:   "1",
		Name: "小王",
		Age:  10,
	},
		{
			Id:   "2",
			Name: "小李",
			Age:  20,
		},
		{Id: "3",
			Name: "小张",
			Age:  30,
		}}
	c.JSON(200, userList)
}
func _getDetail(c *gin.Context) {
	id := c.Param("userId")
	if id == "" {
		c.String(500, "userId is empty")
		return
	}

	c.JSON(200, User{
		Id:   id,
		Name: "晓晓",
		Age:  25,
	})
}
func _addUser(c *gin.Context) {
	name := c.PostForm("name")
	age := c.PostForm("age")
	if name == "" || age == "" {
		c.String(500, "缺少必要参数")
		return
	}
	fmt.Println("add user ok , name=", name, " age=", age)
	c.JSON(200, gin.H{})
}
func _addUserJson(c *gin.Context) {
	var user User
	err := c.BindJSON(&user)
	if err != nil {
		c.String(500, err.Error())
		return
	}

	if user.Name == "" || user.Age == 0 {
		c.String(500, "缺少必要参数")
		return
	}
	fmt.Println("add user ok ,", user)
	c.JSON(200, gin.H{})
}
func _editUser(c *gin.Context) {
	id := c.Param("userId")
	name := c.PostForm("name")
	age := c.PostForm("age")
	fmt.Printf("id:%s name:%s,age:%s", id, name, age)
	c.JSON(200, gin.H{})
}
func _deleteUser(c *gin.Context) {
	id := c.Param("userId")
	middName := c.GetString("middName")
	fmt.Println("delete user ,Id=", id)
	fmt.Println("middName", middName)
	c.JSON(200, gin.H{})
}

```