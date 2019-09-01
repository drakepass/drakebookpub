## gin使用技巧

* 关于末班引入

~~~ go
r := gin.Default()
	//两者不可以重复使用，只能任选其一
	r.LoadHTMLGlob("template/**/*")  //只能检测到二层文件夹内的所有模板
	r.LoadHTMLFiles("template/login.html","....",....)
	r.GET("/posts/index", func(c *gin.Context) {
    //posts/index.html 是模板的名称，会寻找该名称的模板，如果没有则会找文件名为posts/index.html的模板    
		c.HTML(http.StatusOK, "posts/index.html", gin.H{ 
			"title": "posts/index",
		})
	})
~~~

* 模板的文件命名一般要写全路径,顶层的公共模板直接写名字就可以

~~~~ html
{{define "posts/index.html"}}
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <!--顶层的公共模板直接写名字就可以-->
    {{template "header"}}  
</head>
<body>
    {{.title}}
</body>
</html>
{{end}}
~~~~

* 获取参数：

~~~ go
//从url的querystring中param、form表单的field、url的path中获取参数值
//querystring url?username=drake
r.GET("/user/search", func(c *gin.Context) {
	username := c.DefaultQuery("age", "5") //默认为5 string 类型
	address := c.Query("username") //不传默认为空
	//输出json结果给调用方
	c.JSON(http.StatusOK, gin.H{
		"message":  "ok",
		"username": username,
		"address":  address,
	})
})
//form表单
r.POST("/user/search", func(c *gin.Context) {
	// DefaultPostForm取不到值时会返回指定的默认值
	//username := c.DefaultPostForm("username", "daniel")
	username := c.PostForm("username")
	address := c.PostForm("address")
	//输出json结果给调用方
	c.JSON(http.StatusOK, gin.H{
		"message":  "ok",
		"username": username,
		"address":  address,
	})
})
//path 参数
r.GET("/user/search/:username/:address", func(c *gin.Context) {
	username := c.Param("username")
	address := c.Param("address")
	//输出json结果给调用方
	c.JSON(http.StatusOK, gin.H{
		"message":  "ok",
		"username": username,
		"address":  address,
	})
})
~~~

* 页面跳转、路由跳转

~~~ go
//跳转的两种方式
r.GET("/hello/:name", func(c *gin.Context) {
	name := c.Param("name")
    //第一种向客户端发送重定向链接
	c.Redirect(http.StatusMovedPermanently,"/index/"+name)
    //第二种方法内部请求转换
    c.Request.URL.Path = "/index/"+name
	r.HandleContext(c)
})
~~~

* 普通路由

~~~go
r.GET("/index", func(c *gin.Context) {...})
r.GET("/login", func(c *gin.Context) {...})
r.POST("/login", func(c *gin.Context) {...})
//此外，还有一个可以匹配所有请求方法的Any方法如下：
r.Any("/test", func(c *gin.Context) {
    if c.Request.Method == "POST" {...} //判断请求方法
})
//为没有配置处理函数的路由添加处理程序。默认情况下它返回404代码。
r.NoRoute(func(c *gin.Context) {
	c.HTML(http.StatusNotFound, "views/404.html", nil)
})

~~~



* 路由分组

~~~go
//分组可以嵌套
aagp := r.Group("/aa")
{
	aagp.GET("/bb",login)	   //  /aa/bb
	ccgp := aagp.Group("/cc")
	{
		ccgp.GET("/ff",login)  //  /aa/cc/ff
	}
}
~~~

* 表单文件上传

~~~go
// 处理multipart forms提交文件时默认的内存限制是32 MiB
// 可以通过下面的方式修改字节数
// router.MaxMultipartMemory = 8 << 20  // 8 MiB 
//----单文件----
file, err := c.FormFile("file")
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


//----多文件----
form, _ := c.MultipartForm()
files := form.File["file"]

