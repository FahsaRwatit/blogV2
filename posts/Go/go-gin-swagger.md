---
title: "Gin中使用Jswaggo生成接口文档"
description: "Gin中使用Jswaggo生成接口文档"
date: 2022-04-08 18:10:00
slug: go-gin-swagger
image:
categories:
    - Go
tags: ["Go"]


---

### 安装

下载swag命令行

```go
go get -u github.com/swaggo/swag/cmd/swag
```

这时，$GOPATH/bin/ 下多了个`swag.exe`

将 $GOPATH/bin/ 配置到环境变量中就可以使用 swag 命令了。

### 注释格式要求

参考相关文档：https://swaggo.github.io/swaggo.io/

### 编写注释

注释分为两部分，一是整体应用的说明，二是具体api的说明。

#### 整体应用说明

main  函数中的注释写项目相关的信息。

```go

// @title           Swagger Example API
// @version         1.0
// @description     This is a sample server celler server.
// @termsOfService  http://swagger.io/terms/

// @contact.name   API Support
// @contact.url    http://www.swagger.io/support
// @contact.email  support@swagger.io

// @license.name  Apache 2.0
// @license.url   http://www.apache.org/licenses/LICENSE-2.0.html

// @host      localhost:8080
// @BasePath  /api/v1

func main() {
    
}
```

#### 具体api的说明

```go
// @Summary 测试SayHello
// @Description 向你说Hello
// @Tags 测试
// @Accept mpfd
// @Produce json
// @Param who query string true "人名"
// @Success 200 {string} string "{"msg": "hello Razeen"}"
// @Failure 400 {string} string "{"msg": "who are you"}"
// @Router /hello [get]
func HandleHello(c *gin.Context) {
	who := c.Query("who")

	if who == "" {
		c.JSON(http.StatusBadRequest, gin.H{"msg": "who are u?"})
		return
	}

	c.JSON(http.StatusOK, gin.H{"msg": "hello " + who})
}
```

### 生成接口文档数据

```go
// 在main.go 同级目录 运行
swag init
// 同时在同级目录下生成了docs文件夹 
./docs
├── docs.go
├── swagger.json
└── swagger.yaml
```



### 引入gin-swagger渲染文档

```go
import (
	_ "web-app/docs" // web-app 为你的模块名称
	swaggerFiles "github.com/swaggo/files"
	gs "github.com/swaggo/gin-swagger"

	"github.com/gin-gonic/gin"
)
```

#### 注册swagger api相关路由

```go
r := gin.Default()
//加入swagger的路由，可以支持在页面访问
r.GET("/swagger/*any", gs.WrapHandler(swaggerFiles.Handler))

// 测试
r.GET("/hello", controller.HandleHello)
```

### 运行项目，查看接口文档

```go
go build
// 我这里使用的是8081端口
文档地址为：http://localhost:8081/swagger/index.html
```



References

[1] https://www.lixueduan.com/post/go/swagger/