for _, file := range files {
	log.Println(file.Filename)
	dst := fmt.Sprintf("C:/tmp/%s", file.Filename)
	// 上传文件到指定的目录
	c.SaveUploadedFile(file, dst)
}
~~~

8 << 20 = 8 * 2<sup>20</sup>  x向左移动n位，相当于x*2<sup>n</sup>

* 中间件

~~~go
// StatCost 是一个统计耗时请求耗时的中间件,下面是通过返回一个函数的方式（也可以直接是一个函数），这样 做的好处是可以执行一些前置操作
func StatCost() gin.HandlerFunc {
   	//...do something before
	return func(c *gin.Context) {
		start := time.Now()
		c.Set("name", "小王子")  //该值可以在路由对应的函数中取得
		// 执行其他中间件
		c.Next()
		// 计算耗时
		cost := time.Since(start)
		log.Println(cost)
	}
}
func main() {
    // 新建一个没有任何默认中间件的路由 gin.default()里有两个中间件Logger(), Recovery()
	r := gin.New()
	// 注册一个全局中间件，可注册多个
	r.Use(StatCost())  //如果上面定义的是一个没有返回函数的函数的话直接写函数名
	
	r.GET("/test", func(c *gin.Context) {
		name := c.MustGet("name").(string) //注意这个取法
		log.Println(name)
		c.JSON(http.StatusOK, gin.H{
			"message": "Hello world!",
		})
	})
    // 给/test2路由单独注册中间件（可注册多个）
	r.GET("/test2", StatCost(), func(c *gin.Context) { //如果上面定义的是一个没有返回函数的函数的话直接写函数名
		name := c.MustGet("name").(string)
		log.Println(name)
		c.JSON(http.StatusOK, gin.H{
			"message": "Hello world!",
		})
	})
	r.Run()
}
~~~

* 提交参数绑定到结构体 

~~~go
type formA struct {
  Foo string `json:"foo" xml:"foo" binding:"required"`
}

type formB struct {
  Bar string `json:"bar" xml:"bar" binding:"required"`
}
//针对 `Query`, `Form`, `FormPost`, `FormMultipart`
func SomeHandler(c *gin.Context) {
  objA := formA{}
  objB := formB{}
  // c.ShouldBind 使用了 c.Request.Body，不可重用。
  if errA := c.ShouldBind(&objA); errA == nil {
    c.String(http.StatusOK, `the body should be formA`)
  // 因为现在 c.Request.Body 是 EOF，所以这里会报错。
  } else if errB := c.ShouldBind(&objB); errB == nil {
    c.String(http.StatusOK, `the body should be formB`)
  } else {
    ...
  }
}
//针对`JSON`, `XML`, `MsgPack`, `ProtoBuf`
func SomeHandler(c *gin.Context) {
  objA := formA{}
  objB := formB{}
  // 读取 c.Request.Body 并将结果存入上下文。
  if errA := c.ShouldBindBodyWith(&objA, binding.JSON); errA == nil {
    c.String(http.StatusOK, `the body should be formA`)
  // 这时, 复用存储在上下文中的 body。
  } else if errB := c.ShouldBindBodyWith(&objB, binding.JSON); errB == nil {
    c.String(http.StatusOK, `the body should be formB JSON`)
  // 可以接受其他格式
  } else if errB2 := c.ShouldBindBodyWith(&objB, binding.XML); errB2 == nil {
    c.String(http.StatusOK, `the body should be formB XML`)
  } else {
    ...
  }
}
// c.ShouldBindBodyWith 会在绑定之前将 body 存储到上下文中。 这会对性能造成轻微影响，如果调用一次就能完成绑定的话，那就不要用这个方法。
// 只有某些格式需要此功能，如 `JSON`, `XML`, `MsgPack`, `ProtoBuf`。 对于其他格式, 如 `Query`, `Form`, `FormPost`, `FormMultipart` 可以多次调用 `c.ShouldBind()` 而不会造成任任何性能损失
~~~